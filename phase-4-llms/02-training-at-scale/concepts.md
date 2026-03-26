# Concepts: Training at Scale

> **Tag: core** — Understanding how large models are trained explains their capabilities, limitations, and costs — and why you can't replicate frontier results on a laptop.

---

## Why scale matters

The story of modern LLMs is largely a story of scale. The models you use via API today exist because researchers discovered that making neural networks dramatically bigger — and training them on dramatically more data — produces qualitatively better capabilities, not just incrementally better ones.

This isn't obvious. You might expect a model with 10× more parameters to be 10× better at answering questions. What researchers found instead is that scale produces **emergent capabilities**: abilities that didn't exist at smaller sizes appear suddenly as models grow. Multi-step reasoning, code generation, analogical thinking — these weren't explicitly programmed in. They emerged from scale.

But scale comes with a price. Training GPT-3 (175 billion parameters) cost an estimated $4–12 million in compute alone. Frontier models in 2024–2025 likely exceed $100 million per training run. Understanding what makes training expensive — and how engineers manage that cost — is essential context for understanding the models you build with.

---

## Scaling laws

In 2020, OpenAI researchers published findings now called the **Kaplan scaling laws**: model performance improves predictably and smoothly as you scale parameters, data, and compute — following a power law relationship.

In 2022, DeepMind published the **Chinchilla paper**, which refined this significantly. Its key finding: **most large models at the time were undertrained — they had too many parameters for the amount of data they were trained on.**

The Chinchilla insight, roughly:

```
For a fixed compute budget:
  train a SMALLER model on MORE data
  rather than a LARGER model on LESS data
```

The recommended ratio: **~20 tokens of training data per model parameter** for compute-optimal training.

| Model | Parameters | Tokens trained on | Chinchilla-optimal? |
|---|---|---|---|
| GPT-3 | 175B | ~300B | No — undertrained (~3.5T would be optimal) |
| Gopher | 280B | 300B | No |
| Chinchilla | 70B | 1.4T | Yes — same compute, better results |
| Llama 2 7B | 7B | 2T | Over-trained by compute standards, but cheap to run |
| Llama 3 8B | 8B | 15T | Intentionally over-trained for inference efficiency |

The Llama family deliberately trains smaller models on far more data than compute-optimal, because the expensive part (training) happens once at Meta, but the cheap part (inference) happens billions of times for users. A smaller, well-trained model is much cheaper to serve.

> **Production note:** When evaluating model size vs. capability, parameter count alone tells you little. A 7B model trained on 15 trillion tokens can outperform a 70B model trained on 1 trillion tokens on many benchmarks. Always look at training data volume alongside parameter count.

---

## The compute stack

Training a frontier LLM requires infrastructure most companies don't own:

**GPUs / TPUs** — The compute backbone. NVIDIA H100s are the current standard for LLM training. A single H100 costs $25,000–$40,000 to buy (or $2–4/hour to rent on cloud). Frontier training runs use thousands simultaneously.

**High-speed interconnects** — GPUs must share model state and gradients during training. The bandwidth between GPUs becomes the bottleneck as you scale. **NVLink** connects GPUs within a server at ~900 GB/s. **InfiniBand** connects servers at ~400 Gb/s. Without fast interconnects, adding more GPUs makes training slower, not faster — communication overhead exceeds compute benefit.

**Distributed storage** — Training datasets can be tens of terabytes. The storage system must deliver data fast enough to keep thousands of GPUs busy. A GPU cluster waiting on disk I/O is a very expensive way to read files.

**Orchestration and fault tolerance** — With thousands of GPUs running for weeks, hardware failures are not exceptional events — they're expected. One GPU failing can bring down the entire training run unless the job checkpoints frequently and restarts cleanly.

---

## Why one GPU isn't enough

A model with 70 billion parameters stored in 16-bit precision requires ~140 GB of memory just for the weights. A single H100 has 80 GB of memory. The model doesn't fit.

During training, it's worse:

```
Memory required to train a model:
  Model weights:       1× parameter count × bytes per param
  Gradients:           1× (same size as weights)
  Optimizer state:     2× (Adam stores first + second moment estimates)
  Activations:         depends on batch size and sequence length — often large

Total: roughly 12–16 bytes per parameter for Adam with mixed precision
→ 70B model: ~70B × 16 bytes ≈ 1.1 TB
```

1.1 TB across 80 GB GPUs means you need at minimum 14 GPUs just to hold the state — before accounting for batch size, sequence length, or any overhead. In practice, you need many more.

Distributed training solves this by splitting the work. There are three main strategies, usually combined:

---

## Data parallelism

**The idea:** Each GPU holds a complete copy of the model. The training batch is split across GPUs — each GPU processes its shard independently and computes gradients. Gradients are then averaged across all GPUs (via an AllReduce operation), and all model copies update together.

```
GPU 1: [full model] ← batch shard 1 → gradients_1 ─┐
GPU 2: [full model] ← batch shard 2 → gradients_2 ─┼─ AllReduce → all update
GPU 3: [full model] ← batch shard 3 → gradients_3 ─┘
```

**Scales well when** the model fits on a single GPU. Each GPU does the same work in parallel; communication happens once per step (the gradient average). PyTorch's `DistributedDataParallel` (DDP) implements this.

**Breaks when** the model is too large to fit on one GPU — which is the case for anything above ~20B parameters at reasonable batch sizes.

---

## Tensor parallelism

**The idea:** Split individual weight matrices across GPUs. Each GPU holds a horizontal or vertical shard of each layer's weights, and all GPUs process the full batch together.

```
Attention weight matrix W (4096 × 4096), split across 4 GPUs:
  GPU 1: W columns [0:1024]
  GPU 2: W columns [1024:2048]
  GPU 3: W columns [2048:3072]
  GPU 4: W columns [3072:4096]
```

During the forward pass, GPUs communicate to combine their partial results using AllReduce at each layer. This happens at every single layer, every forward pass — it requires extremely fast GPU interconnects. Tensor parallelism is typically used only within a single server node (over NVLink), not across nodes (where bandwidth is too limited).

---

## Pipeline parallelism

**The idea:** Split the model's layers across GPUs. Each GPU holds a consecutive group of layers and passes activations forward to the next GPU.

```
GPU 1: Layers 1–12   →  activations  →
GPU 2: Layers 13–24  →  activations  →
GPU 3: Layers 25–36  →  activations  →
GPU 4: Layers 37–48
```

**The problem — the pipeline bubble:** GPU 2 can't start until GPU 1 finishes processing the batch. At the start and end of each batch, GPUs sit idle waiting. In a naive pipeline, most GPUs are idle most of the time.

**Micro-batching** reduces the bubble: instead of one large batch, use many small micro-batches that fill the pipeline continuously. GPT-NeoX, Megatron-LM, and other frameworks implement this.

Pipeline parallelism works well across server nodes (lower communication frequency than tensor parallelism) and is commonly paired with tensor parallelism within nodes.

---

## ZeRO: Sharding optimizer state

Even when using data parallelism, each GPU holds a full copy of the optimizer state — which for Adam is 3× the model size in FP32 (weights + first moment + second moment). For a 70B model, that's ~840 GB per GPU.

**ZeRO** (Zero Redundancy Optimizer, from Microsoft's DeepSpeed) solves this by sharding across GPUs instead of replicating:

| ZeRO Stage | What's sharded | Approx. memory reduction |
|---|---|---|
| Stage 1 | Optimizer states | ~4× |
| Stage 2 | + Gradients | ~8× |
| Stage 3 | + Model weights | ~N× where N = GPU count |

ZeRO Stage 3 means each GPU holds only 1/N of the model weights. When a GPU needs a weight shard it doesn't have, it fetches it from the GPU that does. This increases communication overhead but makes otherwise-impossible model sizes trainable.

Most frontier training uses ZeRO-style optimization, often combined with model and pipeline parallelism in a "3D parallelism" setup.

---

## Mixed precision training

Training in full 32-bit floating point (FP32) is accurate but memory-intensive. A 70B model in FP32 takes 280 GB just for weights. Mixed precision halves this:

**The approach:**
- Forward pass and gradient computation: done in **16-bit** (BF16) — half the memory, faster on modern GPUs
- Master weights and optimizer state: maintained in **FP32** — necessary to prevent precision loss from accumulating over millions of update steps

```
Memory per parameter:
  FP32:   4 bytes  (full precision, baseline)
  FP16:   2 bytes  (high precision, smaller range — can overflow)
  BF16:   2 bytes  (lower precision, same range as FP32 — preferred)
  INT8:   1 byte   (used for inference, not training)
  INT4:   0.5 bytes (quantization, inference only)
```

**Why BF16 over FP16 for training:**
- FP16 has a smaller dynamic range — large gradients can overflow to `inf`, causing training instability (loss spikes or NaN values)
- BF16 has the same exponent range as FP32, making it stable for large gradient values
- A100 and H100 GPUs have native BF16 support at the same speed as FP16
- BF16 has become the default for LLM pre-training

---

## Gradient checkpointing

During the forward pass, each layer produces **activations** that must be stored to compute gradients during the backward pass. For a large model with large batch sizes and long sequences, these activations consume more memory than the weights themselves.

**Gradient checkpointing** (also called activation recomputation): instead of storing all activations, store only a subset and recompute the rest on demand during the backward pass.

```
Without checkpointing:   store all activations   → low compute, high memory
With checkpointing:      store subset, recompute  → ~33% more compute, 60–70% less memory
```

This is a deliberate compute-for-memory tradeoff. It makes training slower (more forward-pass compute) but allows larger batch sizes or larger models to fit in GPU memory. For models that would otherwise OOM (out of memory), it's not optional — it's the difference between running and not running.

---

## Training data at scale

The data a model is trained on shapes its capabilities as much as its architecture. A 70B model trained on low-quality data will perform worse than a 7B model trained on well-curated data for most real-world tasks.

**Typical dataset composition for a frontier model:**

| Source | Role |
|---|---|
| Web crawls (Common Crawl) | Scale — trillions of tokens, but noisy and redundant |
| Books and literature | Long-form coherent reasoning; narrative structure |
| Code (GitHub, etc.) | Programming ability; structured logical reasoning |
| Academic papers (arXiv, etc.) | Scientific reasoning; factual density |
| Wikipedia and curated sources | High-quality factual grounding |
| Instruction/dialogue data | Used in fine-tuning, not pre-training |

**The curation pipeline — what happens before training:**

1. **Language filtering** — keep only documents in the target language(s)
2. **Deduplication** — remove near-duplicate documents; duplicated content causes models to memorise rather than generalise
3. **Quality filtering** — heuristics (length, punctuation ratios), classifier-based filtering (trained to distinguish high-quality from spam), perplexity scoring
4. **Toxic content filtering** — reduce hateful, illegal, or harmful content in pre-training data
5. **PII scrubbing** — reduce personally identifiable information
6. **Mixture tuning** — adjusting the ratio of data sources matters significantly; more code → better at code; more multilingual data → better multilingual performance

> The Llama 3 paper notes that the data pipeline was the single most important factor in model quality — more impactful than architecture changes. Getting data curation right is harder than it sounds and often consumes more engineering effort than the training code itself.

---

## Training instability

Running a large training job for weeks without failure is a significant engineering challenge. Things go wrong.

**Common failure modes and their causes:**

**Loss spikes** — The loss suddenly jumps before recovering (or not). Caused by: low-quality or corrupted batches, learning rate too high at a given point, or floating point precision issues. Solutions: gradient clipping, careful learning rate schedules, better data filtering, and skipping problematic batches.

**Gradient explosion** — Gradients grow exponentially through layers, causing parameters to update by enormous amounts. **Gradient clipping** addresses this: if the gradient norm exceeds a threshold (typically 1.0), scale it down proportionally. All major training frameworks implement this as a one-line addition.

**Training divergence** — Loss stops improving and starts increasing. Often unrecoverable. The run must restart from the last valid checkpoint.

**NaN / Inf losses** — Floating point overflow produces non-finite values that propagate through the computation graph and corrupt all subsequent updates. More common with FP16 than BF16; automatic loss scaling compensates in FP16 regimes.

**Hardware failures** — At thousands of GPUs over weeks, GPU failures, network outages, and node crashes are expected events. Training jobs must checkpoint model state regularly (typically every few hundred steps) and resume cleanly. Meta and Google report that frontier training runs encounter multiple hardware failures per day.

---

## The real cost of training

Training cost = (number of floating-point operations) / (hardware throughput) × (hardware rental cost)

A rough formula for pre-training compute:

```
FLOPs ≈ 6 × parameters × training tokens
```

For GPT-3: 6 × 175B × 300B ≈ 3.1 × 10²³ FLOPs

At an H100's ~1,000 TFLOPs/s (achievable throughput with BF16):
→ ~3.1 × 10⁸ seconds of single-GPU time
→ with 1,000 H100s: ~87 hours → but real overhead means 2–4× longer

**Estimated training costs:**

```
GPT-3 (175B, 300B tokens, 2020):           ~$4–12M
PaLM (540B, 780B tokens, 2022):            ~$8–23M
Llama 2 70B (2T tokens, 2023):             ~$3–5M
Llama 3 405B (15T tokens, 2024):           ~$30–60M (estimated)
GPT-4 (2023, architecture unknown):        ~$50–100M+ (estimated)
Frontier models (2024–2025):               Likely $100M–$500M
```

These are compute-only estimates. Add: engineering salaries, failed runs (most large training runs encounter at least one restart), data acquisition and curation, evaluation infrastructure, and post-training (instruction tuning, alignment). The full cost of a frontier model is typically 3–5× the raw compute cost.

> **What this means for developers:** You will not train a frontier model from scratch — the economics don't exist outside of a handful of labs. What you *will* do is fine-tune existing models, which costs orders of magnitude less (Phase 4, Module 3). Understanding training cost calibrates your intuitions about which optimizations matter and why open-weight models are such a big deal.

---

## What this means for you as a developer

You won't run pre-training. But understanding it changes how you make decisions:

| Situation | How training knowledge helps |
|---|---|
| Choosing a model | Smaller models trained on more data often beat larger undertrained ones. Parameter count is not the right metric alone. |
| Comparing open-weight models | Llama 3 8B trained on 15T tokens vs. a 13B trained on 1T: the smaller one often wins on benchmarks. |
| Understanding capability gaps | A model's weaknesses often trace back to data gaps — if a domain was underrepresented in training, the model will struggle there. |
| Fine-tuning decisions | Fine-tuning works on top of pre-training. If the base model never saw a domain, fine-tuning adds less than you'd expect. |
| Self-hosting decisions | A 70B model in BF16 requires ~140 GB VRAM — minimum 2× H100 80GB or 4× A100 80GB. Understanding why prevents expensive miscalculations. |
| Evaluating providers | Pricing differences between model providers reflect training investment, serving infrastructure, and model capability — not arbitrary markup. |

---

## Summary

| Concept | Key point |
|---|---|
| Scaling laws | Performance improves predictably with scale. Chinchilla: smaller model + more data often beats larger model + less data for same compute |
| Compute stack | Frontier training requires thousands of GPUs, fast interconnects (NVLink, InfiniBand), and months of wall-clock time |
| Data parallelism | Split batches across GPUs, each with full model copy. Simplest strategy; works until model won't fit on one GPU |
| Tensor parallelism | Split weight matrices across GPUs within a server. Needed for large models; requires fast NVLink interconnects |
| Pipeline parallelism | Split layers across GPUs across servers. Introduces pipeline bubbles; micro-batching reduces idle time |
| ZeRO / DeepSpeed | Shard optimizer state, gradients, and weights to reduce per-GPU memory. Stage 3 enables N× memory reduction across N GPUs |
| Mixed precision (BF16) | Train forward/backward in BF16 for speed and memory; maintain FP32 master weights for stable updates |
| Gradient checkpointing | Recompute activations during backward pass — saves 60–70% activation memory at cost of ~33% more compute |
| Training instability | Loss spikes, gradient explosion, NaN values, and hardware failures are expected at scale. Checkpointing and gradient clipping are essential |
| Training cost | Frontier models cost $10M–$500M+ to train. Explains API pricing, the value of open-weight models, and why fine-tuning is practical where training isn't |

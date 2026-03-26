# Training at Scale

**Phase 4 — Module 2** | Tag: `core` | Type: `theory only`

This module explains how large language models are actually trained — the infrastructure, parallelism strategies, memory optimizations, and data pipelines that make frontier models possible. It is not about writing training code; it is about building the mental model that explains *why* models behave the way they do and *why* certain infrastructure decisions get made.

---

## What this module covers

| Section | What you'll understand |
|---|---|
| [Why scale matters](concepts.md#why-scale-matters) | How scale produces emergent capabilities, not just incremental improvement |
| [Scaling laws](concepts.md#scaling-laws) | The Chinchilla finding: smaller model + more data often beats larger model + less data |
| [The compute stack](concepts.md#the-compute-stack) | GPUs, interconnects, storage — what the infrastructure looks like |
| [Why one GPU isn't enough](concepts.md#why-one-gpu-isnt-enough) | The memory math that forces distributed training |
| [Data parallelism](concepts.md#data-parallelism) | Splitting batches across GPUs, each with a full model copy |
| [Tensor parallelism](concepts.md#tensor-parallelism) | Splitting weight matrices across GPUs within a server |
| [Pipeline parallelism](concepts.md#pipeline-parallelism) | Splitting layers across GPU servers; pipeline bubbles and micro-batching |
| [ZeRO / DeepSpeed](concepts.md#zero-sharding-optimizer-state) | Sharding optimizer state, gradients, and weights to reduce per-GPU memory |
| [Mixed precision training](concepts.md#mixed-precision-training) | BF16 vs FP16 vs FP32 and why BF16 is now the default |
| [Gradient checkpointing](concepts.md#gradient-checkpointing) | Trading compute for memory during the backward pass |
| [Training data at scale](concepts.md#training-data-at-scale) | Dataset composition and the curation pipeline |
| [Training instability](concepts.md#training-instability) | Loss spikes, gradient explosion, NaN values, fault tolerance |
| [The real cost of training](concepts.md#the-real-cost-of-training) | What frontier models cost and why it matters for developers |
| [What this means for you](concepts.md#what-this-means-for-you-as-a-developer) | Practical implications for model selection, fine-tuning, and infrastructure decisions |

---

## Prerequisites — concepts you should understand first

These are covered in earlier modules. If any feel unfamiliar, revisit them before continuing.

| Concept | Where it's covered |
|---|---|
| What an LLM is — the function from tokens to probability distribution | [Phase 4 → 01: LLM Fundamentals → What an LLM actually is](../01-llm-fundamentals/concepts.md#what-an-llm-actually-is) |
| The three-stage training pipeline (pre-training, instruction tuning, alignment) | [Phase 4 → 01: LLM Fundamentals → The training pipeline](../01-llm-fundamentals/concepts.md#the-training-pipeline) |
| What a loss function measures and how cross-entropy works for next-token prediction | [Phase 2 → 01: Loss Functions → Cross-entropy loss](../../phase-2-how-models-learn/01-loss-functions/concepts.md#cross-entropy-loss-classification) |
| The training loop — forward pass, loss, backward pass, weight update | [Phase 2 → 02: Training Loop](../../phase-2-how-models-learn/02-training-loop/concepts.md) |
| Gradient descent and what gradients are | [Phase 2 → 01: Loss Functions → Loss surfaces](../../phase-2-how-models-learn/01-loss-functions/concepts.md#loss-surfaces-and-local-minima) |
| The Transformer architecture — attention, layers, parameter count | [Phase 3 → 04: Transformers](../../phase-3-architectures/04-transformers/concepts.md) |

---

## What comes next — where these concepts are used

| Concept from this module | Where it becomes directly relevant |
|---|---|
| Chinchilla scaling / model selection | Choosing between open-weight models in [Phase 4 → 04: Inference & Deployment](../04-inference-deployment/concepts.md) and [Phase 5 → 01: API Calls](../../phase-5-building/01-api-calls/concepts.md) |
| Training cost and data volume | Justifying fine-tuning over training in [Phase 4 → 03: Efficient Training](../03-efficient-training/concepts.md) |
| Distributed training (tensor/pipeline parallelism) | Context for self-hosting decisions in [Phase 4 → 04: Inference & Deployment](../04-inference-deployment/concepts.md) |
| Mixed precision (BF16/INT8) | Quantization for inference in [Phase 4 → 04: Inference & Deployment](../04-inference-deployment/concepts.md) |
| Training data curation | Foundation for understanding RAG in [Phase 5 → 03: RAG](../../phase-5-building/03-rag/concepts.md) |
| Training instability / checkpoint recovery | Background for understanding model reliability discussed in [Phase 6 → 01: Safety & Alignment](../../phase-6-responsible-ai/01-safety-alignment/concepts.md) |
| Compute costs and economics | Context for the scaling and societal discussion in [Phase 6 → 03: Scaling](../../phase-6-responsible-ai/03-scaling/concepts.md) |

---

## Key terms introduced in this module

| Term | One-line definition |
|---|---|
| Scaling laws | Predictable power-law relationship between compute, model size, data, and performance |
| Chinchilla-optimal | The training token count (~20× parameters) that maximises performance for a given compute budget |
| Data parallelism | Each GPU holds a full model copy; batch is split across GPUs |
| Tensor parallelism | Weight matrices are split across GPUs within a server |
| Pipeline parallelism | Model layers are split across GPU servers; activations passed between them |
| Pipeline bubble | Idle GPU time at the start and end of each batch in pipeline parallelism |
| ZeRO (Stage 1/2/3) | Sharding optimizer state, gradients, and weights across GPUs to reduce per-GPU memory |
| Mixed precision | Training in BF16/FP16 while maintaining FP32 master weights |
| BF16 | 16-bit float with the same range as FP32 — preferred over FP16 for training stability |
| Gradient checkpointing | Recomputing activations during backward pass to reduce memory at cost of compute |
| AllReduce | Communication operation that averages gradients across all GPUs in data parallelism |
| Loss spike | Sudden temporary increase in training loss; indicates a problematic batch or instability |
| Gradient clipping | Capping gradient norm to prevent explosive updates; standard practice in LLM training |

---

## Module files

| File | Contents |
|---|---|
| [concepts.md](concepts.md) | Core theory — read this first |
| [quiz.md](quiz.md) | 13 self-check questions with detailed answers |
| [resources.md](resources.md) | Curated papers, frameworks, GitHub repos, and practitioner guides |

---

## Related modules in this phase

| Module | Relationship |
|---|---|
| [01 — LLM Fundamentals](../01-llm-fundamentals/README.md) | Prerequisite — what LLMs are and the three-stage training pipeline |
| [03 — Efficient Training](../03-efficient-training/README.md) | Directly continues from here — fine-tuning, LoRA, QLoRA, and PEFT techniques |
| [04 — Inference & Deployment](../04-inference-deployment/README.md) | Applies training knowledge to serving decisions — quantization, batching, hardware sizing |

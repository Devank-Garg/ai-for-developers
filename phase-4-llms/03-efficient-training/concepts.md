# Concepts: Efficient Training

> **Tag: core** — Fine-tuning is how you adapt a general model to your use case without training from scratch. This module covers when to do it, how it works, and which techniques to use in 2025.

---

## Fine-tuning vs. prompting — choose first

Before touching any training code, answer this question: **does your problem actually require fine-tuning?**

| Signal | What it suggests |
|---|---|
| You need the model to follow a specific format consistently | Prompting — structured output or few-shot examples often suffice |
| You need the model to know facts it doesn't currently know | RAG (Phase 5), not fine-tuning — fine-tuning is poor at injecting facts reliably |
| You need a specific behaviour, tone, or persona at scale | Fine-tuning — cheaper per-token than long system prompts at volume |
| You need domain-specific reasoning (legal, medical, code style) | Fine-tuning — especially with high-quality domain examples |
| You need to reduce latency or cost by using a smaller model | Fine-tuning a small model to match a larger one's task-specific behaviour |
| You want the model to refuse or allow certain content | Alignment fine-tuning (DPO/GRPO) |

> **Production note:** Most developers reach for fine-tuning too early. A well-written system prompt with few-shot examples solves 80% of "the model isn't behaving how I want" problems. Fine-tune when prompting has genuinely hit its ceiling.

---

## Full fine-tuning and why it's usually impractical

**Full fine-tuning** updates all parameters of the model on your task-specific dataset. For a base model like Llama 3 8B, this means updating 8 billion weights — requiring roughly 120–160 GB of GPU memory and a dataset of hundreds of thousands of examples to avoid catastrophic forgetting.

**Catastrophic forgetting**: when you fine-tune all parameters on a narrow dataset, the model forgets much of what it learned during pre-training. A model fine-tuned heavily on legal contracts may lose the ability to write coherent English prose.

Full fine-tuning is the right choice when:
- You have a very large, high-quality dataset (100K+ examples)
- You have the GPU budget (multiple H100s, days of compute)
- You want to change the model's fundamental behaviour, not just its surface style

For most developers, **Parameter-Efficient Fine-Tuning (PEFT)** is the answer.

---

## LoRA — Low-Rank Adaptation

**LoRA** is the dominant PEFT technique and the most important one to understand.

The core idea: instead of updating the full weight matrix W (which might be 4096 × 4096 = 16 million values), inject two small matrices A and B whose product approximates the update:

```
Original:   W  (4096 × 4096 = 16M parameters)
LoRA:       W + A × B
            where A is (4096 × r) and B is (r × 4096)
            with rank r = 16 → only 2 × (4096 × 16) = 131K parameters
```

During training:
- The original weights W are **frozen** — they receive no gradient updates
- Only A and B are trained — typically 0.1–1% of the total parameter count
- At inference time, A × B is merged into W — **zero added latency**

**Why rank matters:** The rank `r` controls the expressiveness of the update. `r=4` is very constrained (good for style); `r=64` allows more complex changes (better for domain adaptation). Common values: 8–64 for most tasks.

**Where LoRA is applied:** Typically to the attention weight matrices (Q, K, V, and output projections). Some implementations also apply it to MLP layers. The `target_modules` parameter in PEFT controls this.

**LoRA variants in 2025:**
- **LoRA+**: Applies different learning rates to A and B matrices — small improvement in convergence
- **DoRA** (Weight-Decomposed LoRA): Decomposes weights into magnitude and direction components; often outperforms standard LoRA on downstream tasks
- **LoRAFusion**: Running multiple LoRA adapters in parallel for ensemble-like benefits

---

## QLoRA — Quantized LoRA

**QLoRA** (Dettmers et al., 2023) makes LoRA accessible on consumer hardware by quantizing the frozen base model weights to 4 bits while keeping the LoRA adapter in 16-bit.

```
Without QLoRA: Llama 3 70B fine-tuning → ~1 TB VRAM → impossible on a single GPU
With QLoRA:    Llama 3 70B fine-tuning → ~48 GB VRAM → fits on 2× RTX 4090 (24GB each)
               Llama 3 8B fine-tuning  → ~10 GB VRAM → fits on a single RTX 4090
```

**Three key innovations in QLoRA:**

**1. NF4 (Normal Float 4):** A 4-bit data type specifically designed for normally distributed weights. Standard INT4 quantization treats all values equally; NF4 places more quantization levels near zero (where most weights cluster), reducing quantization error.

**2. Double quantization:** The quantization constants themselves are quantized — saving another ~0.5 bits per parameter. Small but meaningful at billions of parameters.

**3. Paged optimizers:** Uses NVIDIA unified memory to page optimizer state between GPU and CPU RAM during gradient computation. Prevents OOM crashes during the brief memory spikes that occur at certain steps.

**QLoRA quality:** Studies consistently show QLoRA achieves 90–95% of full fine-tuning quality at 5–10% of the memory cost. For most practical applications, the gap is negligible.

---

## Other PEFT methods

LoRA/QLoRA are the default for 2025, but other PEFT approaches exist for specific situations:

**Prefix tuning:** Prepends learned "virtual tokens" to the input at every layer. The prefixes are trained; the base model is frozen. Less popular than LoRA — harder to tune, doesn't merge at inference time (adds latency).

**Prompt tuning:** Similar to prefix tuning but applied only to the input embedding layer. Very parameter-efficient (sometimes <0.01% of parameters) but underperforms LoRA on most benchmarks. Useful at very large model scales (100B+) where even LoRA is expensive.

**IA³ (Infused Adapter by Inhibiting and Amplifying Inner Activations):** Injects learned scaling vectors into attention and feed-forward activations. Extremely parameter-efficient — fewer parameters than LoRA — but lower ceiling on adaptation quality.

**Adapter layers:** Insert small neural network modules between transformer layers. Adds inference latency (unlike LoRA). Largely superseded by LoRA for new work.

---

## Supervised Fine-Tuning (SFT)

**SFT** is the first stage of post-training: given a dataset of (prompt, response) pairs, train the model to produce those responses given those prompts. This is the same technique used to turn pre-trained base models into instruction-following assistants.

**Data format — chat template:** Modern SFT uses a chat template that wraps prompts and responses in structured tokens the model recognises:

```
<|system|>You are a helpful assistant.</s>
<|user|>Explain transformers in one paragraph.</s>
<|assistant|>Transformers are neural network architectures that process sequences...
```

The loss is computed **only on the assistant's response** — the prompt tokens are masked. This prevents the model from "learning" to reproduce the prompt itself.

**Data quality trumps quantity:** 1,000 high-quality, diverse, correct examples consistently outperforms 100,000 noisy, redundant, or incorrect ones. The Alpagasus paper showed that filtering a 52K dataset down to 9K high-quality examples improved performance. "Fewer, better" is the consistent finding.

**Dataset size rules of thumb (2025):**

| Task | Minimum examples | Notes |
|---|---|---|
| Style / format adaptation | 500–2,000 | Very achievable |
| Domain specialisation | 5,000–20,000 | With high-quality examples |
| Task-specific behaviour | 10,000–50,000 | Standard fine-tuning range |
| Instruction following (general) | 100,000+ | Full SFT for assistant behaviour |

---

## DPO — Direct Preference Optimization

RLHF (covered in module 01) requires training a separate reward model and running a full RL loop — complex, unstable, and compute-intensive. **DPO** (Rafailov et al., 2023) sidesteps all of this.

**How DPO works:** Instead of learning a reward model, DPO directly trains the model using preference pairs — (prompt, chosen_response, rejected_response). A binary cross-entropy-style loss increases the probability of the chosen response and decreases the probability of the rejected response, relative to a reference model.

```
Training data format:
{
  "prompt":    "Explain how vaccines work",
  "chosen":    "Vaccines work by introducing...",   ← preferred by humans
  "rejected":  "I'm not sure, vaccines are...",     ← less preferred
}
```

**Why DPO over RLHF:**
- No reward model to train and maintain
- No RL instability (PPO reward hacking, mode collapse)
- Faster and cheaper — can run on the same hardware as SFT
- Competitive quality with RLHF on most tasks

**DPO is now standard** for preference alignment at companies that can't afford OpenAI-scale RLHF infrastructure. Claude, Llama, and Mistral models all use variants of preference optimization in their post-training pipelines.

---

## GRPO — The 2025 Standard for Reasoning Models

**GRPO** (Group Relative Policy Optimization), introduced by DeepSeek, has rapidly become the preferred method for training reasoning-focused models. It was central to DeepSeek-R1's capabilities.

**The problem GRPO solves:** Standard DPO requires pre-written preference pairs. For complex reasoning tasks (math, code), generating those pairs is expensive and requires knowing the correct answer in advance. GRPO instead samples multiple responses per prompt, scores them (e.g., by checking against a verifiable ground truth), and optimizes relative to the group's average reward — no critic model needed.

```
For a math problem:
  Sample 8 responses → check which are correct
  Correct responses: reward = +1
  Incorrect responses: reward = −1
  Update: increase probability of correct, decrease of incorrect
  (relative to group average, not absolute reference)
```

**Why GRPO over PPO:** PPO requires a separate value/critic network (expensive). GRPO uses the group of sampled responses as its own baseline — simpler, more stable, cheaper.

**When to use GRPO:** When your task has a verifiable ground truth (math problems, code that can be executed, logic puzzles). When ground truth requires human judgment, DPO remains more practical.

---

## Practical fine-tuning frameworks in 2025

| Framework | Best for | Key differentiator |
|---|---|---|
| **Unsloth** | Single-GPU, fast iteration | 2–5× faster than baseline, 80% less VRAM; custom CUDA kernels |
| **LLaMA-Factory** | Beginners, no-code experimentation | Web UI, supports SFT/DPO/GRPO/PPO, 100+ model architectures |
| **Axolotl** | Production, complex configs | Flexible YAML config, strong community, supports all PEFT methods |
| **TRL (Hugging Face)** | Integration with HF ecosystem | Standard library for SFT, DPO, GRPO, PPO; well-maintained |
| **torchtune (Meta)** | PyTorch-native, full control | Native PyTorch 2.x, FSDP2, reference recipes for all post-training |

**For getting started:** LLaMA-Factory (web UI, no code) or Unsloth (fastest on a single GPU) are the lowest-friction entry points. For production pipelines, Axolotl or TRL.

---

## When fine-tuning fails — common mistakes

**Using fine-tuning to inject facts:** Fine-tuning is poor at memorising facts reliably. A model fine-tuned on "our product costs $49/month" will sometimes say $49, sometimes hallucinate a different price. Use RAG for fact retrieval; use fine-tuning for behaviour and style.

**Dataset quality neglect:** Noisy, inconsistent, or incorrect examples in training data produce noisy, inconsistent outputs. Curating 1,000 excellent examples takes longer than collecting 100,000 bad ones — but produces far better results.

**Overfitting to training format:** A model fine-tuned on a narrow set of prompt styles learns to expect exactly those styles. If production prompts look different, quality drops. Ensure training data is diverse enough to cover real usage.

**Forgetting to evaluate:** Fine-tuned models should be evaluated systematically — not just "it looks good in a few examples." Use a held-out eval set, track metrics before and after, and compare to the base model on tasks you want to preserve.

**Rank too high:** A LoRA rank of 128+ on a small dataset causes overfitting. Start with r=16, increase only if you have a large, diverse dataset and can afford the parameters.

---

## Summary

| Concept | Key point |
|---|---|
| Fine-tune vs. prompt | Try prompting first. Fine-tune for consistent behaviour, domain reasoning, or cost reduction — not fact injection |
| Full fine-tuning | Updates all weights; risks catastrophic forgetting; needs large dataset and GPU budget |
| LoRA | Injects low-rank update matrices; freezes base weights; 0.1–1% of parameters; zero inference latency after merging |
| QLoRA | 4-bit quantized base model + 16-bit LoRA; enables 70B fine-tuning on 2× consumer GPUs |
| SFT | Supervised training on (prompt, response) pairs; quality beats quantity; loss only on response tokens |
| DPO | Preference optimization without a reward model; uses (prompt, chosen, rejected) triples; replaces RLHF for most use cases |
| GRPO | Samples multiple responses, scores by ground-truth, optimizes relative to group average; standard for reasoning model training |
| Unsloth | Fastest single-GPU fine-tuning framework; 2–5× speed, 80% VRAM reduction |
| LLaMA-Factory | No-code web UI for fine-tuning; supports 100+ architectures and all major methods |
| Data quality | 1K excellent examples > 100K noisy ones; consistent finding across all fine-tuning research |

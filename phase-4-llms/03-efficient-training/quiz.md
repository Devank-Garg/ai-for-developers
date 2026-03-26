# Quiz: Efficient Training

Attempt these before checking answers. The goal is to surface gaps that will affect real decisions — model selection, fine-tuning strategy, budget estimation.

---

## Questions

**1.** A product manager asks you to fine-tune a model on the company's internal knowledge base (5,000 documents about products, policies, and procedures) so users can ask questions and get accurate answers. Is fine-tuning the right approach? What would you recommend instead, and why?

---

**2.** A 7B model in BF16 has approximately 14 GB of weights. Estimate the total GPU memory required to full fine-tune this model using the Adam optimizer. Why is the actual number so much larger than just the weights?

---

**3.** LoRA uses a rank `r` to control the expressiveness of the weight update. A team is fine-tuning a model for a very specific customer support tone (consistent style, fixed greeting, fixed sign-off). Another team is doing domain adaptation for legal reasoning across many document types. Which team should use a higher rank, and why?

---

**4.** Explain what QLoRA's NF4 data type is and why it outperforms standard INT4 for quantizing LLM weights. What property of LLM weight distributions makes NF4 the better choice?

---

**5.** You are running QLoRA on a Llama 3 8B model on a single RTX 4090 (24 GB VRAM). Halfway through training, you get an OOM (out of memory) error. List three specific QLoRA/training settings you would adjust, in order of how much memory they save.

---

**6.** What is catastrophic forgetting? Describe a concrete scenario where it would occur, and explain why LoRA/QLoRA largely avoids it while full fine-tuning does not.

---

**7.** Compare SFT and DPO:

a) What does each require as training data?
b) What does each optimise for?
c) When would you use one versus the other?

---

**8.** DeepSeek used GRPO to train DeepSeek-R1's reasoning capabilities. A team wants to apply the same technique to improve a model's ability to solve Python coding problems. What does GRPO require that makes it applicable here, and what would you use as the reward signal?

---

**9.** A colleague proposes this fine-tuning dataset: scrape 200,000 customer support chat logs from your ticketing system and use them as-is for SFT. What are three problems with this approach and how would you address each?

---

**10.** You want to fine-tune a model so it always responds in a specific JSON format with fields `{action, confidence, reasoning}`. After SFT on 500 examples, the model mostly follows the format but occasionally adds extra fields or wraps the JSON in markdown code blocks. What is the likely cause and what two techniques would fix it?

---

**11.** Rank these fine-tuning frameworks by their fit for each scenario, and justify each choice:

- **Scenario A:** A solo developer with one RTX 3090 (24 GB) wants to fine-tune Llama 3 8B as quickly as possible to test an idea over a weekend
- **Scenario B:** A team without deep ML knowledge wants a no-code interface to experiment with SFT and DPO on a 13B model
- **Scenario C:** A production ML team needs a reproducible, config-driven pipeline for QLoRA + DPO on 70B models across multiple cloud GPU nodes

---

**12.** After fine-tuning a model for medical triage classification, you notice it performs well on the fine-tuning eval set but produces incoherent English in tasks unrelated to medical classification (e.g., general summarisation). What has happened and what would you do differently?

---

## Answers

**1.** Fine-tuning is the wrong tool here. Fine-tuning is poor at injecting facts reliably — a model trained on product documentation will still hallucinate prices, policy details, and specific procedures, especially as the knowledge base changes over time.

The right approach is **Retrieval-Augmented Generation (RAG)**: store the documents in a vector database, retrieve relevant chunks at query time, and inject them into the prompt. RAG gives the model accurate, up-to-date context at inference time rather than baking facts into weights during training.

Fine-tuning could complement RAG later — for example, to teach the model the specific tone and format expected in responses — but it should not be the primary mechanism for knowledge access.

---

**2.** A 7B model in BF16 (2 bytes/param) = **14 GB** for weights.

Full fine-tuning with Adam requires:
- **Weights (BF16):** 14 GB
- **Gradients (BF16):** 14 GB — same size as weights
- **Adam first moments (FP32):** 28 GB — one FP32 copy per parameter
- **Adam second moments (FP32):** 28 GB — another FP32 copy per parameter
- **Activations:** Variable — depends on batch size and sequence length; often 10–40 GB

**Minimum (weights + gradients + optimizer state):** ~84 GB — more than a single H100 80 GB can hold, before even considering activations. In practice, full fine-tuning a 7B model requires 2–4 GPUs.

The optimizer state is the largest component most people don't account for. Adam's two momentum terms cost 4× the weight storage in FP32 — this is exactly what ZeRO and QLoRA solve by sharding or quantizing the optimizer state.

---

**3.** The **legal reasoning team** should use the higher rank.

Style adaptation (consistent tone, fixed greetings) is a low-dimensional change — it affects surface-level token patterns and requires very few degrees of freedom. A rank of 4–8 is often sufficient.

Domain adaptation for legal reasoning requires the model to learn new relationships between concepts, new reasoning patterns, and domain-specific interpretation rules. This is a higher-dimensional change across many weight matrices. A rank of 32–64 (or higher, with a large dataset) gives the adapter enough expressiveness to capture these complex changes.

The practical test: if you're changing *what* the model knows or *how* it reasons, use higher rank. If you're changing *how* it sounds, use lower rank.

---

**4.** Standard INT4 divides the weight value range into 16 equally-spaced buckets. LLM weights are approximately **normally distributed** — most weights cluster near zero with the density falling off toward the tails. Equally-spaced buckets waste most of their capacity on sparse regions (the tails) and have too few buckets near zero where most weights live.

**NF4 (Normal Float 4)** places its 16 quantization levels according to the quantiles of a standard normal distribution — more levels near zero where weight density is highest, fewer in the tails. This minimises quantization error for normally distributed weights. The result: lower average distance between the true weight and its quantized approximation compared to INT4, at the same 4-bit budget.

The key property NF4 exploits is that pre-trained LLM weights consistently follow an approximately normal distribution — a property that holds because of how gradient-based training converges.

---

**5.** Three adjustments, ordered by memory impact:

**1. Enable gradient checkpointing (saves the most — 40–60% of activation memory):**
```python
model.gradient_checkpointing_enable()
```
Recomputes activations during the backward pass instead of storing them. Largest single saving for long sequences.

**2. Reduce batch size and/or sequence length:**
Each token in the sequence at each layer produces activations proportional to batch_size × seq_len × hidden_dim. Cutting batch size from 4→2 or max sequence length from 2048→1024 halves activation memory. Use gradient accumulation (accumulate_grad_batches=4) to maintain effective batch size without memory cost.

**3. Lower the LoRA rank or reduce target modules:**
```python
lora_config = LoraConfig(r=8, ...)  # down from r=32
```
Fewer trainable parameters = smaller gradient tensors. Also consider removing LoRA from the MLP layers (target only attention Q/K/V) if you added it everywhere — this is a smaller saving than the above two but may be enough to unblock.

---

**6.** **Catastrophic forgetting** occurs when fine-tuning updates all model weights on a narrow task dataset, overwriting the broadly distributed representations learned during pre-training.

**Concrete scenario:** A 7B model is fully fine-tuned on 50,000 examples of Python debugging conversations. After training, it excels at Python debugging — but when asked to write an email, it produces syntactically plausible but contextually inappropriate responses, mixes in Python-like phrasing, and loses the general writing ability it had before fine-tuning. The weights that encoded general language patterns were overwritten by patterns that reinforce Python debugging responses.

**Why LoRA/QLoRA avoids this:** LoRA keeps the original weights W **frozen** — they are never modified. Only the low-rank matrices A and B are updated. The original representations are completely preserved. At most, the LoRA update adds a small adjustment on top of the frozen weights. The base model's general capabilities remain intact; LoRA only changes what the combined W + AB produces for the specific task.

---

**7.**

**a) Training data:**
- **SFT:** (prompt, response) pairs — you need examples of the output you want the model to produce
- **DPO:** (prompt, chosen_response, rejected_response) triples — you need examples of preferred and dispreferred outputs for the same prompt

**b) What each optimises:**
- **SFT:** Maximises log-likelihood of the chosen response — teaches the model to produce that exact type of output
- **DPO:** Maximises the probability of the chosen response *relative to* the rejected response — teaches the model to prefer one type of output over another

**c) When to use each:**
- **SFT first:** When the base model doesn't yet produce the right format or behaviour at all — SFT establishes the baseline behaviour
- **DPO after SFT:** When the base behaviour is right but you want to steer preferences (safer, more helpful, better-formatted, more concise) — DPO is more sample-efficient for preference shifts than rewriting the training set
- **SFT only:** When you have high-quality reference outputs and preference pairs would be expensive to generate
- **DPO only:** When you have preference data (e.g., human rankings of model outputs) and a reasonably capable base model

In practice, production fine-tuning pipelines typically run SFT → DPO in sequence.

---

**8.** GRPO requires a **verifiable reward signal** — a function that can automatically judge whether a response is correct or not, without human annotation.

Python coding problems are ideal: the reward signal is **code execution**. Given a problem with test cases, the model generates a solution, you execute it against the test cases, and the reward is:
- +1 if all test cases pass
- 0 or −1 if tests fail (with partial credit possible for passing some tests)

GRPO samples multiple candidate solutions per problem (e.g., 8), executes each, and optimises the model to produce more solutions like the ones that passed — relative to the group's average reward. No human labelling is required; the test cases are the reward model.

This is exactly the approach used for math (verify against ground-truth answer) and code (verify via execution) — both domains where GRPO has produced strong results.

---

**9.** Three problems with using raw chat logs as-is:

**Problem 1: Agent responses may be poor quality.** Chat logs contain real support agent responses, which may be incorrect, inconsistent, incomplete, or poorly worded. Training on these teaches the model to reproduce bad responses, not ideal ones. **Fix:** Filter or rewrite responses — use LLM-based quality scoring to keep only high-quality exchanges, or have subject matter experts review and revise a representative sample.

**Problem 2: Privacy and PII.** Customer support logs contain names, account numbers, email addresses, order details, and other PII. Training a model on this data risks the model memorising and reproducing sensitive customer information. **Fix:** Run a PII detection and redaction pipeline before training. Consult legal/compliance on whether these logs can be used for model training at all.

**Problem 3: Distribution mismatch — logs contain multi-turn context that gets lost.** Single exchanges extracted from multi-turn conversations may lack the context that makes the response sensible. A response of "Yes, that's right" is uninformative without the preceding question. **Fix:** Include the relevant prior turns as context in each training example, or filter out exchanges that are incomprehensible without prior context.

---

**10.** **Likely cause:** Inconsistency in the training examples themselves. If some of the 500 examples wrapped JSON in markdown code blocks and others didn't, the model learned both patterns. Similarly, if the training data had any variation in the field set, the model learned that variation too.

**Fix 1 — Audit and normalise the training data:** Go through all 500 examples and ensure the JSON format is 100% consistent — no markdown wrapping, exactly the three fields, consistent whitespace. Even a handful of inconsistent examples at 500 total is enough to cause this.

**Fix 2 — Use structured output / JSON mode at inference time:** Most APIs provide a mechanism to constrain model output to valid JSON (Anthropic's tool use / structured output, OpenAI's `response_format`). This enforces the schema at inference time regardless of what the model "wants" to produce, eliminating the markdown wrapping issue entirely. Use both fixes together: clean training data for consistent learning + constrained decoding for robust output.

---

**11.**

**Scenario A (solo developer, single RTX 3090, speed priority):**
→ **Unsloth** — delivers 2–5× faster training and 80% less VRAM than baseline on a single GPU. Specifically designed for exactly this use case. QLoRA on Llama 3 8B will fit comfortably on 24 GB with Unsloth, and the weekend timeline makes training speed the priority.

**Scenario B (non-ML team, no-code, SFT + DPO experiments):**
→ **LLaMA-Factory** — provides a web UI where users can configure training runs, upload datasets, and launch SFT/DPO jobs without writing code. Supports 100+ model architectures and all major training methods through a GUI. Lowest barrier for teams without ML engineering expertise.

**Scenario C (production team, config-driven, multi-node 70B):**
→ **Axolotl** — YAML-driven configuration for reproducible pipelines, strong community support for complex setups, integrates with DeepSpeed for multi-node distributed training. Well-suited for production engineering teams who need versioned, testable, deployable training configs. TRL (Hugging Face) is a close alternative if the team is HF-ecosystem-native.

---

**12.** **Catastrophic forgetting** — the model's general language capabilities were damaged by fine-tuning. With a narrow medical classification dataset and (likely) full fine-tuning or a LoRA rank too high for the dataset size, the weight updates overwrote representations needed for general English generation.

**What to do differently:**

1. **Switch to LoRA/QLoRA instead of full fine-tuning** — keep base weights frozen; the LoRA adapter adds classification capability without modifying the general language backbone

2. **Lower the LoRA rank** — classification is a low-dimensional task; rank 8–16 is sufficient and prevents overfitting to the narrow domain

3. **Add general-purpose examples to the training mix** — include a small proportion of diverse, general instruction-following examples alongside medical classification examples; this regularises the fine-tuning and preserves general capabilities

4. **Evaluate on held-out general tasks** — add a general capability eval (summarisation, QA, writing) alongside the domain eval to detect forgetting early, before deploying

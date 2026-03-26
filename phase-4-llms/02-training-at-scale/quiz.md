# Quiz: Training at Scale

Attempt these questions before checking the answers. The goal is to expose gaps before they cause confusion later — especially when reading about model releases, benchmarks, or infrastructure choices.

---

## Questions

**1.** The Chinchilla paper's central finding is often summarised as "train smaller models on more data." What does this mean in practice? Why would you choose a 7B model over a 70B model for a given compute budget?

---

**2.** A colleague claims: "GPT-3 is the most capable model for its size because it has 175 billion parameters." What is wrong with this reasoning, and what metric would you need to compare fairly?

---

**3.** You need to store a 70B parameter model for training in BF16. Calculate how much GPU memory the weights alone require. A single H100 has 80 GB of memory. How many H100s are needed just to hold the weights — before accounting for gradients or optimizer state?

---

**4.** When using the Adam optimizer with data parallelism, each GPU holds a full copy of:

a) Only the model weights
b) The model weights and gradients
c) The model weights, gradients, and optimizer state (first + second moment estimates)
d) Only the optimizer state

---

**5.** You are designing a distributed training setup for a 13B parameter model. You have 8 GPUs, each with 80 GB memory. Describe which parallelism strategy (data, tensor, or pipeline) you would start with, and why. What would change if the model were 400B parameters?

---

**6.** What is the "pipeline bubble" in pipeline parallelism? Describe what causes it and how micro-batching reduces it.

---

**7.** ZeRO Stage 3 is configured with 64 GPUs. A training step requires one of the GPUs to compute a forward pass through layers whose weights are sharded across all 64 GPUs. What happens to those weights at runtime, and what is the tradeoff compared to data parallelism?

---

**8.** During a training run on a 40B model, the loss suddenly jumps from 2.4 to 4.1 before recovering over the next 500 steps. What are two likely causes, and what two safeguards should have been in place?

---

**9.** A team is training a model in FP16 and keeps encountering NaN losses after several thousand steps. Switching to BF16 resolves the issue. Explain why, in terms of how the two formats differ.

---

**10.** You're evaluating two open-weight models:
- **Model A:** 13B parameters, trained on 1T tokens
- **Model B:** 7B parameters, trained on 14T tokens

Neither has been fine-tuned. All else being equal, which would you expect to perform better on reasoning benchmarks and why? What is the approximate Chinchilla-optimal token count for each model size?

---

**11.** Gradient checkpointing reduces activation memory by 60–70%. What is the cost? If a training step normally takes 10 seconds without checkpointing, approximately how long would it take with checkpointing enabled?

---

**12.** A startup wants to train a domain-specific model from scratch on 500 GB of proprietary text data. A senior engineer suggests fine-tuning an existing open-weight model instead. What is the strongest argument in favour of the engineer's suggestion, grounded specifically in what you learned about training costs?

---

**13.** Your team is building a training data pipeline. Rank these steps in the order they should typically occur, and explain the risk of getting the order wrong:

- Quality filtering (heuristics + classifiers)
- Deduplication
- Language filtering
- PII scrubbing
- Mixture ratio tuning

---

## Answers

**1.** For a fixed compute budget (a fixed number of GPU-hours), Chinchilla showed that you get better model performance by allocating that budget more evenly between model size and training data — rather than maximising parameters while skimping on data. Most pre-Chinchilla models were undertrained: GPT-3 at 175B parameters was trained on only ~300B tokens, when the Chinchilla-optimal amount for that size is roughly 3.5 trillion.

Choosing a 7B model means you can train it on 14× more tokens for the same compute. The resulting 7B model often outperforms the 70B model trained on 1T tokens at many tasks — because it has seen far more diverse data. Additionally, the 7B model is much cheaper to run at inference time: it fits on a single GPU, processes requests faster, and costs a fraction as much per query. Training cost is paid once; inference cost is paid every time a user makes a request.

---

**2.** Parameter count alone is not a meaningful comparison. A 175B model trained on 300B tokens is undertrained compared to a 7B model trained on 2T tokens by Chinchilla standards. The right comparison includes: number of training tokens, compute budget (FLOPs), and benchmark performance. GPT-3 was undertrained by Chinchilla metrics — the same compute budget, redistributed to a smaller model and more tokens, would have produced a better-performing model. "Largest parameter count" is marketing; "best benchmark performance per compute dollar" is engineering.

---

**3.** 70B parameters × 2 bytes per parameter (BF16) = **140 GB** for weights alone.

140 GB ÷ 80 GB per H100 = **2 H100s minimum** just for weights.

But training also requires:
- Gradients: another 140 GB (same size as weights in BF16)
- Adam optimizer state (FP32): 70B × 4 bytes × 2 moments = 560 GB

Total: ~840 GB. At 80 GB per H100, that's **~11 H100s minimum** before accounting for activations or batch size. In practice, training a 70B model typically requires 16–32 H100s or equivalent.

---

**4.** **c)** The model weights, gradients, and optimizer state (first + second moment estimates).

In standard data parallelism, every GPU holds the complete model replica. During the backward pass, each GPU accumulates its own gradient copy. The AllReduce operation averages these gradients — but does not eliminate the per-GPU storage. Adam stores first and second moment estimates for every parameter in FP32, which is 8 bytes per parameter on top of the 2-byte BF16 weights. This is exactly the redundancy that ZeRO is designed to eliminate.

---

**5.** For a 13B model with 8× 80 GB GPUs:

13B × 16 bytes (full training state with Adam) ≈ ~208 GB. Total available: 8 × 80 GB = 640 GB. The model fits with room for activations. Start with **data parallelism** (PyTorch FSDP or DDP + ZeRO Stage 2). It's the simplest strategy, requires the least inter-GPU communication overhead, and the model fits within the available memory budget.

For a 400B model: 400B × 16 bytes ≈ 6.4 TB total training state. 8 GPUs × 80 GB = 640 GB — nowhere near enough even for the weights alone. You would need to combine:
- **Tensor parallelism** within each server node (splitting weight matrices across GPUs)
- **Pipeline parallelism** across nodes (assigning layer groups to different servers)
- **ZeRO Stage 3** to shard the optimizer state

This is the "3D parallelism" approach used for frontier model training.

---

**6.** In pipeline parallelism, the model is split so GPU 1 handles layers 1–12, GPU 2 handles layers 13–24, and so on. GPU 2 cannot start processing until GPU 1 has finished its forward pass and passed the activations. At the start of each batch: GPU 2, 3, and 4 sit idle while GPU 1 processes. At the end: GPU 1 is idle while downstream GPUs finish. This idle time is the **pipeline bubble** — wasted compute on expensive hardware.

**Micro-batching** splits the batch into smaller chunks (micro-batches). While GPU 2 processes micro-batch 1, GPU 1 can immediately start on micro-batch 2. The pipeline stays filled with work, reducing idle time. More micro-batches = smaller bubble, but also more communication overhead per step. The optimal number of micro-batches is a tunable hyperparameter.

---

**7.** With ZeRO Stage 3, each GPU holds only 1/64 of each weight tensor. When GPU N needs to run a forward pass through a layer, it must **gather** the full weight tensor from all 64 GPUs, compute, then discard the full weights (keeping only its own 1/64 shard). This happens at every layer of every forward pass.

The tradeoff vs. data parallelism:
- **Memory:** ZeRO Stage 3 reduces per-GPU weight memory by ~64× — enabling models that couldn't otherwise fit
- **Communication:** Every layer requires an AllGather across 64 GPUs before computing and a ReduceScatter after. Communication volume is much higher than data parallelism's single AllReduce per step
- **Speed:** Training is slower per step due to communication overhead; this is acceptable when the alternative is the model not fitting at all

---

**8.** Two likely causes of a loss spike:
1. **A corrupted or anomalous batch** — a batch containing unusually long sequences, malformed text, or extreme token distributions can cause an abnormally large gradient update
2. **Learning rate too high** for the current point in training — causing parameter updates that overshoot

Two safeguards that should have been in place:
1. **Gradient clipping** — capping gradient norm (typically at 1.0) prevents any single batch from causing catastrophically large weight updates. The spike would have been dampened or eliminated
2. **Frequent checkpointing** — saving model state every few hundred steps means that if the loss spike causes divergence, the run restarts from a checkpoint rather than from scratch. Without this, a 2-week training run can be lost to a single hardware failure or loss spike that doesn't recover

---

**9.** The root cause is **dynamic range**. FP16 allocates 5 bits to the exponent, giving it a maximum representable value of ~65,504. BF16 allocates 8 bits to the exponent (same as FP32), giving it a range up to ~3.4 × 10³⁸.

During training, gradients can be very large (especially early in training or during loss spikes). If a gradient value exceeds FP16's maximum (~65,504), it overflows to `inf`. Inf values propagate through subsequent operations: `inf × weight = inf`, `inf - inf = NaN`. Once NaN appears in the computation graph, it contaminates everything and the training is corrupted.

BF16 accommodates the same large gradient magnitudes as FP32, so overflow doesn't occur. The tradeoff is lower precision in the mantissa (7 bits vs. FP16's 10 bits), but for training this matters less than range stability. FP16 requires **automatic loss scaling** (scaling gradients to a smaller range before backward, then unscaling) to avoid this problem; BF16 does not.

---

**10.** **Model B** (7B, trained on 14T tokens) would be expected to perform better on most benchmarks, all else equal.

The Chinchilla-optimal token count is approximately 20 tokens per parameter:
- Model A (13B): optimal ≈ 13B × 20 = **260B tokens**. It was trained on 1T tokens — already significantly over-trained by compute standards, but not exceptionally so.
- Model B (7B): optimal ≈ 7B × 20 = **140B tokens**. Trained on 14T tokens — over 100× the compute-optimal amount.

Model B has seen vastly more data diversity. It has encountered more language patterns, more domains, more reasoning examples. For a model this over-trained, the sheer data volume tends to dominate raw parameter count at downstream tasks. This is the Llama 3 strategy: the 8B model trained on 15T tokens was designed to outperform much larger but undertrained competitors at inference time, accepting higher training cost in exchange for cheap, capable deployment.

---

**11.** The cost of gradient checkpointing is approximately **33% additional forward-pass compute**. Without checkpointing, the backward pass uses stored activations. With checkpointing, the backward pass must recompute activations for the uncheckpointed layers — effectively running a partial forward pass again.

If a training step normally takes 10 seconds:
- Forward pass: ~3.3 seconds
- Backward pass (using stored activations): ~6.7 seconds
- Total: 10 seconds

With checkpointing:
- Forward pass: ~3.3 seconds
- Backward pass (recomputing activations): ~6.7 + ~3.3 seconds = ~10 seconds
- Total: ~13.3 seconds — roughly **33% slower**

The tradeoff is accepted when the memory savings are the difference between a run fitting in memory and OOM (out of memory) errors. In practice, checkpointing is used almost universally in large model training.

---

**12.** The strongest argument is **compute cost and data efficiency**. Training from scratch means learning everything — every word, every grammar pattern, every common reasoning structure — from your 500 GB of proprietary data. You would need trillions of tokens (far more than 500 GB provides) to teach a model the fundamentals of language before it can even begin to specialise in your domain.

An existing open-weight model (e.g., Llama 3 8B, trained on 15 trillion tokens) already encodes all of that general language knowledge in its weights. Fine-tuning teaches it your domain on top of what it already knows — requiring far less data and compute to reach the same domain-specific performance. The proprietary data becomes the differentiator, not the foundation. Training from scratch with 500 GB of data would likely produce a model that's inferior to fine-tuning an 8B base model, at dramatically higher cost.

---

**13.** Recommended order and rationale:

| Step | Order | Why here |
|---|---|---|
| Language filtering | 1st | Eliminate non-target-language documents before any expensive processing. No point deduplicating or classifying text you'll discard anyway. |
| Deduplication | 2nd | Remove near-duplicates early. If you quality-filter first, you may keep many copies of a high-scoring document — wasting compute on training duplicates that cause memorisation. |
| Quality filtering | 3rd | Now that duplicates are gone, classify the remaining documents. Running classifiers on duplicates you'll later deduplicate wastes inference compute. |
| PII scrubbing | 4th | Apply after quality filtering — PII scrubbing tools have false positive rates; running them on low-quality documents that will be discarded wastes effort. Scrub the final retained corpus. |
| Mixture ratio tuning | 5th | Once each source corpus is clean, decide how much of each to include in training. This is a global decision that depends on final corpus sizes — you can't tune ratios until you know what you have. |

Getting the order wrong risks: running expensive classifiers on documents you'll discard (wrong step 3 placement), training on duplicated data that causes memorisation (dedup too late), or exposing PII in training data (scrubbing too early and missing later-added documents).

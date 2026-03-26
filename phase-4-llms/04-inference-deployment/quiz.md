# Quiz: Inference & Deployment

Attempt these before checking answers. These questions reflect real infrastructure decisions you'll face when taking a model to production.

---

## Questions

**1.** A user sends a 2,000-token prompt to your LLM-powered application. The model generates a 50-token response. Explain the difference in how these two phases (processing the 2,000 tokens and generating the 50 tokens) are executed by the GPU, and why their performance characteristics are fundamentally different.

**2.** Your production LLM API is returning the first token very quickly (< 200ms) but the full response takes 30+ seconds. Meanwhile, another application returns the first token after 5 seconds but completes the full response in 8 seconds. What is the most likely cause of each pattern, and which metric would you focus on improving for a user-facing chat application?

**3.** A 7B model in BF16 requires ~14 GB of VRAM. You quantize it to AWQ 4-bit, reducing it to ~4 GB. The GPU now has 10 GB free. Why can't you simply run 2.5× as many concurrent requests with this extra memory?

**4.** Explain what the KV cache is, why it grows with sequence length, and what happens to serving capacity when you increase max context from 4K to 128K tokens.

**5.** You're comparing GPTQ and AWQ for quantizing a 13B model to 4-bit for GPU inference. Walk through how each works and which you'd choose, and why.

**6.** A developer is building a local AI assistant that must run entirely on their laptop (16 GB RAM, no dedicated GPU). They want to run a 7B model. What file format, runtime, and quantization level would you recommend, and what quality tradeoffs should they expect?

**7.** Explain PagedAttention using an analogy from operating systems. What problem does it solve, and what is the concrete memory efficiency improvement over naive KV cache allocation?

**8.** You have a vLLM deployment serving a coding assistant. 90% of requests share the same 3,000-token system prompt containing coding guidelines and examples. Request throughput is lower than expected. What optimization would you apply, what does it do, and what cost reduction should you expect?

**9.** Speculative decoding claims to achieve 2–3× throughput improvement without changing output quality. A colleague is skeptical: "If the draft model gets tokens wrong, doesn't that corrupt the output?" Explain how speculative decoding maintains output fidelity despite using a smaller, less accurate draft model.

**10.** Your company is considering building a self-hosted LLM infrastructure to serve a 70B model. Current API costs are $15,000/month. A cloud provider offers H100 80GB instances at $4/hour. Build the minimum case for self-hosting: what hardware do you need, what is the monthly cost, and at what API cost does self-hosting break even?

**11.** A team is building a real-time voice assistant using an LLM. They require total response latency under 800ms (from speech end to first audio word). They plan to use a 70B model API. What are the latency components in this pipeline, and is 800ms achievable? What architectural changes would make it feasible?

**12.** Rank these serving scenarios from highest to lowest batch size recommendation, and explain your reasoning:

- Batch A: 10 users each sending 50,000-token document analysis requests
- Batch B: 10,000 users sending 100-token chat messages
- Batch C: 1 user sending 500-token requests, expecting a 5,000-token response

---

## Answers

**1.** The 2,000-token prompt is processed in **prefill**: all tokens are processed in parallel in a single forward pass. The GPU performs large matrix multiplications across the entire sequence simultaneously — this is compute-intensive but GPUs excel at parallel matrix math. It is fast relative to the number of tokens processed.

The 50-token response is generated in **decode**: tokens are produced one at a time, each requiring a full sequential forward pass. At each step, the GPU must load all model weights from memory, perform the forward pass, and produce one token — then repeat. This is **memory-bandwidth-bound**: the GPU is mostly waiting for weights to arrive from VRAM, not computing. Even a faster GPU with more FLOPS won't help much; what matters is memory bandwidth.

The fundamental difference: prefill parallelises across tokens (one pass for 2,000 tokens); decode is sequential (2,000 passes for 2,000 tokens). This is why input tokens are cheaper than output tokens in most API pricing.

---

**2.** **Application 1 (fast TTFT, slow completion):** The model is generating a very long response, or the GPU is being shared and the decode phase is slow due to insufficient GPU memory bandwidth for the request volume. The fast TTFT confirms the model received and processed the prompt quickly (prefill is fast). The slow completion reflects a decode bottleneck — likely serving multiple large requests simultaneously, causing each to compete for the same memory bandwidth, or simply generating many tokens.

**Focus metric:** For a user-facing chat application, **TTFT** is the primary metric because it determines when the user sees the first word — the perceived responsiveness. A fast TTFT with streaming output feels responsive even if total completion takes 10 seconds. Focus on TTFT for user experience; optimise throughput (total tokens/second across all users) for cost.

**Application 2 (slow TTFT, fast completion):** A 5-second TTFT with a short completion suggests a long or expensive prefill — either the prompt is very long (tens of thousands of tokens), the system prompt is large, or the server is queuing requests and this one waited.

---

**3.** KV cache is the hidden cost. At runtime, each active request consumes GPU memory for:

1. **Model weights** (now reduced to ~4 GB)
2. **KV cache per active request**: stores key and value tensors for every token processed, at every attention layer

KV cache size = `batch_size × seq_len × num_layers × num_heads × head_dim × 2 (K+V) × bytes_per_element`

For a 7B model at 4,096 tokens per request, each concurrent request consumes roughly 1–2 GB of KV cache in BF16. The freed VRAM from quantization goes to hosting more concurrent requests — but each new request immediately starts consuming KV cache memory. The relationship between freed VRAM and concurrent request capacity is not linear.

Additionally, activations during prefill are temporarily memory-intensive, creating peak usage spikes that must be budgeted for. The 10 GB freed is real, but each additional concurrent long-context request consumes it quickly.

---

**4.** During generation, the attention mechanism at each step needs the key (K) and value (V) vectors from every token generated so far. Without caching, you'd reprocess the entire sequence at every step — O(n²) cost for an n-token sequence.

The **KV cache** stores these K and V tensors after computing them, so each new token only needs to compute its own K/V and then attend to the cached vectors from previous tokens. Cost per step: O(n) rather than O(n²).

**Why it grows with sequence length:** Each token added to the sequence adds one row to the K cache and one row to the V cache, for every attention layer. A 4K-token sequence stores 4× as much KV data as a 1K-token sequence.

**Impact of 4K → 128K context expansion:** KV cache memory per request grows by 32× (128K ÷ 4K). If a 4K request consumed 500 MB of KV cache, a 128K request consumes ~16 GB — nearly the entire VRAM of a consumer GPU. This means:
- Far fewer concurrent requests per GPU
- Much higher per-request cost
- Possible OOM on hardware that handled 4K fine

Long-context serving requires either significantly more GPU memory, aggressive KV cache compression techniques, or accepting lower concurrency.

---

**5.** **GPTQ** (post-training quantization): runs a calibration dataset through the model and uses the Hessian of the loss to determine which weights to quantize aggressively and which to protect. It compresses layer by layer, minimising the output error of each layer individually. Fast to quantize (hours for a 13B model); well-supported; quality is good but not state-of-the-art.

**AWQ** (Activation-Aware Weight Quantization): analyses which weights have high activation magnitudes (i.e., which weights are most "used" by the model's computations) and protects those weights by scaling their channel before quantization. The key insight: a small fraction of weights (those with large corresponding activations) contribute disproportionately to output quality. Protecting them while aggressively quantizing the rest achieves better output quality than GPTQ at the same bit width.

**Choice: AWQ** — it consistently outperforms GPTQ at 4-bit on most benchmarks, at similar quantization time. Both are widely supported by vLLM, llama.cpp (via GGUF conversion), and Hugging Face. AWQ produces slightly larger files in some formats but the quality difference is meaningful for production. GPTQ is acceptable if AWQ quantized weights are not available for the specific model you need.

---

**6.** **Recommendation:**
- **Format:** GGUF
- **Runtime:** llama.cpp (or Ollama, which wraps llama.cpp)
- **Quantization:** Q4_K_M (4-bit with medium quality K-quant)

**Why GGUF + llama.cpp:** It is the only mature runtime that can run quantized LLMs efficiently using CPU RAM + system memory, with optional use of integrated GPU (Metal on macOS, CUDA on NVIDIA). It handles mixed CPU/GPU offloading automatically.

**Why Q4_K_M:** A 7B model at Q4_K_M uses ~4.5 GB of RAM — comfortably within 16 GB, leaving room for the OS and other applications. Q5_K_M (~5.5 GB) gives better quality but tighter fit. Q8_0 (~7 GB) approaches full BF16 quality but leaves less headroom.

**Quality tradeoffs:**
- Responses will be noticeably lower quality than the full BF16 model on complex reasoning tasks
- Simple Q&A, summarisation, and chat will work well
- Code generation degrades more than general text at lower bit widths
- Inference speed on CPU is ~10–30 tokens/second — usable for offline work, too slow for high-volume production

---

**7.** **OS analogy:** Virtual memory in an operating system allows programs to use more address space than physical RAM by storing memory pages on disk and loading them on demand. Physical RAM is divided into fixed-size pages; a page table maps virtual addresses to physical pages; pages are allocated on-demand and freed when no longer needed.

**PagedAttention applies the same concept to KV cache:** GPU memory is divided into fixed-size blocks (each holding KV tensors for a fixed number of tokens — e.g., 16). A block table maps each sequence's logical token positions to physical memory blocks. Blocks are allocated on-demand as tokens are generated; freed immediately when a request completes.

**The problem it solves:** Naive KV cache allocation reserves a contiguous chunk of memory for each request's *maximum possible length* — upfront. A request with max_length=4096 reserves 4096 tokens of KV cache even if it only generates 50 tokens. Most of that memory is wasted.

**Concrete improvement:** Pre-PagedAttention, 60–80% of KV cache memory was wasted on unused pre-allocated space and internal fragmentation. PagedAttention reduces waste to **under 4%** — almost all GPU memory goes to active token storage, enabling much larger effective batch sizes and significantly higher throughput.

---

**8.** **Optimization: Enable prefix caching (automatic prefix caching / RadixAttention in SGLang).**

**What it does:** The first time a request containing the 3,000-token system prompt is processed, vLLM computes and stores its KV tensors. All subsequent requests sharing the same system prompt prefix skip the prefill computation for those 3,000 tokens — they immediately reuse the cached KV vectors.

**Effect:** For 90% of your traffic, prefill effectively drops from 3,000 + user_prompt tokens to just user_prompt tokens. This:
- Reduces TTFT significantly for the 90% of requests using the cached prefix
- Frees GPU compute that was being spent recomputing the same system prompt
- Increases throughput because less prefill compute = more capacity for concurrent requests

**Cost reduction:** Anthropic's prompt caching documentation reports up to 90% reduction in cost for the cached portion. If your system prompt is 3,000 tokens and the average user message is 300 tokens, the system prompt is ~91% of the per-request input — enabling dramatic savings. In vLLM, enable with `enable_prefix_caching=True`. In SGLang, RadixAttention handles this automatically.

---

**9.** Speculative decoding maintains exact output fidelity through a **rejection sampling** mechanism that is mathematically guaranteed to produce the same distribution as the target model alone.

The process:
1. The draft model proposes K tokens with probabilities p_draft(token)
2. The target model computes the probabilities p_target(token) for each proposed token **in a single parallel pass**
3. Each token is **accepted** with probability min(1, p_target / p_draft)
   - If p_target ≥ p_draft: always accept (target model is at least as confident)
   - If p_target < p_draft: accept with probability p_target/p_draft (reject with probability 1 − p_target/p_draft)
4. At the first rejection, the target model samples a new token from an adjusted distribution that corrects for the draft model's overconfidence

This rejection sampling procedure is a well-known technique from statistics that guarantees the **accepted tokens follow exactly the same distribution as if the target model had generated them independently**. The draft model's errors are caught and corrected — they don't corrupt the output. The only effect of a "wrong" draft token is reduced efficiency (that token's speedup is lost), not incorrect output.

---

**10.** **Hardware needed for a 70B model:**

A 70B model in BF16 requires ~140 GB VRAM. The minimum configuration for serving (not just holding weights):
- **2× H100 80GB** (160 GB combined) — minimum practical setup
- With tensor parallelism across both GPUs

**Monthly cost:**
- H100 instances at $4/hour × 2 GPUs × 24 hours × 30 days = **$5,760/month**
- Add ~20% for storage, networking, orchestration overhead: ~**$6,900/month total**

**Comparison:** Current API cost = $15,000/month

**Break-even analysis:** Self-hosting costs ~$6,900/month vs. $15,000/month → **self-hosting saves ~$8,100/month** at current volume.

**Caveats not in the raw numbers:**
- Engineering time to set up and maintain vLLM/serving infrastructure
- Redundancy (if one GPU fails, service goes down — production needs at least 2× the minimum)
- The capability gap: a self-hosted 70B open-weight model may not match GPT-4o/Claude on your specific task
- Actual break-even with 2× redundancy and ops overhead: closer to $14,000/month — margins tighten significantly

Self-hosting makes economic sense here, but the decision should account for total cost of ownership, not just GPU rental.

---

**11.** **Latency components in the voice assistant pipeline:**

```
Speech → [ASR transcription]    → 200–400ms  (Whisper or cloud ASR)
       → [LLM TTFT]             → 500–2000ms (70B model, depends on prompt length)
       → [LLM first token]      → part of above
       → [TTS first audio word] → 100–200ms  (streaming TTS)
Total to first audio word:       → 800–2600ms
```

A 70B model API TTFT is typically 500–2,000ms even with prompt caching, depending on provider load and prompt length. **800ms total is not achievable with a 70B API** under realistic conditions.

**Architectural changes to make it feasible:**

1. **Downsize the model:** Use a 7B or 13B model (self-hosted or API) — TTFT drops to 100–300ms. Quality decreases but may be acceptable for voice Q&A.

2. **Use streaming TTS:** Start generating audio from the first sentence while the LLM continues generating. Perceived latency drops significantly even if full response takes longer.

3. **Optimise the pipeline order:** Run ASR and LLM in a streaming pipeline (start LLM as ASR produces partial transcription). Reduces sequential wait.

4. **Use a model with fast prefill:** Providers with prefix caching and optimised prefill (e.g., Groq, which uses LPU hardware) achieve TTFT of 100–200ms even for larger models.

Realistic target for voice-grade latency: 7–14B model + streaming TTS + streaming ASR → ~500–700ms achievable.

---

**12.** Ranked from highest to lowest recommended batch size:

**Batch B (highest): 10,000 users, 100-token chats**
Short prompts = small, fast prefill. Short expected responses = small KV cache per request. Many concurrent users benefit from large batches because each request is small and GPU compute can be shared efficiently. Continuous batching shines here — requests arrive and complete quickly, keeping the batch full.

**Batch C (medium): 1 user, 500-token input, 5,000-token response**
Long response means large KV cache that grows over 5,000 decode steps. KV cache memory limits how many of these can run concurrently. But with only 1 user, batch size is limited to 1 regardless — this is a serial workload. If scaled to many users, the long output severely limits concurrency.

**Batch A (lowest): 10 users, 50,000-token documents**
Each request's KV cache alone is enormous (50,000 tokens × layers × hidden_dim). At 50K tokens, a single request's KV cache can consume 20–40 GB on a typical GPU. Even with PagedAttention, fitting more than 2–3 such requests simultaneously on a single H100 may be impossible. Batch size is severely constrained by KV cache memory, not compute.

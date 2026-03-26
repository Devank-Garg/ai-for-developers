# Concepts: Inference & Deployment

> **Tag: core** — Getting a model to production is a different problem from training one. This module covers what makes inference hard, how production serving works, and how to make informed decisions about hosting, hardware, and cost.

---

## The inference problem

Training a model is expensive but happens once. **Inference** — running the model to serve user requests — happens continuously, for every user, forever. The economics of inference dominate the economics of running any real AI application.

The fundamental bottleneck: LLM inference is **memory-bandwidth-bound**, not compute-bound.

During a forward pass, the GPU must:
1. Load model weights from GPU memory into compute units
2. Perform matrix multiplications
3. Write results back

For large models, step 1 dominates. The GPU's compute units spend most of their time waiting for weights to arrive from memory, not actually computing. This means throwing more compute at the problem (faster GPUs with more FLOPS) helps less than you'd expect — what matters is memory bandwidth.

**Practical consequence:** A model with twice the parameters takes roughly twice as long to generate each token, regardless of how fast the GPU's math units are.

---

## Prefill vs. decode — the two phases of inference

Every LLM generation request has two distinct phases with very different characteristics:

**Prefill (prompt processing):**
- The full input prompt is processed in parallel in a single forward pass
- Compute-intensive — many tokens processed simultaneously
- Fast (typically milliseconds to seconds, depending on prompt length)
- Produces the KV cache (explained below) and the first output token

**Decode (token generation):**
- Tokens are generated one at a time, each requiring a full forward pass
- Memory-bandwidth-bound — weights reloaded for every token
- The dominant cost for long responses
- Each step produces one token; a 500-token response = 500 sequential forward passes

```
Request lifecycle:
[input prompt: 500 tokens]  →  prefill (fast, parallel)  →  first token
                            →  decode token 2            →  token 2
                            →  decode token 3            →  token 3
                            ...
                            →  decode token 500          →  final token
```

This is why **Time to First Token (TTFT)** and **Tokens Per Second (TPS)** are measured separately in production — they reflect two different bottlenecks.

---

## KV cache

During the decode phase, the model needs the key and value vectors from every previous token to compute attention for the next token. Without caching, this would require reprocessing the entire sequence at every step.

The **KV cache** stores these key and value tensors so they don't need to be recomputed. Once a token has been processed, its K and V vectors are saved and reused for all subsequent tokens in the same sequence.

```
Step 1: process [token_1]           → compute K1, V1 → save to cache
Step 2: process [token_2]           → compute K2, V2 → save to cache
        attend to [K1,V1] from cache (no recomputation)
Step 3: process [token_3]           → compute K3, V3 → save to cache
        attend to [K1,V1,K2,V2] from cache
```

**KV cache memory cost:** For a typical model serving setup, the KV cache consumes a large portion of GPU memory — often as much as the model weights themselves. For long contexts (128K tokens) with large batch sizes, KV cache becomes the dominant memory consumer.

**Prompt caching (prefix caching):** Many providers (Anthropic, OpenAI, Google) now cache the KV tensors for frequently-used system prompts. If 1,000 requests share the same 2,000-token system prompt, that prefix is computed once and reused — dramatically reducing cost and latency for repeated calls with the same prefix. Anthropic's prompt caching reduces costs by up to 90% for long system prompts.

---

## Quantization for inference

Quantization reduces the precision of model weights to use less memory and run faster. Unlike training (which uses BF16 master weights), inference quantization can go much lower without unacceptable quality loss.

**Common inference quantization formats:**

| Format | Bits per weight | Memory (7B model) | Quality loss | Use case |
|---|---|---|---|---|
| BF16 | 16 | ~14 GB | Baseline | Full-precision inference |
| FP8 | 8 | ~7 GB | Negligible | H100/A100 native; production APIs |
| GPTQ | 4 | ~4 GB | Small | GPU inference, post-training quantization |
| AWQ | 4 | ~4 GB | Small | Slightly better than GPTQ; activation-aware |
| GGUF (Q4_K_M) | ~4.5 | ~4.5 GB | Moderate | CPU/local inference via llama.cpp |
| GGUF (Q8_0) | 8 | ~7 GB | Minimal | CPU/local high-quality inference |

**GPTQ vs. AWQ:**
- Both achieve 4-bit quantization of GPU models
- AWQ (Activation-Aware Weight Quantization) identifies which weights are most important (based on activation magnitudes) and preserves their precision — consistently outperforms GPTQ at the same bit width
- GPTQ is faster to quantize; AWQ produces better quality; both are widely supported

**GGUF and llama.cpp:**
- GGUF is a file format (not a quantization algorithm) that packages weights + metadata
- Designed for **llama.cpp** — a C/C++ inference runtime that runs quantized models on CPU or Apple Silicon
- The standard for local/offline inference — Ollama uses llama.cpp and GGUF under the hood
- Do not use GGUF with vLLM; it adds overhead. For GPU serving, use GPTQ/AWQ/FP8

**FP8:**
- Native support on H100 and A100 GPUs
- ~2× throughput vs. BF16 at negligible quality cost for most tasks
- Increasingly the default for cloud API providers serving large models

---

## PagedAttention and vLLM

The original KV cache implementation allocated a contiguous block of GPU memory for each request's full maximum context length — upfront, even if the request only used a fraction. This wasted 60–80% of KV cache memory.

**PagedAttention** (introduced by the vLLM team, 2023) applies the virtual memory / paging concept from operating systems to KV cache management:

- Memory is divided into fixed-size **pages** (blocks of KV vectors for N tokens)
- Pages are allocated on-demand as tokens are generated — not upfront
- Pages from different requests can be non-contiguous in physical memory
- Completed requests release their pages immediately for reuse

Result: **<4% KV cache memory waste** vs. 60–80% with naive allocation. This enables far larger batch sizes for the same GPU memory, directly translating to higher throughput.

**vLLM** is the open-source serving engine that introduced PagedAttention and remains the most widely used production LLM serving framework. It also implements:
- Continuous batching (see below)
- Multi-LoRA serving (serve many fine-tuned adapters with one base model)
- Speculative decoding
- FP8, GPTQ, AWQ quantization support
- Multi-GPU and multi-node tensor parallelism

---

## Continuous batching

**Naive batching:** Wait for N requests to arrive, process them together as a batch, return all results. Problem: a request generating 500 tokens holds up the batch even if 9 other requests finished at token 10.

**Continuous batching** (also called in-flight batching): The GPU is never idle. As soon as a sequence in the batch finishes generating, a new request is immediately inserted to take its place. The batch composition changes dynamically with every token step.

```
Naive batching:
  Batch = [req1(500 tokens), req2(10 tokens), req3(10 tokens)]
  All wait for req1 to finish → GPU idle after req2, req3 complete

Continuous batching:
  Step 1: [req1, req2, req3] → req2 done → insert req4
  Step 2: [req1, req3, req4] → req3 done → insert req5
  ...GPU continuously utilized
```

Combined with PagedAttention, continuous batching is what makes vLLM capable of 10–30× higher throughput than naive single-request inference.

---

## SGLang and RadixAttention

**SGLang** is a newer inference framework (Stanford, 2023) that extends PagedAttention with **RadixAttention** — a radix tree-based cache that enables automatic prefix sharing across requests.

When many requests share a common prefix (e.g., a 2,000-token system prompt + few-shot examples), RadixAttention automatically detects and reuses the cached KV tensors for that prefix — even across different users. This is more aggressive than prefix caching in vLLM.

SGLang particularly excels for:
- Workloads with long shared system prompts (RAG, agent frameworks)
- Structured generation (JSON output with grammars)
- Multi-turn conversations where context is reused

In 2025, SGLang and vLLM are the two dominant open-source serving frameworks. SGLang tends to outperform vLLM on structured generation and prefix-heavy workloads; vLLM has broader ecosystem integration.

---

## Speculative decoding

**The bottleneck:** Decode is sequential — you cannot generate token N+1 until you have token N. This means the GPU runs one small operation per step (one matrix multiply per layer per token), leaving most compute unused.

**Speculative decoding** breaks this bottleneck: a small, fast **draft model** generates K candidate tokens in parallel. The large **target model** then verifies all K tokens in a single forward pass (because verification can be parallelised across positions).

```
Without speculative decoding:
  Target model generates tokens 1, 2, 3, 4, 5 sequentially (5 steps)

With speculative decoding:
  Draft model generates tokens 1–5 as candidates (fast)
  Target model verifies all 5 in one parallel pass
  If tokens 1–4 are accepted and token 5 is rejected:
    Accept 1–4, target model samples token 5 correctly
  Net result: 4 tokens in ~1 step instead of 4 steps
```

**Speed gain:** 2–3× on typical workloads. The output is **mathematically identical** to running the target model alone — speculative decoding doesn't change the distribution, only the speed.

**Draft model options:**
- A smaller version of the same model family (e.g., 1B draft for a 70B target)
- N-gram models (use frequent patterns from the prompt as draft tokens)
- EAGLE — a lightweight model trained specifically to predict the target model's outputs

Speculative decoding is now production-ready in vLLM, TensorRT-LLM, and SGLang.

---

## Self-hosting vs. managed APIs

Every team deploying LLMs faces this decision. There is no universally correct answer.

**Managed APIs (Claude, GPT-4o, Gemini):**

| Pros | Cons |
|---|---|
| No infrastructure to manage | Per-token cost compounds at scale |
| Access to frontier models | Data leaves your infrastructure |
| Scales to zero automatically | Rate limits and availability dependencies |
| No ML expertise required | No control over model updates |

**Self-hosted open-weight models (Llama 3, Mistral, Qwen):**

| Pros | Cons |
|---|---|
| Data stays in your infrastructure | Significant ops burden |
| Fixed cost at high volume | Requires ML/infra expertise |
| Full control (fine-tune, version) | Must manage hardware or cloud VMs |
| No rate limits or vendor lock-in | Frontier capability gap vs. best APIs |

**The break-even calculation:** At low request volumes, managed APIs are always cheaper (no fixed cost). At very high volumes (millions of tokens/day), the per-token API cost can exceed the amortised cost of owning/renting hardware. Most teams hit this crossover somewhere between 100M and 1B tokens/month, depending on the model.

---

## Hardware sizing for self-hosting

A practical reference for running open-weight models:

| Model | Precision | Min GPU VRAM | Recommended setup |
|---|---|---|---|
| 7–8B | BF16 | 16 GB | 1× RTX 4090 (24GB) or 1× A10G (24GB) |
| 7–8B | Q4 | 6 GB | 1× RTX 3080 (10GB), runs on CPU with llama.cpp |
| 13B | BF16 | 28 GB | 2× RTX 4090 or 1× A100 40GB |
| 70B | BF16 | 140 GB | 2× H100 80GB or 4× A100 80GB |
| 70B | Q4 | 40 GB | 2× RTX 4090 (48GB combined) |
| 405B | BF16 | 810 GB | 10+ H100 80GB — cloud only |

**Cloud GPU options in 2025:** RunPod, Lambda Labs, and Vast.ai offer on-demand H100/A100 access at lower cost than AWS/GCP. For self-hosting at scale, Kubernetes-based deployments on cloud GPU fleets are standard.

---

## Key inference metrics to track

| Metric | Definition | Why it matters |
|---|---|---|
| TTFT (Time to First Token) | Latency from request to first output token | User-perceived responsiveness |
| TPS (Tokens Per Second) | Output generation speed | Streaming experience quality |
| Throughput | Requests per second (across all users) | Capacity planning |
| GPU utilisation | % of GPU compute in use | Efficiency of serving setup |
| KV cache hit rate | % of prefill reused from cache | Cost reduction via prefix caching |
| P95/P99 latency | 95th/99th percentile response time | Production reliability planning |

---

## Disaggregated serving — the 2025 frontier

Traditional serving runs prefill and decode on the same GPUs. **Disaggregated serving** splits them:
- **Prefill workers:** Optimised for compute (processing long input prompts)
- **Decode workers:** Optimised for memory bandwidth (generating tokens)

Because prefill is compute-bound and decode is memory-bandwidth-bound, they have different optimal hardware and batching strategies. Separating them allows each to be tuned independently.

Disaggregated serving is emerging in production at large scale (used internally by major providers) and is being integrated into vLLM as of 2025. It's not yet standard for self-hosted deployments but represents the direction the field is moving.

---

## Summary

| Concept | Key point |
|---|---|
| Memory-bandwidth-bound | Inference speed is limited by how fast weights move from memory to compute — not raw FLOP count |
| Prefill vs. decode | Prefill processes the prompt in parallel (fast); decode generates tokens sequentially (the bottleneck) |
| KV cache | Stores key/value tensors to avoid recomputing attention for previous tokens; grows with sequence length |
| Prompt caching | Reuses KV cache for shared prefixes (system prompts); up to 90% cost reduction on repeated prefixes |
| FP8 / AWQ / GGUF | Quantization formats for inference; FP8 for GPU APIs, AWQ for GPU self-hosting, GGUF for CPU/local |
| PagedAttention | OS-paging-inspired KV cache management; reduces waste from 60–80% to <4%; enables large batches |
| Continuous batching | Dynamically replaces finished sequences in the batch; GPUs never idle; 10–30× throughput gain |
| vLLM | The standard open-source LLM serving engine; implements PagedAttention + continuous batching |
| SGLang | Alternative serving framework with RadixAttention; excels at prefix-heavy and structured workloads |
| Speculative decoding | Draft model proposes tokens; target model verifies in parallel; 2–3× speedup, identical outputs |
| Self-hosting break-even | APIs cheaper at low volume; self-hosting cheaper at 100M–1B+ tokens/month |
| Disaggregated serving | Separates prefill and decode across specialised worker pools; emerging 2025 standard at scale |

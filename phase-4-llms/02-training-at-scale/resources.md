# Resources: Training at Scale

These are curated external resources for going deeper on topics from this module. None are required — the concepts file covers everything you need to continue the course. Resources marked **[paper]** are academic; read the abstract and introduction unless you want technical depth.

---

## Essential watching

**Andrej Karpathy — Let's reproduce GPT-2 (124M)**
https://www.youtube.com/watch?v=l8pRSuU81PU
A 4-hour live coding session where Karpathy trains a GPT-2 equivalent from scratch and pushes it to run on a GPU cluster. Covers the full training pipeline: data loading, mixed precision, gradient clipping, distributed training, and cost optimisation. One of the best end-to-end demonstrations of how training actually works — not just theory. Complements this module directly.

**Andrej Karpathy — Let's build GPT from scratch**
https://www.youtube.com/watch?v=kCc8FmEb1nY
The 2-hour predecessor to the above. Builds a character-level GPT from scratch in PyTorch. If you haven't watched this, do it before the GPT-2 video. The training loop, loss curve interpretation, and generation logic are all covered.

**Yannic Kilcher — Chinchilla (Hoffmann et al.) Paper Explained**
https://www.youtube.com/watch?v=VdKsdsB0PdA
A clear explanation of the Chinchilla scaling laws paper — arguably the most practically important paper for understanding modern LLM training decisions. Kilcher walks through the methodology and findings in plain terms.

---

## Foundational papers

**"Scaling Laws for Neural Language Models" — Kaplan et al., 2020 (OpenAI)** [paper]
https://arxiv.org/abs/2001.08361
The original scaling laws paper. Shows that LLM loss decreases predictably as you scale parameters, data, and compute. Read the abstract, figures, and Section 1. The key takeaway: performance follows smooth power laws — more compute reliably produces better models.

**"Training Compute-Optimal Large Language Models" — Hoffmann et al., 2022 (DeepMind)** [paper]
https://arxiv.org/abs/2203.15556
The Chinchilla paper. Introduces the finding that most existing large models are undertrained, and establishes the ~20 tokens per parameter rule for compute-optimal training. More practically important than the Kaplan paper for understanding modern model releases. Read Section 1 and the results tables.

**"ZeRO: Memory Optimizations Toward Training Trillion Parameter Models" — Rajbhandari et al., 2020 (Microsoft)** [paper]
https://arxiv.org/abs/1910.02054
The paper behind DeepSpeed's ZeRO optimizer. Introduces the three stages of optimizer state, gradient, and weight sharding. Understanding Figure 1 and Sections 1–3 gives you everything you need — the rest is implementation detail.

**"Megatron-LM: Training Multi-Billion Parameter Language Models Using Model Parallelism" — Shoeybi et al., 2019 (NVIDIA)** [paper]
https://arxiv.org/abs/1909.08053
Introduces efficient tensor parallelism for transformer layers. The technique described here is used in virtually every frontier model training setup today. Read Sections 1–3.

**"Efficient Large Scale Language Modeling with Mixtures of Experts" — Artetxe et al., 2021 (Meta)** [paper]
https://arxiv.org/abs/2112.10684
Background on Mixture of Experts (MoE) — the architecture used by models like Mixtral and likely GPT-4 — and how it changes the compute/parameter tradeoff. Relevant once you understand dense training.

---

## Frameworks and libraries

**DeepSpeed (Microsoft)**
https://github.com/microsoft/DeepSpeed
The most widely used library for large-scale distributed training. Implements ZeRO Stages 1–3, pipeline parallelism, mixed precision, and gradient checkpointing. Integrates with PyTorch and Hugging Face Trainer. If you're running serious training jobs, this is the starting point. Documentation: https://www.deepspeed.ai/

**Megatron-LM (NVIDIA)**
https://github.com/NVIDIA/Megatron-LM
NVIDIA's research framework for training very large transformer models. Implements tensor parallelism, pipeline parallelism, and sequence parallelism — the same stack used to train Megatron models internally. More research-oriented than DeepSpeed, less turnkey, but highly performant on NVIDIA hardware.

**torchtitan (Meta / PyTorch)**
https://github.com/pytorch/torchtitan
Meta's production-quality training framework built natively on PyTorch 2.x. Implements FSDP2 (Fully Sharded Data Parallel), tensor parallelism, pipeline parallelism, and async checkpointing. Used to train Llama 3. Actively developed and increasingly the reference implementation for PyTorch-native large model training.

**PyTorch FSDP (Fully Sharded Data Parallel)**
https://pytorch.org/docs/stable/fsdp.html
PyTorch's built-in implementation of ZeRO-style sharding. FSDP2 (in PyTorch 2.x) is the production choice for distributed training without external libraries. Good starting point if you want to understand distributed training before adding DeepSpeed complexity. Tutorial: https://pytorch.org/tutorials/intermediate/FSDP_tutorial.html

**nanoGPT (Andrej Karpathy)**
https://github.com/karpathy/nanoGPT
A clean, minimal implementation of GPT training in ~300 lines of PyTorch. No abstractions, no framework overhead — just the training loop. Invaluable for understanding what actually happens during training before layering on distributed complexity. If you can read and understand this code, you understand LLM training.

**llm.c (Andrej Karpathy)**
https://github.com/karpathy/llm.c
A pure C implementation of GPT-2 training — no Python, no PyTorch. Extreme clarity about what training actually does at the hardware level. Not for production use; for understanding. Pairs with the "Let's reproduce GPT-2" video.

**Hugging Face Accelerate**
https://github.com/huggingface/accelerate
A lightweight library that wraps distributed training boilerplate for PyTorch and Hugging Face models. Lower overhead than DeepSpeed for small-to-medium scale. Good starting point for developers new to multi-GPU training.

**Hugging Face Trainer**
https://huggingface.co/docs/transformers/main_classes/trainer
The high-level training API in the `transformers` library. Integrates with DeepSpeed and Accelerate. If you're fine-tuning existing models (as opposed to training from scratch), this is the fastest path from idea to running job.

---

## Company engineering blogs and technical reports

**"Llama 3 Technical Report" — Meta AI, 2024** [paper]
https://arxiv.org/abs/2407.21783
Meta's detailed account of training Llama 3 — data curation, model architecture, training infrastructure, and post-training. Unusually transparent for a frontier model report. Read Section 2 (pre-training) and Section 3 (data) for this module's content. One of the best primary sources on what industrial-scale training actually looks like.

**"The Llama 3 Herd of Models" — Meta AI blog**
https://ai.meta.com/blog/meta-llama-3-1/
The accompanying blog post — more readable than the paper. Covers the data pipeline, compute infrastructure, and key training decisions. Explains why they chose to over-train smaller models on more data.

**"GPT-4 Technical Report" — OpenAI, 2023** [paper]
https://arxiv.org/abs/2303.08774
OpenAI's report on GPT-4 — deliberately sparse on architecture and training details. Useful primarily for the capability evaluations and the discussion of RLHF and alignment. Contrast with Meta's much more detailed Llama reports.

**"PaLM: Scaling Language Modeling with Pathways" — Chowdhery et al., 2022 (Google)** [paper]
https://arxiv.org/abs/2204.02311
Google's paper on training a 540B model on TPUs using their Pathways infrastructure. Sections 3–4 cover the training setup, data, and infrastructure. Notable for the transparency about training instabilities encountered (Section 6).

**"OPT: Open Pre-trained Transformer Language Models" — Zhang et al., 2022 (Meta)** [paper]
https://arxiv.org/abs/2205.01068
Meta's 175B open-weight model paper. Includes a logbook of training failures — loss spikes, hardware issues, checkpoint restarts. One of the most honest accounts of what large-scale training actually looks like in practice. The logbook in Appendix C is valuable.

**"Scaling, Efficiency, and Alignment" — Anthropic Research Blog**
https://www.anthropic.com/research
Anthropic's research blog. Contains posts on Constitutional AI, model evaluation, and training approaches. Less training-infrastructure focused than Meta/Google but strong on alignment and post-training topics relevant to Phase 4.

**"Training Great LLMs Entirely from Scratch in the Hugging Face Ecosystem" — Hugging Face**
https://huggingface.co/blog/finetuning-llms
An overview of the Hugging Face training ecosystem for LLMs, covering SFT, DPO, and infrastructure choices. Practitioner-oriented; good bridge between theory and implementation.

---

## Prominent AI practitioners — written guides and newsletters

**Lilian Weng — "Large Language Model (LLM) Training" blog post**
https://lilianweng.github.io/posts/2023-01-27-the-transformer-family-v2/
Weng (formerly VP of Research at OpenAI) writes some of the most reliable and deep technical blog posts in the field. This post covers the transformer family and training evolution. Her blog (https://lilianweng.github.io/) is a reference-quality resource across all Phase 4 topics.

**Sebastian Raschka — "Training Large Language Models"**
https://magazine.sebastianraschka.com/p/llms-from-scratch-implementing-the
Raschka's newsletter consistently explains training concepts in a way that's accurate and accessible. His book "Build a Large Language Model from Scratch" covers the full training loop at a level appropriate for developers. The newsletter (https://magazine.sebastianraschka.com/) tracks new papers with reliable commentary.

**Chip Huyen — "MLOps and LLMOps" blog**
https://huyenchip.com/blog/
Huyen writes about production ML and LLM systems from an engineering and systems perspective. Posts on training infrastructure, data pipelines, and serving are relevant to this module and Phase 5. Her book "Designing Machine Learning Systems" (O'Reilly) covers infrastructure at depth.

**Tim Dettmers — "A Beginner's Guide to GPU Hardware for Machine Learning"**
https://timdettmers.com/2023/01/30/which-gpu-for-deep-learning/
Dettmers (quantization researcher, creator of bitsandbytes) explains GPU memory, compute, and which hardware to use for different scales of training and inference. Essential reading if you're making any infrastructure decisions. Regularly updated.

---

## Interactive tools and monitoring

**Weights & Biases (W&B)**
https://wandb.ai/
The standard tool for tracking training runs — logging loss curves, gradients, GPU utilisation, and hyperparameters. Free for personal use. Most open training runs (including Llama models) publish their W&B logs publicly. Looking at real loss curves from large training runs builds intuition for what healthy and unhealthy training looks like.

**PyTorch Profiler**
https://pytorch.org/tutorials/recipes/recipes/profiler_recipe.html
The built-in PyTorch tool for identifying training bottlenecks — whether time is spent on compute, data loading, or communication. Before optimising a training pipeline, profile it. Essential for diagnosing why adding more GPUs isn't speeding things up.

**NVIDIA Nsight Systems**
https://developer.nvidia.com/nsight-systems
NVIDIA's profiling tool for GPU utilisation and kernel-level performance. For diagnosing GPU memory, utilisation, and communication overhead in distributed training. More detailed than PyTorch Profiler; useful once you've exhausted what Profiler can tell you.

---

## Reference documentation

**DeepSpeed Configuration Reference**
https://www.deepspeed.ai/docs/config-json/
The full reference for DeepSpeed's `ds_config.json` — where ZeRO stages, mixed precision, gradient clipping, and checkpointing are configured. When you set up a real training job, you'll live in this document.

**PyTorch Distributed Training Overview**
https://pytorch.org/tutorials/beginner/dist_overview.html
PyTorch's own overview of distributed training options: DDP, FSDP, RPC. Covers when to use which and links to tutorials. Start here before reaching for external frameworks.

**Hugging Face Distributed Training Guide**
https://huggingface.co/docs/transformers/en/perf_train_gpu_many
A practical guide to multi-GPU training with the Hugging Face ecosystem — covers DDP, FSDP, DeepSpeed integration, and mixed precision. Most practically useful for developers who want to scale fine-tuning jobs, not just train from scratch.

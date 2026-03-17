# Resources: Transformers

This module is the most resource-rich in the course. The Transformer is the foundation of everything in Phase 4 and 5, so invest time here.

---

## Visual explainers (start here)

**"The Illustrated Transformer" — Jay Alammar**
https://jalammar.github.io/illustrated-transformer/
The most widely referenced visual guide to Transformer architecture. Covers the attention mechanism, multi-head attention, encoder/decoder stacks, and positional encoding with clear, step-by-step diagrams. Required reading before Phase 4. If you read one resource from this module, make it this one.

**"The Illustrated GPT-2" — Jay Alammar**
https://jalammar.github.io/illustrated-gpt2/
Focuses specifically on the decoder-only Transformer (the GPT architecture). Walks through how tokens are generated one at a time. A natural follow-up to the Illustrated Transformer.

**3Blue1Brown — "Attention in transformers, visually explained" (Deep Learning, Chapter 6)**
https://www.youtube.com/watch?v=eMlx5fFNoYc
Grant Sanderson's visual explanation of self-attention and multi-head attention. Animated, intuitive, and mathematically honest without being overwhelming. Highly recommended alongside the Alammar posts.

---

## The original paper

**"Attention Is All You Need" — Vaswani et al. (2017)**
https://arxiv.org/abs/1706.03762
The paper that introduced the Transformer. The abstract, introduction, and architecture diagram (Figure 1) are essential. The full paper is dense but readable — the methods section explains multi-head attention and positional encoding directly.

---

## Annotated implementations

**"The Annotated Transformer" — Harvard NLP (Austin Huang et al.)**
https://nlp.seas.harvard.edu/annotated-transformer/
A line-by-line annotated implementation of "Attention Is All You Need" in PyTorch. Every equation from the paper maps to a code block with explanation. The best way to move from conceptual understanding to implementation-level understanding. Requires basic PyTorch.

**Andrej Karpathy — "Let's build GPT: from scratch, in code, spelled out"**
https://www.youtube.com/watch?v=kCc8FmEb1nY
A 2-hour video building a GPT-style decoder-only Transformer from scratch in ~200 lines of PyTorch. Builds directly on the micrograd video. By the end you have a working character-level LLM. One of the highest-signal pieces of ML education available.

Karpathy's nanoGPT (the code): https://github.com/karpathy/nanoGPT

---

## Efficient attention (for context)

**"Efficient Transformers: A Survey" — Tay et al. (2020)**
https://arxiv.org/abs/2009.06732
Survey of approaches to solving the O(n²) attention problem. Not required reading, but useful context for why long-context models are hard and how the field is addressing it.

**"FlashAttention" — Dao et al. (2022)**
https://arxiv.org/abs/2205.14135
The most impactful practical improvement to Transformer training efficiency. FlashAttention computes exact attention faster and with less memory by being clever about GPU memory access patterns. Used in virtually all production LLM training today.

---

## Conceptual depth

**"In-context Learning and Induction Heads" — Anthropic (2022)**
https://transformer-circuits.pub/2022/in-context-learning-and-induction-heads/index.html
Explores one of the most interesting emergent behaviours in Transformers — the ability to learn from examples in the prompt (in-context learning). From Anthropic's mechanistic interpretability research.

**Transformer Circuits Thread — Anthropic**
https://transformer-circuits.pub/
A series of papers and essays on understanding *how* Transformers compute internally. Relevant from Phase 4 onward. Start with "A Mathematical Framework for Transformer Circuits" if curious about the mechanistic perspective.

---

## Going deeper

**"Scaling Laws for Neural Language Models" — Kaplan et al., OpenAI (2020)**
https://arxiv.org/abs/2001.08361
The paper that established predictable power-law relationships between model size, data, compute, and loss. Understanding scaling laws is foundational for Phase 4 (training at scale). The abstract and figures tell the story clearly.

**"Training Compute-Optimal Large Language Models" — Hoffmann et al. (Chinchilla, 2022)**
https://arxiv.org/abs/2203.15556
The "Chinchilla" paper — revised the scaling law recipe and showed that most large models at the time were undertrained (too large for their data budget). Profoundly influenced how all subsequent LLMs are trained. One of the most practically impactful papers in the field.

**"RoFormer: Enhanced Transformer with Rotary Position Embedding" — Su et al. (2021)**
https://arxiv.org/abs/2104.09864
Introduces RoPE (Rotary Position Embedding), which is now the dominant positional encoding scheme in modern LLMs (LLaMA, Mistral, Gemma, and others). If you want to understand why modern models handle position differently from the original sinusoidal scheme, this is the source.

**"LLM Visualization" — Brendan Bycroft**
https://bbycroft.net/llm
An interactive 3D visualisation of a small GPT running a forward pass in real time. Shows every matrix multiplication, attention head, and layer. Exceptional for making the abstract Transformer architecture concrete and spatial.

# Concepts: Transformers

> **Tag: core** — Every LLM you will work with in Phase 4 is built on the Transformer architecture. This module is required.

---

## The key insight

Transformers, introduced in the 2017 paper "Attention Is All You Need," solved the two core problems with RNNs:

1. **Sequential computation** — replaced with parallel attention over all positions
2. **Information bottleneck** — replaced with direct connections between any two positions

The mechanism that enables both is **attention**.

---

## Self-attention

**Self-attention** allows each position in a sequence to "look at" every other position and decide how much to weight its contribution.

For each token in the input, the model creates three vectors (learned projections of the token's embedding):
- **Query (Q)** — "what am I looking for?"
- **Key (K)** — "what do I contain?"
- **Value (V)** — "what do I contribute if selected?"

The attention score between two positions is the dot product of one's Query with the other's Key. These scores are normalised with softmax to produce attention weights (summing to 1). The output for each position is the weighted sum of all Value vectors:

```
Attention(Q, K, V) = softmax(QKᵀ / √d_k) × V
```

The `√d_k` scaling prevents dot products from becoming too large in high dimensions, which would push softmax into near-zero gradient regions.

In plain terms: every token can attend to every other token, and the model learns *which* tokens are relevant to *which* other tokens — dynamically, based on content, not position.

---

## Multi-head attention

Running a single attention operation limits what patterns can be detected. **Multi-head attention** runs several attention operations in parallel (each with different Q/K/V projections), then concatenates and linearly combines the results.

Each "head" can specialise: one head might capture syntactic relationships, another semantic similarity, another positional patterns. The full model benefits from all of them simultaneously.

---

## Positional encoding

Self-attention has no inherent notion of order — it treats the input as a set, not a sequence. A sentence's words in any order would produce the same attention scores.

**Positional encodings** are added to the input embeddings to inject sequence order information. The original paper used sinusoidal functions with different frequencies for different positions. Modern models typically learn positional encodings or use relative position schemes like **RoPE** (Rotary Position Embedding).

---

## The Transformer block

A single **Transformer block** (also called a **layer**) consists of:

1. **Multi-head self-attention** — each token attends to all others
2. **Add & Norm** — residual connection + layer normalisation
3. **Feed-forward network (FFN)** — two linear layers with a non-linearity, applied independently to each position
4. **Add & Norm** — again

```
Input
→ Multi-head attention → Add & Norm
→ Feed-forward network → Add & Norm
→ Output
```

The **residual connections** (Add) allow gradients to flow directly from output to input, enabling very deep stacking without vanishing gradients. **Layer normalisation** stabilises activations across the layer.

Modern LLMs stack dozens to hundreds of these blocks. GPT-3 has 96 layers; GPT-4's architecture is undisclosed.

---

## Encoder vs decoder vs encoder-decoder

The original Transformer had two parts:

**Encoder** — processes the full input bidirectionally (each token attends to all other tokens, including future ones). Used for understanding tasks: BERT and its variants are encoder models.

**Decoder** — processes tokens autoregressively, with **causal (masked) self-attention**: each position can only attend to previous positions, not future ones. This is necessary for generation — you can't look at what you haven't generated yet. GPT-style models are decoder-only.

**Encoder-decoder** — the original architecture for sequence-to-sequence tasks (translation, summarisation): encoder processes the input, decoder generates the output while attending to the encoder's representations. T5 and BART use this architecture.

For LLMs, **decoder-only** is now dominant.

---

## Why Transformers work so well

- **Parallelism** — all positions processed simultaneously → dramatically faster training on GPUs
- **Direct long-range connections** — any token can attend to any other in one step, no matter the distance
- **Scale** — Transformer performance scales predictably with parameters and data (see Phase 4)
- **Versatility** — the same architecture handles text, code, images (Vision Transformers), audio, and more

---

## Summary

| Component | Role |
|-----------|------|
| Self-attention | Each token attends to all others; captures context dynamically |
| Q/K/V | Learned projections: what to look for, what to offer |
| Multi-head attention | Multiple parallel attention operations; different relationship types |
| Positional encoding | Injects sequence order (attention itself is order-agnostic) |
| Residual connections | Allow gradient flow through deep stacks |
| Layer normalisation | Stabilises training across depth |
| Decoder-only | Causal masking for autoregressive generation (GPT style) |

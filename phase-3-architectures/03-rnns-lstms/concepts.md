# Concepts: RNNs and LSTMs (Optional)

> **Tag: optional** — RNNs have largely been replaced by Transformers for language tasks, but understanding them makes the Transformer's advantages concrete. Read this module to understand why the field moved on and what problem attention solves.

---

## Why a developer needs this

You will almost certainly not build or run an RNN. But reading this module gives you two things:

**1. The "why" behind Transformers.** The Transformer's key design choices — attention, parallel processing, no hidden state — only make full sense when you understand what problems they solve. RNNs are those problems. If you skip this module you can still use a Transformer, but you won't understand why it's designed the way it is.

**2. Vocabulary that reappears.** Autoregressive generation, hidden state, sequence-to-sequence, encoder-decoder — this vocabulary originates with RNNs and carries directly into how LLMs work. LLMs generate tokens one at a time (autoregressive), just like RNNs did. The difference is in the mechanism, not the high-level concept.

---

## The sequential data problem

A fully connected network processes each input independently. A sentence is not a bag of independent features — the meaning of a word depends on what came before it. "The bank was steep" and "The bank was solvent" share the same words but mean entirely different things based on context.

**Recurrent Neural Networks (RNNs)** were designed to handle sequences by maintaining a **hidden state** — a compact summary of everything seen so far — that gets updated at each step.

---

## How RNNs work

At each time step `t`, an RNN takes two inputs: the current element of the sequence (`xₜ`) and the previous hidden state (`hₜ₋₁`). It produces a new hidden state (`hₜ`) and optionally an output:

```
hₜ = f(W_h × hₜ₋₁ + W_x × xₜ + b)
```

The same weights (`W_h`, `W_x`) are used at every time step. After processing all elements, the final hidden state is supposed to encode everything the model needs about the sequence.

RNNs can generate text by predicting one token at a time, feeding each output back in as the next input — a process called **autoregressive generation**. LLMs use the same principle, implemented with Transformers.

---

## The vanishing gradient problem

In theory, the hidden state can carry information from the beginning of a sequence to the end. In practice, it can't — at least not for long sequences.

When training an RNN with backpropagation, gradients must flow backwards through all the time steps. At each step, they get multiplied by the weight matrix. If those multiplications make the values shrink repeatedly, gradients approaching the early timesteps become vanishingly small — **vanishing gradients**. The model essentially forgets distant context.

This makes vanilla RNNs ineffective for sequences longer than ~20–30 tokens. Long-range dependencies — understanding that "it" in sentence 10 refers to "the bank" in sentence 1 — are exactly what language understanding requires.

---

## LSTMs: gated memory

**Long Short-Term Memory networks (LSTMs)**, introduced by Hochreiter and Schmidhuber in 1997, solve the vanishing gradient problem with a more complex cell that separates short-term and long-term memory.

An LSTM cell has three **gates** — learned mechanisms that control information flow:

- **Forget gate** — decides what to erase from long-term memory
- **Input gate** — decides what new information to add to long-term memory
- **Output gate** — decides what to expose from long-term memory at this step

The **cell state** (long-term memory) flows through the network with only minor modifications — the "gradient highway" that allows gradients to flow back many steps without vanishing.

LSTMs dominated sequence modelling (language, speech, time series) from roughly 2014–2018. Then Transformers arrived.

---

## Why Transformers replaced RNNs

RNNs and LSTMs have two fundamental limitations:

1. **Sequential computation** — each step depends on the previous hidden state, so you can't parallelise across time steps. Training is slow.
2. **Information bottleneck** — the entire history must be compressed into a fixed-size hidden state. Long-range information still gets lost for very long sequences.

Transformers solve both: they attend directly to every position in the sequence simultaneously (no sequential bottleneck), and their attention mechanism can explicitly look back at any earlier position (no information compression). The tradeoff is quadratic memory cost with sequence length — addressed by newer efficient attention variants.

---

## What this means when you're building

RNNs are not something you'll call or deploy today, but the concepts have direct parallels in how you think about LLMs:

| RNN concept | LLM equivalent |
|-------------|---------------|
| Hidden state | The model's internal representation, updated at each token (but via attention, not recurrence) |
| Autoregressive generation | Identical — LLMs also generate one token at a time, each conditioned on previous tokens |
| Sequence-to-sequence | Still used for translation, summarisation — modern LLMs are encoder-decoder or decoder-only Transformers doing the same task |
| Context bottleneck | The context window is the LLM equivalent — a hard limit on how much history can be considered |
| Forgetting long-range context | LLMs have this problem too at very long contexts, for different reasons |

The transition from RNNs to Transformers is the most important architectural shift in the history of modern AI. Understanding both makes you a more effective builder of LLM systems.

---

## Summary

| Architecture | Key mechanism | Main weakness |
|-------------|--------------|---------------|
| Vanilla RNN | Hidden state updated each step | Vanishing gradients on long sequences |
| LSTM | Gated cell state separates long/short memory | Still sequential; still bottlenecked |
| Transformer | Direct attention to all positions at once | Quadratic cost with sequence length |

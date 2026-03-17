# Quiz: Transformers

---

## Questions

**1.** What two problems with RNNs do Transformers solve, and how?

---

**2.** In self-attention, what are the Query, Key, and Value vectors? Use an analogy if it helps.

---

**3.** A decoder-only Transformer generating text uses **causal masking**. What does this mean and why is it necessary?

---

**4.** Why are residual connections important in deep Transformer stacks?

---

**5.** BERT is an encoder-only model; GPT is decoder-only. What does this mean for what each is suited for?

---

**6.** Self-attention computes a score between every pair of tokens. If a sequence has 1,000 tokens, how many pairs are there? What problem does this create for very long contexts?

---

## Answers

**1.** Transformers solve:
- **Sequential computation bottleneck**: RNNs process one token at a time. Transformers apply self-attention to all positions simultaneously in parallel, enabling full GPU parallelisation and much faster training.
- **Information bottleneck**: RNNs compress history into a fixed-size hidden state. Transformers maintain direct connections between every pair of positions via attention — any token can attend to any other token in a single step, regardless of distance.

---

**2.** For each token, the model creates three vectors:
- **Query** — what this token is looking for in other tokens ("I need context about the subject")
- **Key** — what this token contains / can be found by ("I am the subject")
- **Value** — the actual information this token contributes when selected

A useful analogy: a search engine. The Query is your search query. Keys are the index terms for each document. Values are the document contents. Attention computes how well your query matches each key, then retrieves a weighted blend of the values.

---

**3.** Causal masking (also called **autoregressive masking**) prevents each position from attending to future positions — at step `t`, the model can only attend to positions 1 through `t`. This is necessary because during generation, future tokens don't exist yet. If the model could see future tokens during training, it would just copy them rather than learn to predict. The mask enforces the causal constraint: predict the next token using only the preceding context.

---

**4.** Residual connections add the layer's input directly to its output: `output = layer(input) + input`. This creates a shortcut for gradients to flow directly backwards through the network, bypassing the layer transformations. Without them, gradients through very deep stacks (50+ layers) would vanish before reaching early layers, preventing those layers from learning. Residual connections are why training 96-layer Transformers is feasible.

---

**5.**
- **BERT (encoder-only)**: each token attends to all other tokens bidirectionally — including those after it. This full-context attention makes BERT excellent at *understanding* tasks: question answering, classification, named entity recognition. But bidirectional attention can't be used autoregressively for generation (you can't see future tokens you haven't generated yet).
- **GPT (decoder-only)**: causal masking means each token attends only to previous tokens. This is the right architecture for *generation* — predicting the next token. Modern LLMs (GPT-4, Claude, Llama) are all decoder-only.

---

**6.** 1,000 tokens → 1,000 × 1,000 = **1,000,000 attention pairs** (or ~500,000 unique pairs). Attention cost scales **quadratically** with sequence length (O(n²) in memory and computation). For very long contexts — 100,000+ tokens — this becomes expensive. This is why long-context models are harder and more expensive to build and run, and why research into **efficient attention** (sparse attention, linear attention, state space models like Mamba) exists.

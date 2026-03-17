# Quiz: RNNs and LSTMs

---

## Questions

**1.** What is the key architectural feature that allows RNNs to handle sequential data, and what does it represent?

---

**2.** What is the vanishing gradient problem in RNNs, and what causes it?

---

**3.** Which of the following tasks would benefit most from an architecture that handles long-range dependencies well?

a) Classifying the sentiment of a 5-word tweet
b) Predicting the next word in "I want a cup of ___"
c) Understanding whether the pronoun "it" in the final paragraph of a document refers to a subject mentioned in the first paragraph
d) Detecting the language of a short phrase

---

**4.** An LSTM has three gates. What is the purpose of the forget gate?

---

**5.** Give two reasons why Transformers replaced LSTMs as the dominant architecture for language tasks, despite LSTMs being a significant improvement over vanilla RNNs.

---

## Answers

**1.** The **hidden state** — a vector that is updated at each time step and passed to the next step as input. It represents the model's "memory" of everything it has processed so far in the sequence. The same weights are used to update the hidden state at every time step.

---

**2.** During backpropagation through time, gradients must flow backwards through every time step in the sequence. At each step, they are multiplied by the weight matrices. If these multiplications consistently produce values less than 1 (which is common), the gradients shrink exponentially as they flow back through many steps. By the time gradients reach early time steps, they are so small that those parameters receive almost no learning signal. The model can't learn dependencies between distant parts of a sequence.

---

**3.** c) Understanding whether a pronoun in the final paragraph refers to a subject in the first paragraph. This requires maintaining and accessing information across very long distances in the sequence. Options (a), (b), and (d) all involve short, local context that even a vanilla RNN or simple n-gram model can handle.

---

**4.** The forget gate decides what information to **erase** from the cell state (long-term memory) at each time step. It produces a value between 0 and 1 for each element of the cell state — 0 means "forget completely", 1 means "keep completely". This allows the model to actively discard information that is no longer relevant (e.g. clearing the subject when a new sentence begins) rather than having it slowly decay.

---

**5.** Two reasons:
1. **Sequential computation bottleneck**: RNNs/LSTMs process one token at a time; each step depends on the previous hidden state. This prevents parallelisation across the sequence during training. Transformers process all positions simultaneously using attention, making them dramatically faster to train on modern parallel hardware (GPUs/TPUs).
2. **Information bottleneck**: LSTMs must compress all history into a fixed-size cell state. For very long sequences, important early information still gets lost. Transformers' attention mechanism can directly attend to any position in the sequence, making long-range dependencies explicit and lossless — not compressed into a hidden state.

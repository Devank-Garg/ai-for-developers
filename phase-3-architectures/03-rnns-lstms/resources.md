# Resources: RNNs and LSTMs

---

## Essential reading

**"Understanding LSTM Networks" — Christopher Olah**
https://colah.github.io/posts/2015-08-Understanding-LSTMs/
The definitive introduction to LSTMs. Uses clear diagrams to explain the cell state and all three gates without overwhelming with equations. This post is referenced in virtually every LSTM tutorial that came after it, and with good reason. Read this if you read nothing else in this module.

**"The Unreasonable Effectiveness of Recurrent Neural Networks" — Andrej Karpathy**
https://karpathy.github.io/2015/05/21/rnn-effectiveness/
A compelling demonstration of what RNNs can do — generates Shakespeare, Linux kernel code, and LaTeX papers character-by-character. Captures both the excitement of what was possible and, implicitly, the motivation for the improvements that followed. Written at the peak of RNN dominance (2015).

---

## Understanding the problem RNNs solve

**"The Vanishing Gradient Problem" — Machine Learning Mastery**
https://machinelearningmastery.com/gentle-introduction-vanishing-exploding-gradients-recurrent-neural-networks/
A clear explanation of vanishing and exploding gradients in RNNs — the core limitation that motivated LSTMs. Includes practical strategies for detecting the problem.

---

## Going deeper on LSTMs

**"Illustrated Guide to LSTM's and GRU's" — Michael Phi**
https://towardsdatascience.com/illustrated-guide-to-lstms-and-gru-s-a-step-by-step-explanation-44e9eb85bf21
A visual step-by-step walkthrough of LSTM internals — gate mechanisms, cell state updates, and how information flows. A good companion to the Olah post if you prefer more structured, step-by-step diagrams.

**"A Critical Review of Recurrent Neural Networks for Sequence Learning" — Lipton et al. (2015)**
https://arxiv.org/abs/1506.00019
A thorough survey of RNN architectures and their applications. More academic, but useful if you want the full picture of what was tried before Transformers.

---

## Historical context

**"Long Short-Term Memory" — Hochreiter & Schmidhuber (1997)**
https://www.bioinf.jku.at/publications/older/2604.pdf
The original LSTM paper. Dense and mathematical, but the introduction is readable and explains the vanishing gradient problem clearly. Worth knowing exists; not required reading.

**Sequence to Sequence Learning with Neural Networks — Sutskever, Vinyals, Le (2014)**
https://arxiv.org/abs/1409.3215
Introduced the encoder-decoder architecture using LSTMs that became standard for machine translation — a direct precursor to the Transformer encoder-decoder. The architectural concept carries directly into Phase 3 / 04.

---

## Andrej Karpathy — character-level language model implementation

**min-char-rnn.py (GitHub gist)**
https://gist.github.com/karpathy/d4dee566867f8291f086
A complete character-level RNN language model in ~100 lines of Python. The simplest possible working implementation of autoregressive text generation. Run it on any text file. Understanding this code directly prepares you for understanding GPT-style generation in Phase 4.

---

## Going deeper

**"Attention and Augmented Recurrent Neural Networks" — Olah & Carter, Distill (2016)**
https://distill.pub/2016/augmented-rnns/
Written just before Transformers arrived, this Distill article explores how attention was being added to RNNs to address their limitations. A fascinating historical snapshot of the problem that Transformers would solve — and the conceptual bridge between RNNs and the attention mechanism.

**"The Illustrated Seq2Seq with Attention" — Jay Alammar**
https://jalammar.github.io/visualizing-neural-machine-translation-mechanics-of-seq2seq-models-with-attention/
Jay Alammar's visual walkthrough of sequence-to-sequence models with attention — the direct architectural predecessor to the Transformer. Shows the encoder-decoder pattern and attention mechanism as they appeared in RNN-based machine translation. Read this before "The Illustrated Transformer" for maximum clarity.

**"Recurrent Neural Networks cheatsheet" — Stanford CS230**
https://stanford.edu/~shervine/teaching/cs-230/cheatsheet-recurrent-neural-networks
Stanford's CS230 RNN reference sheet. Covers vanilla RNN, LSTM, GRU, and bidirectional RNNs in one structured document. A good quick reference when you encounter RNN-related terms in papers or documentation.

**"GRU (Gated Recurrent Unit)" — Cho et al. (2014)**
https://arxiv.org/abs/1406.1078
Introduced the GRU — a simplified LSTM with two gates instead of three. GRUs are often used in practice because they are faster to train with comparable performance. Worth knowing exists if you encounter GRU references in model documentation.

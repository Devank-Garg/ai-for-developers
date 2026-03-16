# Resources: Optimization Reference

---

## The definitive overview

**"An overview of gradient descent optimization algorithms" — Sebastian Ruder**
https://ruder.io/optimizing-gradient-descent/
The most-cited explainer on ML optimisers. Covers SGD, Momentum, Adagrad, RMSProp, Adam, and variants — with equations, intuitive descriptions, and animated visualisations of how each navigates a loss surface. Required reading if you plan to train models yourself. Updated periodically; check for the latest version.

---

## Original papers (for reference)

**"Adam: A Method for Stochastic Optimization" — Kingma & Ba (2014)**
https://arxiv.org/abs/1412.6980
The original Adam paper. Section 1 (introduction) and the algorithm box are readable without deep math. Widely cited; worth knowing exists.

**"Decoupled Weight Decay Regularization" — Loshchilov & Hutter (2017)**
https://arxiv.org/abs/1711.05101
The AdamW paper. Short introduction explains clearly why the original Adam's weight decay is broken and how the fix works. Directly relevant to anyone fine-tuning Transformers.

**"Cyclical Learning Rates for Training Neural Networks" — Smith (2017)**
https://arxiv.org/abs/1506.01186
Introduced the idea of cyclically varying the learning rate during training. The 1cycle policy built from this is the default fast.ai schedule and widely used in practice.

---

## Visual and interactive

**"Why Momentum Really Works" — Gabriel Goh, Distill.pub**
https://distill.pub/2017/momentum/
An interactive visual explanation of momentum optimisation — arguably the clearest such explanation online. Distill.pub articles are peer-reviewed, interactive, and consistently excellent. This one lets you drag parameters and watch momentum vs SGD behaviour.

**Optimizer visualisation (various)**
Search for "optimizer visualisation loss surface" — several blog posts and GitHub repos (e.g. Emilien Dupont's optimizer-landscapes) animate how different optimisers traverse the same loss surface. These build strong intuition for why Adam outperforms SGD on most deep learning tasks.

---

## Practical guides

**fast.ai — "How to Train Your Model" (Lesson 4, Practical Deep Learning)**
https://course.fast.ai/
Jeremy Howard's lessons on the learning rate finder (a technique to automatically find a good learning rate), one-cycle training, and mixed precision. Hands-on, opinionated, and production-relevant.

**"A Recipe for Training Neural Networks" — Andrej Karpathy**
https://karpathy.github.io/2019/04/25/recipe/
A practitioner's guide to the full training workflow — including optimiser choice, learning rate tuning, diagnosing training instabilities, and knowing when to change what. One of the most useful short posts in ML engineering.

---

## Reference documentation

**PyTorch torch.optim**
https://pytorch.org/docs/stable/optim.html
Official API reference for all PyTorch optimisers: `Adam`, `AdamW`, `SGD`, and schedules (`CosineAnnealingLR`, `OneCycleLR`, etc.). Look here when configuring training runs.

**Hugging Face Transformers — Training arguments**
https://huggingface.co/docs/transformers/main_classes/trainer#transformers.TrainingArguments
The parameters that control optimisation in HuggingFace's `Trainer`: learning rate, warmup, weight decay, gradient clipping. Directly applicable if you fine-tune LLMs in Phase 4.

# Quiz: Optimization Reference

---

## Questions

**1.** What is the key advantage of Adam over basic SGD?

---

**2.** You are fine-tuning a Transformer model. Which optimiser would you choose, and why?

---

**3.** A training run starts well but then the loss suddenly spikes to `NaN` after a few thousand steps. What is the likely cause, and what would you add to prevent it?

---

**4.** What does a "learning rate warmup" do, and why is it used in Transformer training?

---

**5.** True or false: weight decay and dropout both reduce overfitting, but they do so by different mechanisms. Briefly describe how each works.

---

## Answers

**1.** Adam adapts the learning rate individually for each parameter based on its gradient history. Parameters that receive large, consistent gradients get smaller step sizes (preventing overshooting); parameters with small or inconsistent gradients get larger steps (accelerating learning). This makes Adam less sensitive to the global learning rate setting and faster to converge than SGD, which applies the same step size to every parameter.

---

**2.** **AdamW** — the standard choice for Transformers and LLMs. AdamW is Adam with correctly decoupled weight decay, which is important for Transformer training. The original Adam implementation conflates weight decay with L2 regularisation in a way that behaves poorly in adaptive optimisers; AdamW fixes this. Pair it with a linear warmup + decay learning rate schedule.

---

**3.** **Exploding gradients** — during backpropagation, gradients through deep layers can multiply to very large values, causing parameter updates that are enormous and destabilise training. The fix is **gradient clipping** (`max_grad_norm=1.0` is standard): before the parameter update, scale down the gradient vector if its norm exceeds the threshold. This is a routine part of LLM training configs.

---

**4.** Learning rate warmup starts training with a very small learning rate and gradually increases it over the first few hundred to few thousand steps before beginning to decay. In Transformers, random initialisation means early gradients are noisy and large; starting with a high learning rate can push parameters into bad regions they never recover from. Warmup lets the model stabilise before taking large steps. It's a standard part of the training schedule for LLMs and BERT-style models.

---

**5.** **True.**
- **Weight decay (L2 regularisation)**: penalises large parameter values by adding a term to the loss proportional to the squared magnitude of the weights. This pushes weights toward zero during training, preventing any single parameter from dominating — effectively keeping the model simpler.
- **Dropout**: randomly zeroes out a fraction of activations during each training step. This prevents co-adaptation — the network can't rely on any specific set of neurons always being present, so it learns more distributed, redundant representations. Disabled at inference time.

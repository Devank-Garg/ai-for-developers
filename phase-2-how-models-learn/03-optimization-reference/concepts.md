# Concepts: Optimization Reference

---

## What optimisation means here

In ML, **optimisation** refers to the algorithms that update model parameters to minimise the loss. These are all variations of gradient descent — they differ in *how* they use gradients and *how much* they adapt the step size.

This is a reference module. You don't need to tune these deeply to build with LLMs — but knowing the vocabulary helps you read papers, configure training runs, and understand why models behave as they do.

---

## SGD: the baseline

**Stochastic Gradient Descent** is the simplest optimiser. For each mini-batch:

```
parameter = parameter - learning_rate × gradient
```

SGD with a fixed learning rate is the conceptual baseline everything else is compared against. It works, but it's slow to converge and sensitive to learning rate choice.

**SGD with momentum** adds a velocity term — rather than jumping in the direction of the current gradient, it accumulates a moving average of past gradients. This helps the optimiser move faster in consistent directions and dampens oscillation in others. The momentum hyperparameter (typically 0.9) controls how much past gradient history is retained.

---

## Adam: the default

**Adam** (Adaptive Moment Estimation) is the most widely used optimiser in deep learning. It adapts the learning rate individually for each parameter based on the history of its gradients.

It maintains two running averages:
- **First moment (m)**: mean of past gradients (like momentum)
- **Second moment (v)**: mean of squared past gradients (tracks variance)

The effective learning rate for each parameter is scaled by `1 / √v` — parameters with large, consistent gradients get smaller step sizes; parameters with small or noisy gradients get larger ones.

**Why Adam is the default**: it converges faster than SGD, is less sensitive to learning rate choice, and handles sparse gradients well (important for NLP). For most new projects, start with Adam.

**AdamW** is a variant of Adam with corrected weight decay (regularisation). It's the standard choice for training LLMs and Transformers.

---

## Learning rate

The **learning rate** (abbreviated `lr`) is the most important hyperparameter in training. Too high: loss diverges. Too low: training is slow or gets stuck.

Typical starting values: `3e-4` (0.0003) for Adam is a common starting point; SGD usually needs a higher initial rate (e.g. 0.01–0.1).

**Learning rate schedules** adjust the learning rate during training:

- **Step decay** — reduce by a fixed factor (e.g. ×0.1) every N epochs
- **Cosine annealing** — follow a cosine curve from a high initial value to near zero, optionally with "restarts"
- **Warmup** — start with a very small learning rate and ramp up over the first N steps. Used in Transformer training to stabilise early-stage updates.
- **Linear decay with warmup** — the standard schedule for LLM fine-tuning: warmup for a few hundred steps, then linearly decay to zero.

---

## Weight decay and regularisation

**Weight decay** (also called **L2 regularisation**) adds a penalty to the loss proportional to the magnitude of the weights:

```
loss = original_loss + λ × Σ(weights²)
```

This encourages the optimiser to keep weights small, which prevents overfitting. The `λ` hyperparameter controls the regularisation strength. Common values: `1e-4` to `1e-2`.

**L1 regularisation** uses the sum of absolute values instead of squares. It tends to produce sparse weights (many exactly zero), which can act as implicit feature selection.

**Dropout** is a different regularisation approach: during training, randomly set a fraction of activations to zero on each forward pass, forcing the network to not rely on any single path. Disabled at inference time. Dropout rate of 0.1–0.5 is common.

---

## Gradient clipping

In very deep networks or RNNs, gradients can become extremely large during backpropagation — **exploding gradients** — causing training to destabilise. **Gradient clipping** caps the gradient norm at a maximum value (typically 1.0) before the update step.

This is a standard part of LLM training. If you see `max_grad_norm=1.0` in a training config, that's gradient clipping.

---

## Summary

| Technique | What it does | When to use |
|-----------|-------------|-------------|
| SGD + momentum | Basic update with direction memory | Image models, when Adam overfits |
| Adam | Adaptive per-parameter learning rates | Default for most models |
| AdamW | Adam with corrected weight decay | Standard for Transformers and LLMs |
| Learning rate warmup | Ramp up lr at training start | Transformer training |
| Weight decay | Penalise large weights | Almost always |
| Gradient clipping | Cap gradient magnitude | Deep networks, RNNs, LLMs |

# Concepts: The Training Loop

---

## The loop in plain terms

Training a model is a repeated cycle:

```
1. Forward pass   — run input through the model, get a prediction
2. Compute loss   — measure how wrong the prediction was
3. Backward pass  — calculate how each parameter contributed to the error
4. Update         — adjust parameters slightly to reduce the loss
5. Repeat
```

This loop runs millions or billions of times. Each iteration is called a **step** or **iteration**. When the loop has processed every sample in the dataset once, that's one **epoch**.

---

## Forward pass

The **forward pass** is the computation that takes an input, passes it through every layer of the model in sequence, and produces an output (a prediction).

Nothing changes during the forward pass. Parameters are frozen; data flows through the network from input to output. The result is a prediction and, after comparing to the label, a loss value.

---

## Backward pass and backpropagation

**Backpropagation** is the algorithm that computes how much each parameter contributed to the loss. It works backwards through the network — from the loss, back through each layer — applying the **chain rule** of calculus to compute the **gradient** of the loss with respect to each parameter.

The gradient is a direction: it tells you which way to move each parameter to increase the loss. To decrease the loss, you move in the opposite direction.

You don't need to understand the calculus to use this. What matters is the concept: backprop answers the question "which knobs, turned which way, would have reduced the error?" for every single parameter in the model, in one efficient pass.

---

## Gradient descent update

After backprop, each parameter gets a gradient. The **update rule** is:

```
parameter = parameter - (learning_rate × gradient)
```

**Learning rate** controls the step size. Too large and the model overshoots; too small and training is slow or gets stuck. It's the single most important hyperparameter in training.

This process — computing gradients, then stepping in the direction that reduces loss — is **gradient descent**.

---

## Batches and mini-batches

Running one full forward + backward pass on the entire dataset before updating is called **batch gradient descent**. It's computationally expensive and memory-intensive for large datasets.

**Stochastic gradient descent (SGD)** updates after every single sample. Fast but noisy — the gradient estimate from one sample is a rough approximation of the true gradient.

**Mini-batch gradient descent** is the standard: process a small batch of samples (typically 32–512), compute the average gradient across the batch, then update. This balances computation efficiency with gradient quality.

When someone says "batch size" in a training context, they almost always mean mini-batch size.

---

## Epochs and convergence

One **epoch** = one pass through the full training dataset. Models are typically trained for many epochs. How many depends on the model, dataset, and whether validation performance has stopped improving.

**Convergence** is when the loss stops decreasing meaningfully. Signs of convergence:
- Training loss has flattened
- Validation loss has stopped improving (or is getting worse — a sign of overfitting)

**Early stopping** is a regularisation technique: monitor validation loss, and stop training when it stops improving, even if training loss is still decreasing.

---

## What happens in practice

Training a large model looks like this:

1. Data is loaded in mini-batches
2. Each batch runs forward pass, loss, backward pass, parameter update
3. Every N steps, log training loss and run evaluation on the validation set
4. Adjust learning rate on schedule
5. Save checkpoints periodically
6. Stop when validation metrics plateau or budget runs out

The "training loop" in code is literally a `for` loop around these steps.

---

## Summary

| Term | Definition |
|------|-----------|
| Forward pass | Input → prediction, no parameter changes |
| Loss | How wrong the prediction was |
| Backward pass | Compute gradients via backpropagation |
| Gradient | Direction to adjust each parameter to increase loss |
| Update | Subtract learning_rate × gradient from each parameter |
| Batch size | Number of samples processed before each update |
| Epoch | One full pass through the training dataset |
| Early stopping | Halt training when validation performance plateaus |

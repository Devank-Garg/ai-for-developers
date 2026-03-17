# Quiz: The Training Loop

---

## Questions

**1.** List the four steps of the training loop in order.

---

**2.** A model has 1 million parameters. After one backward pass, how many gradients are computed?

a) 1
b) One per layer
c) One per training sample
d) One per parameter — 1 million

---

**3.** You set the learning rate too high. What symptom would you observe during training?

---

**4.** A dataset has 100,000 samples. You train with a batch size of 500. How many parameter updates (steps) occur per epoch?

---

**5.** Your model's training loss is 0.08 and still declining, but your validation loss has been 0.21 for the last 5 epochs and is slowly rising. What is happening, and what should you do?

---

**6.** What is the conceptual purpose of backpropagation? You don't need to explain the math.

---

## Answers

**1.** Forward pass → Compute loss → Backward pass (backpropagation) → Update parameters. Then repeat.

---

**2.** d) One per parameter — 1 million. Backpropagation computes the gradient of the loss with respect to every single parameter in the model. That's exactly what makes it powerful — and computationally expensive for large models.

---

**3.** The loss would oscillate wildly or diverge (increase rather than decrease). With too large a step size, the parameter update overshoots the minimum — you jump past it to the other side, then overshoot back, and never converge. In severe cases the loss becomes `NaN` or explodes to infinity.

---

**4.** 100,000 samples ÷ 500 batch size = **200 steps per epoch**. Each step processes one mini-batch and performs one parameter update.

---

**5.** This is **overfitting**. The model has learned the training data well (training loss is low and improving) but is failing to generalise to new data (validation loss is rising). You should apply **early stopping** — stop training at the epoch where validation loss was lowest. Additionally consider: increasing regularisation (dropout, weight decay), reducing model size, or adding more training data.

---

**6.** Backpropagation efficiently answers: "for every parameter in the model, how much did it contribute to the current error, and in which direction should it be adjusted to reduce that error?" It does this by applying the chain rule backwards through the network in a single pass, rather than computing each parameter's contribution separately (which would be impossibly slow for models with billions of parameters).

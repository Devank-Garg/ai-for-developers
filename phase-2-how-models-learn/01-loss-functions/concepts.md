# Concepts: Loss Functions

---

## What loss means

A **loss function** (also called a **cost function** or **objective function**) measures how wrong a model's prediction is. It takes the model's output and the correct answer, and returns a single number — the **loss**. Training is the process of minimising this number.

The choice of loss function determines what "wrong" means for your problem. A loss function that penalises being 10% off by the same amount regardless of direction is different from one that penalises underestimates more than overestimates. Getting this choice right matters.

---

## Mean Squared Error (regression)

**MSE** is the standard loss for regression problems:

```
MSE = (1/n) × Σ(predicted - actual)²
```

Squaring the errors does two things: makes all errors positive, and penalises large errors disproportionately. A prediction that's off by 10 contributes 100 to the loss; one off by 1 contributes only 1.

This makes MSE sensitive to outliers. **Mean Absolute Error (MAE)** — the average of absolute differences — is more robust but harder to optimise mathematically (it has no gradient at zero).

**Root Mean Squared Error (RMSE)** is just √MSE, restoring the original units of the prediction.

---

## Cross-entropy loss (classification)

For classification problems, the model outputs a probability for each class. **Cross-entropy loss** measures how well the predicted probability distribution matches the actual one:

```
Loss = -log(predicted probability of the correct class)
```

If the model assigns 90% probability to the correct class, the loss is −log(0.9) ≈ 0.1 — small. If it assigns 10% probability, the loss is −log(0.1) ≈ 2.3 — large. The log function punishes confident wrong answers harshly.

For binary classification (two classes), this simplifies to **binary cross-entropy**. For multi-class problems, **categorical cross-entropy** (also called **softmax loss**) is the standard choice.

---

## Loss surfaces and local minima

Visualise the loss as a landscape of hills and valleys. Each point on the landscape represents a configuration of the model's parameters; the height is the loss at that configuration. Training is the process of navigating downhill to find the lowest valley.

Key concepts:

**Global minimum** — the absolute lowest loss achievable. In theory, what you want. In practice, impossible to guarantee you've found it.

**Local minimum** — a valley that's lower than its immediate surroundings but not the global lowest point. In practice, deep learning models rarely get trapped here — large models have highly non-convex loss surfaces with many saddle points, and optimisers navigate them surprisingly well.

**Saddle points** — flat regions where the gradient is near zero in all directions but it's not a minimum. These can slow training.

**Loss landscape** research shows that well-trained large models tend to find "flat" minima — wide valleys where small parameter perturbations don't change the loss much. These flat minima tend to generalise better than sharp ones.

---

## Summary

| Loss function | Use for | Key property |
|--------------|---------|--------------|
| MSE | Regression | Penalises large errors heavily |
| MAE | Regression | Robust to outliers |
| Binary cross-entropy | Binary classification | Penalises confident wrong answers |
| Categorical cross-entropy | Multi-class classification | Extends to probability distributions |

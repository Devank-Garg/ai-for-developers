# Quiz: Loss Functions

---

## Questions

**1.** A regression model predicts house prices. One prediction is off by ₹10 lakh; another is off by ₹100 lakh. Under MSE, how much more does the second error contribute to the loss than the first?

---

**2.** A classification model outputs these probabilities for a 3-class problem: [Cat: 0.7, Dog: 0.2, Bird: 0.1]. The correct answer is "Cat". What is the cross-entropy loss (approximately)?

a) 0.15
b) 0.36
c) 0.70
d) 1.0

---

**3.** Why is cross-entropy preferred over MSE for classification tasks?

---

**4.** A model is 90% confident the answer is "fraud" but the transaction is actually legitimate. Is the cross-entropy loss for this example high or low? Why does this matter?

---

**5.** What is a "flat minimum" in the loss landscape, and why might it generalise better than a sharp minimum?

---

## Answers

**1.** MSE squares the errors. ₹10 lakh error → 10² = 100. ₹100 lakh error → 100² = 10,000. The second contributes **100× more** to the loss. This is what makes MSE sensitive to outliers — large errors are penalised disproportionately.

---

**2.** b) 0.36. Cross-entropy for the correct class = −log(0.7) ≈ 0.36. Only the probability assigned to the correct class matters.

---

**3.** MSE was designed for continuous outputs. For classification, the model outputs probabilities (0–1). MSE applied to probabilities behaves poorly — it has a flat gradient in regions where the model is wrong but confident, making learning slow. Cross-entropy's logarithm produces strong gradients precisely when the model is confidently wrong, which is exactly when you want learning to be fast.

---

**4.** The loss is **high**. Cross-entropy = −log(1 − 0.9) = −log(0.1) ≈ 2.3. The model assigned only 10% probability to the correct class (legitimate) while being 90% confident it was fraud. Cross-entropy heavily penalises confident errors, which creates a strong gradient signal to correct this mistake during training.

---

**5.** A flat minimum is a wide valley where small perturbations to the parameters barely change the loss. A sharp minimum is a narrow spike — small parameter changes dramatically increase the loss. Models at flat minima tend to generalise better because, in production, data distributions shift slightly from training distributions. A model at a flat minimum is robust to these small shifts; a model at a sharp minimum may perform poorly as soon as parameters or inputs move slightly.

# Resources: Loss Functions

---

## Visual introductions

**3Blue1Brown — "Gradient descent, how neural networks learn" (Neural Networks, Chapter 2)**
https://www.youtube.com/watch?v=IHZwWFHWa-w
Grant Sanderson's visual explanation of the loss surface and gradient descent. The animated loss landscape makes the concept of "navigating downhill" genuinely intuitive. Part of the best neural network series on the internet. Watch Chapter 1 first if you haven't.

**3Blue1Brown — "What is backpropagation really doing?" (Chapter 3)**
https://www.youtube.com/watch?v=Ilg3gGewQ5U
Directly follows the loss function video; connects loss to the gradient updates that change the model.

---

## Conceptual explainers

**"A Gentle Introduction to Cross-Entropy for Machine Learning" — Jason Brownlee, Machine Learning Mastery**
https://machinelearningmastery.com/cross-entropy-for-machine-learning/
The clearest written explanation of cross-entropy available. Covers entropy, KL divergence, and how cross-entropy loss connects to probability theory. Well-structured with formulas and code examples.

**"Loss Functions in Machine Learning Explained" — ML Glossary**
https://ml-cheatsheet.readthedocs.io/en/latest/loss_functions.html
A concise reference covering MSE, MAE, cross-entropy, and others with equations and graphs. Good as a lookup when you encounter an unfamiliar loss function later in the course.

---

## For deeper understanding

**"Visualising the Loss Landscape of Neural Nets" — Li et al. (2018)**
https://arxiv.org/abs/1712.09913
The academic paper behind the "loss landscape" visualisations that appear everywhere. The images of flat vs sharp minima are compelling. Abstract and introduction are readable; full paper is technical.

**"Sharp Minima Can Generalise For Deep Nets" — Dinh et al. (2017)**
https://arxiv.org/abs/1703.04933
A counterpoint to the flat-minima-generalise idea — worth knowing the debate exists. Not required reading, but good if you hear conflicting claims about minima and generalisation.

---

## Reference

**PyTorch Loss Functions Documentation**
https://pytorch.org/docs/stable/nn.html#loss-functions
Official reference for all loss functions available in PyTorch: `MSELoss`, `CrossEntropyLoss`, `BCELoss`, and more. Useful when you start implementing models in Phase 5.

**TensorFlow / Keras Losses Documentation**
https://www.tensorflow.org/api_docs/python/tf/keras/losses
The TensorFlow equivalent. Same conceptual coverage; useful if your team uses Keras/TensorFlow.

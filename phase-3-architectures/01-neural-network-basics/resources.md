# Resources: Neural Network Basics

---

## The best visual series

**3Blue1Brown — Neural Networks (full series)**
https://www.3blue1brown.com/topics/neural-networks
Grant Sanderson's 4-part visual series on neural networks. Chapter 1 ("But what is a neural network?") is the finest introduction to layers, neurons, and activation functions available — fully animated, no calculus required. This series is the recommended pre-reading for the entire course. Watch before or alongside this module.

Individual videos:
- Chapter 1: https://www.youtube.com/watch?v=aircAruvnKk — What is a neural network?
- Chapter 2: https://www.youtube.com/watch?v=IHZwWFHWa-w — Gradient descent and the loss
- Chapter 3: https://www.youtube.com/watch?v=Ilg3gGewQ5U — What is backpropagation?
- Chapter 4: https://www.youtube.com/watch?v=tIeHLnjs5U8 — Backpropagation calculus

---

## Free online book

**"Neural Networks and Deep Learning" — Michael Nielsen**
http://neuralnetworksanddeeplearning.com/
A free online book that builds from single neurons to full backpropagation to convolutional networks. Chapters 1–2 are directly relevant to this module. The writing is exceptionally clear — it earns its place among the best explanations of how neural networks actually work. If you prefer reading to watching, start here.

---

## Interactive tools

**TensorFlow Neural Network Playground**
https://playground.tensorflow.org/
An in-browser simulation where you can build small networks, configure layers and activations, and watch training in real time. Change the activation function and watch what the decision boundary can and cannot express. Essential for building intuition about what different architectural choices do.

**CNN Explainer (for when you reach the CNNs module)**
https://poloclub.github.io/cnn-explainer/
An interactive visual walk-through of a convolutional neural network processing an image in real time. Bookmark for the next module.

---

## Written reference

**"Activation Functions in Neural Networks" — Towards Data Science**
https://towardsdatascience.com/activation-functions-neural-networks-1cbd9f8d91d6
A solid survey of common activation functions — ReLU, sigmoid, tanh, GELU, Swish — with graphs and practical guidance on when to use each. Useful as a reference when you encounter unfamiliar activations.

**CS231n — "Neural Networks Part 1: Setting up the Architecture"**
https://cs231n.github.io/neural-networks-1/
Stanford's deep learning for vision course notes. The section on neural network architecture is dense but authoritative — covers layer types, activation functions, and output neurons in detail. Good reference material.

---

## For deeper implementation

**Andrej Karpathy — micrograd**
https://github.com/karpathy/micrograd
https://www.youtube.com/watch?v=VMj-3S1tku0 (video walkthrough)
Building a working neural network and backpropagation engine from scratch. The most valuable exercise for understanding what a neural network actually is at the computational level.

---

## Going deeper

**"Deep Learning" textbook — Goodfellow, Bengio, Courville (free online)**
https://www.deeplearningbook.org/
The canonical academic reference on deep learning. Chapters 6 (deep feedforward networks), 7 (regularisation), and 8 (optimisation) are directly relevant to this module. Dense but authoritative — useful as a reference when you encounter unfamiliar terms.

**"A Recipe for Training Neural Networks" — Andrej Karpathy**
https://karpathy.github.io/2019/04/25/recipe/
A practitioner's guide to the actual process of training neural networks — debugging, diagnosing failures, iterating. Opinionated and practical. The perspective of someone who has trained a lot of models.

**Distill.pub — "A Gentle Introduction to Graph Neural Networks"**
https://distill.pub/2021/gnn-intro/
Distill's interactive articles set the standard for ML visualisation. This one is on GNNs (not required), but the visual format is worth experiencing — bookmark Distill.pub as a general-purpose resource for understanding neural network concepts.

**"Yes you should understand backprop" — Andrej Karpathy**
https://karpathy.medium.com/yes-you-should-understand-backprop-e2f06eab496b
A short argument for why understanding backpropagation — not just the forward pass — matters for practitioners. Useful context before moving into training and optimisation in Phase 2.

# Quiz: Neural Network Basics

---

## Questions

**1.** A neural network has: 10 input features, two hidden layers of 64 neurons each, and an output layer with 3 neurons. What does the output layer size tell you about the task?

---

**2.** Why can't you build a useful deep network using only linear activations (no ReLU, sigmoid, etc.)?

---

**3.** Which activation function would you use on the output layer for:
- Predicting tomorrow's temperature (regression)
- Detecting whether an email is spam (binary classification)
- Classifying a photo into one of 1,000 categories

---

**4.** A fully connected layer has 128 input neurons and 64 output neurons. How many weight parameters does it have (not counting biases)?

---

**5.** Why does "depth" (more layers) tend to help, rather than simply using one very wide hidden layer?

---

## Answers

**1.** The task is **multi-class classification with 3 classes**. The output layer has one neuron per class; the model will apply softmax to produce a probability distribution over the three categories. The training label is whichever of the 3 classes each sample belongs to.

---

**2.** A linear activation `f(z) = z` means the neuron outputs exactly its weighted sum — no transformation. Any composition of linear transformations is itself a linear transformation. So a 10-layer network with linear activations is mathematically equivalent to a single linear layer. No depth, no hierarchical representations, no ability to model non-linear patterns. Activation functions are what make depth meaningful.

---

**3.**
- **Temperature (regression)**: no activation on the output neuron — let it output any real number.
- **Spam detection (binary classification)**: **sigmoid** on a single output neuron — produces a probability between 0 and 1.
- **1,000 categories (multi-class classification)**: **softmax** on 1,000 output neurons — produces a probability distribution that sums to 1 across all classes.

---

**4.** 128 × 64 = **8,192 weight parameters**. Each of the 64 output neurons has one weight connecting it to each of the 128 input neurons. (Plus 64 bias terms, for 8,256 total if counting biases.)

---

**5.** Deep networks learn **hierarchical representations**. Early layers learn low-level features (edges, textures, short word patterns), intermediate layers combine them into mid-level concepts, and deep layers represent high-level abstractions. A single wide layer has no such hierarchy — it must learn everything in one transformation. Depth allows the model to build complex concepts from simpler ones compositionally, which is how the world is actually structured. Empirically, deep-but-narrower networks consistently outperform shallow-but-wider ones for complex tasks.

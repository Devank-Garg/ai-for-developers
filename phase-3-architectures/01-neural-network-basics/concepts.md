# Concepts: Neural Network Basics

---

## What a neural network is

A **neural network** is a mathematical function composed of layers of simpler functions. Data flows in, gets transformed by each layer, and an output comes out. The name and visual metaphor come from biological neurons, but the useful analogy stops there — a neural network is a parameterised function, not a brain.

```
Input layer → [Hidden layers] → Output layer
```

---

## Neurons and layers

A single **neuron** (also called a **node** or **unit**) does two things:

1. Takes a weighted sum of its inputs: `z = w₁x₁ + w₂x₂ + ... + b`
2. Applies an **activation function** to that sum: `output = f(z)`

The weights (`w`) and bias (`b`) are the learnable parameters. The activation function determines whether and how strongly the neuron "fires."

A **layer** is a collection of neurons that all receive the same inputs (the previous layer's outputs). A **fully connected** (or **dense**) layer connects every neuron in one layer to every neuron in the next.

**Input layer** — holds the raw features. No computation happens here.
**Hidden layers** — where the model learns representations. "Deep" learning refers to having many hidden layers.
**Output layer** — produces the final prediction. Its size matches the task: one neuron for regression, one per class for classification.

---

## Activation functions

Without activation functions, stacking layers does nothing — a composition of linear functions is still a linear function. Activation functions introduce **non-linearity**, allowing the network to learn complex patterns.

**ReLU** (Rectified Linear Unit): `f(z) = max(0, z)`. Zero for negative inputs, identity for positive. The default hidden layer activation — computationally cheap and works well in practice.

**Sigmoid**: `f(z) = 1 / (1 + e⁻ᶻ)`. Squashes output to (0, 1). Used in binary classification output layers to produce a probability.

**Softmax**: normalises a vector of values to a probability distribution that sums to 1. Used in multi-class classification output layers.

**Tanh**: squashes output to (−1, 1). Used in some hidden layers and in RNNs.

For hidden layers: use ReLU (or its variants like GELU, used in Transformers). For outputs: use sigmoid (binary), softmax (multi-class), or no activation (regression).

---

## Forward pass through a network

On each forward pass, data moves through layers sequentially. Each layer:
1. Takes the previous layer's output as input
2. Multiplies by its weight matrix
3. Adds a bias
4. Applies activation function
5. Passes result to the next layer

The final layer produces the prediction. This is just matrix multiplication and function application — no magic.

---

## Why depth matters

A shallow network (one hidden layer) can theoretically approximate any function, but it may need an exponentially large number of neurons. Deep networks — many layers — learn **hierarchical representations**: early layers learn simple features (edges, n-grams), deeper layers combine them into complex concepts (faces, sentences).

This hierarchical feature learning is why deep networks are powerful and why adding depth often improves performance up to a point.

---

## Summary

| Concept | Definition |
|---------|-----------|
| Neuron | Weighted sum of inputs + activation function |
| Layer | Collection of neurons at the same depth |
| Fully connected layer | Every neuron connects to every neuron in adjacent layers |
| ReLU | Most common hidden layer activation: max(0, z) |
| Softmax | Converts output layer values to a probability distribution |
| Depth | Number of layers; enables hierarchical feature learning |

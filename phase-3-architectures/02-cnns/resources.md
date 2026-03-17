# Resources: Convolutional Neural Networks

---

## Visual and interactive

**CNN Explainer — Polo Club of Data Science, Georgia Tech**
https://poloclub.github.io/cnn-explainer/
An interactive, in-browser visualisation of a CNN processing a real image in real time. Shows convolution, activation, and pooling layer outputs at each step. The best way to build intuition for what CNNs actually compute. No setup required.

**"How Computers See" — 3Blue1Brown (Neural Networks, Chapter 1 — CNN discussion)**
The same 3Blue1Brown neural networks series covers CNNs in the context of digit recognition. Watch Chapter 1 if you haven't already: https://www.youtube.com/watch?v=aircAruvnKk

---

## Conceptual explainers

**"An Intuitive Explanation of Convolutional Neural Networks" — Ujjwal Karn**
https://ujjwalkarn.me/2016/08/11/intuitive-explanation-convnets/
One of the most widely shared written introductions to CNNs. Covers filters, convolution, pooling, and the full architecture with diagrams. Well-paced for developers who haven't done image ML before.

**CS231n — "Convolutional Neural Networks" lecture notes**
https://cs231n.github.io/convolutional-networks/
Stanford's authoritative lecture notes on CNN architecture. More detailed than the Karn post — covers stride, padding, receptive fields, and batch normalisation. Use as a reference when you need depth.

**"Visualising what ConvNets learn" — CS231n**
https://cs231n.github.io/understanding-cnn/
Explains and shows what filters in trained CNNs have actually learned. The visualisations of what deep layers respond to (dog faces, text, wheels) are compelling evidence for the hierarchical representation idea.

---

## Landmark papers (reference)

**"ImageNet Classification with Deep Convolutional Neural Networks" — Krizhevsky, Sutskever, Hinton (2012)**
https://papers.nips.cc/paper_files/paper/2012/hash/c399862d3b9d6b76c8436e924a68c45b-Abstract.html
AlexNet — the paper that launched the deep learning era. Won ImageNet 2012 by a wide margin and demonstrated that deep CNNs could dominate visual recognition. Short, readable introduction.

**"Deep Residual Learning for Image Recognition" — He et al. (2015)**
https://arxiv.org/abs/1512.03385
ResNet — introduced skip connections (residual connections), which allowed training networks 100+ layers deep. Skip connections appear in virtually every modern architecture including Transformers. The concept is worth understanding before Phase 3 / 04.

---

## Practical

**fast.ai — Practical Deep Learning, Lesson 1**
https://course.fast.ai/
Lesson 1 trains an image classifier from scratch and covers CNNs conceptually before diving into code. Includes transfer learning from pre-trained models. Hands-on; requires basic Python.

**PyTorch — `torch.nn.Conv2d` documentation**
https://pytorch.org/docs/stable/generated/torch.nn.Conv2d.html
Reference for the standard 2D convolution layer. Useful when implementing CNNs or reading model architectures.

---

## Going deeper

**"An Image is Worth 16x16 Words: Transformers for Image Recognition at Scale" — Dosovitskiy et al. (2020)**
https://arxiv.org/abs/2010.11929
The Vision Transformer (ViT) paper — applying a Transformer directly to image patches instead of using convolutions. Now the backbone of many multimodal LLMs. Abstract and Figure 1 are sufficient to understand the concept; the full paper is readable.

**"EfficientNet: Rethinking Model Scaling for Convolutional Neural Networks" — Tan & Le (2019)**
https://arxiv.org/abs/1905.11946
EfficientNet is one of the most widely used CNN backbones in production. The paper introduces compound scaling — a principled way to scale depth, width, and resolution simultaneously. Worth understanding if you're working with vision models in production.

**"Feature Visualization" — Olah et al., Distill (2017)**
https://distill.pub/2017/feature-visualization/
An interactive deep-dive into what neurons in CNNs actually respond to — from edges at early layers to complex textures and objects in deep layers. Visually stunning and builds genuine intuition for hierarchical representation learning.

**Torchvision pre-trained models — PyTorch**
https://pytorch.org/vision/stable/models.html
The full list of available pre-trained CNN and Vision Transformer models in PyTorch. Useful reference when choosing a backbone for a vision task or when reading about model architectures.

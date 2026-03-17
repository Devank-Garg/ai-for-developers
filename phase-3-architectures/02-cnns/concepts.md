# Concepts: Convolutional Neural Networks (Optional)

> **Tag: optional** — CNNs are essential for image tasks but not required for LLM work. Read this module if you work with images or want to understand why architectural choices matter before reaching Transformers.

---

## The problem with flat networks for images

A 224×224 pixel colour image has 224 × 224 × 3 = **150,528 values**. A fully connected hidden layer with 1,000 neurons would need 150 million weight parameters — just for the first layer. This is computationally expensive, prone to overfitting, and ignores a key property of images: nearby pixels are related.

A fully connected layer also treats every pixel as independent from every other. It has no built-in concept of spatial structure — a cat in the top-left and a cat in the bottom-right look completely different to it.

CNNs solve both problems.

---

## Convolution: local pattern detection

A **convolutional layer** applies a small filter (also called a **kernel**) across the entire image, sliding it from left to right, top to bottom. At each position, it computes the dot product of the filter weights with the image patch it's currently over.

The result is a **feature map** — a 2D grid showing where in the image that filter found a match.

A filter for detecting horizontal edges will produce a high activation wherever horizontal edges appear. A different filter detects vertical edges. Another detects corners. The model **learns** what filters to use during training.

Key properties:
- **Parameter sharing** — the same filter weights are applied everywhere in the image. A filter that detects a left eye will also detect a right eye, and a cat's eye. This dramatically reduces parameters.
- **Local connectivity** — each neuron connects to only a small patch of the previous layer, not the whole image.
- **Translation invariance** — learned features activate regardless of where in the image they appear (approximately).

---

## Pooling: spatial reduction

After convolution, **pooling** layers reduce the spatial dimensions of feature maps. **Max pooling** takes the maximum value in each small window (e.g. 2×2) and discards the rest. This reduces resolution, concentrates the strongest activations, and makes the representation less sensitive to small translations.

A typical CNN alternates: convolution → activation (ReLU) → pooling. This builds up a hierarchy from fine-grained to coarse-grained features.

---

## CNN architecture pattern

```
Input image
→ Conv + ReLU (detect low-level features: edges, colours)
→ Pooling (reduce spatial size)
→ Conv + ReLU (detect mid-level features: textures, shapes)
→ Pooling
→ Conv + ReLU (detect high-level features: object parts)
→ Flatten
→ Fully connected layers
→ Softmax output
```

Early layers respond to simple patterns. Deep layers respond to complex, semantic features. The final fully connected layers combine these features into a classification.

---

## Why CNNs matter for this course

CNNs established several ideas that carry forward to Transformers and LLMs:
- **Hierarchical representation learning** — building complex concepts from simple ones
- **The value of architectural inductive biases** — baking assumptions about structure (spatial locality) into the architecture reduces what needs to be learned from data
- **Transfer learning** — large CNNs trained on ImageNet are routinely fine-tuned for new tasks, a pattern LLMs repeat at vastly larger scale

---

## Summary

| Concept | What it does |
|---------|-------------|
| Convolution | Slides a filter across input; detects local patterns |
| Feature map | Output of a convolution — spatial map of where a pattern appears |
| Parameter sharing | Same filter weights used everywhere — reduces parameters dramatically |
| Pooling | Reduces spatial dimensions; increases robustness |
| Depth in CNNs | Low layers → edges; mid layers → textures; deep layers → objects |

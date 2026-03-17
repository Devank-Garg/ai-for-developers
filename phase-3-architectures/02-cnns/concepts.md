# Concepts: Convolutional Neural Networks (Optional)

> **Tag: optional** — CNNs are essential for image tasks but not required for LLM work. Read this module if you work with images or want to understand why architectural choices matter before reaching Transformers.

---

## Why a developer needs this

When you call an image classification API, a vision model, or any service that processes images, a CNN (or more recently, a Vision Transformer — a CNN-inspired architecture) is running underneath. Understanding CNNs tells you:

- Why image models generalise well across different positions and scales
- Why transfer learning from a pre-trained image model is so powerful
- Why the first layers of any vision system detect edges before it detects objects
- How the Vision Transformer (ViT) — used in modern multimodal LLMs — relates to CNNs

If you work with image input at all in Phase 5 or Phase 6, this module is worth the time.

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

## What this means when you're building

**Transfer learning in practice**: most production image models are not trained from scratch. You take a large CNN (e.g., ResNet, EfficientNet) pre-trained on ImageNet, strip the final classification layer, and replace it with a new one trained on your data. The convolutional layers — which learned to detect edges, textures, and shapes — transfer directly. This is exactly the same pattern LLMs use at vastly larger scale (Phase 4).

**Image embedding APIs**: services that turn an image into a vector (for search, recommendation, or classification) are running an image through a CNN or Vision Transformer and returning the output of a specific layer before the final classifier. The vector *is* the feature representation the CNN learned.

**Vision Transformers (ViT)**: modern multimodal LLMs (including those that accept image inputs) often use a Vision Transformer rather than a CNN. ViT splits an image into fixed-size patches, treats each patch as a token, and runs it through a Transformer. The inductive biases are different but the output — a rich spatial representation — serves the same purpose.

---

## Summary

| Concept | What it does |
|---------|-------------|
| Convolution | Slides a filter across input; detects local patterns |
| Feature map | Output of a convolution — spatial map of where a pattern appears |
| Parameter sharing | Same filter weights used everywhere — reduces parameters dramatically |
| Pooling | Reduces spatial dimensions; increases robustness |
| Depth in CNNs | Low layers → edges; mid layers → textures; deep layers → objects |

# Quiz: Convolutional Neural Networks

---

## Questions

**1.** Why is a fully connected network inefficient for image inputs, and how does convolution address this?

---

**2.** A CNN filter is trained and learns to detect vertical edges. If the cat's whiskers appear on the left side of the image in training data, will the filter also detect whiskers on the right side? Why?

---

**3.** What does max pooling do, and what is the cost of applying it?

---

**4.** What patterns do early vs deep layers in a CNN typically learn?

---

**5.** A CNN trained on 1 million labelled cat/dog images is now being adapted to classify 10 species of birds. What approach would you take, and why?

---

## Answers

**1.** A fully connected network applied to a 224×224×3 image would need ~150 million parameters in the first layer alone, is computationally intractable, and ignores spatial structure — pixels near each other are semantically related, but a dense layer treats them all as independent. Convolution applies small filters locally and shares the same weights across all spatial positions, reducing parameters dramatically while exploiting the spatial structure of images.

---

**2.** **Yes** — because of **parameter sharing**. The same filter weights are applied at every position in the image during the convolution operation. The filter doesn't have a "position" — it slides across the entire image equally. If it's learned to detect a pattern, it detects that pattern wherever it appears. This is what makes CNNs approximately **translation invariant**.

---

**3.** Max pooling takes the maximum value in a small spatial window (e.g. 2×2) and discards the rest, reducing the spatial dimensions of the feature map (typically by half). This compresses the representation, reduces computation in later layers, and makes feature detection less sensitive to exact position. The **cost**: spatial information is lost — you know a feature was present somewhere in the window, but not exactly where.

---

**4.** **Early layers** respond to low-level features: edges, corners, colour gradients. **Middle layers** combine these into textures, shapes, and parts (like eyes, wheels). **Deep layers** activate for high-level, semantic concepts: faces, animals, objects. This hierarchy — from simple to complex — mirrors how we think about visual perception and is one of the reasons CNNs work so well.

---

**5.** **Transfer learning / fine-tuning.** Keep the convolutional layers (which have learned general visual features — edges, textures, shapes) and replace only the final classification layer (changing from 2 outputs to 10). Freeze the early layers and train only the later ones, or fine-tune the whole network at a low learning rate. This works because low-level visual features are universal; you only need to retrain the high-level decision boundary for the new classes. This requires far less data and training time than starting from scratch.

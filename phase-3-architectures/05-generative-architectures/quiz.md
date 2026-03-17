# Quiz: Generative Architectures

---

## Questions

**1.** What does it mean for a generative model to "learn the distribution" of its training data?

---

**2.** In a GAN, what is mode collapse, and why is it a problem?

---

**3.** Diffusion models generate images by reversing a noising process. Starting from what input, and what is the output?

---

**4.** Stable Diffusion is described as a "latent diffusion model." What does this mean, and why does it matter practically?

---

**5.** A colleague says "LLMs are just diffusion models for text." Is this accurate? Briefly explain the difference.

---

## Answers

**1.** Training data comes from some underlying distribution — the set of all plausible cat photos, or all plausible English sentences. A generative model learns to approximate that distribution well enough to **sample new examples that could plausibly have come from it**. It doesn't memorise training examples; it learns the patterns, statistics, and structure of the data well enough to produce novel instances that look like they belong to the same population.

---

**2.** Mode collapse is when the generator learns to produce only a narrow range of outputs — perhaps a few specific images that reliably fool the discriminator — ignoring the full diversity of the training data. For example, a GAN trained on faces might generate only one type of face at many different angles, rather than the full diversity of the training set. It's a problem because the model is technically "fooling" the discriminator but failing at the actual goal of learning the full data distribution. The discriminator provides no signal to explore diversity, only to be realistic.

---

**3.** Diffusion models at inference start from **pure Gaussian noise** (a random noise image) and iteratively denoise over many steps until a clean image emerges. The output is a realistic image consistent with the training distribution (and, in conditional models, consistent with the input text prompt).

---

**4.** Standard (pixel-space) diffusion runs the noising/denoising process directly on the image's pixel values — computationally expensive for high-resolution images. Latent diffusion models first compress the image into a lower-dimensional **latent space** using a VAE encoder, then run diffusion in that compact space. The VAE decoder converts the final denoised latent back to pixels. This dramatically reduces computation — denoising in latent space is orders of magnitude cheaper than in pixel space — making high-quality image generation practical for consumer hardware.

---

**5.** **Not accurate.** LLMs are **autoregressive** generative models: they generate text by predicting the next token conditioned on all previous tokens, one token at a time. Each step is a single forward pass of a Transformer. Diffusion models generate by iteratively denoising from noise over many steps, with all output dimensions refined simultaneously. LLMs don't add noise to text and reverse it — the generation mechanisms are fundamentally different. The end goal (generating plausible new examples) is similar, but the training objective, inference process, and architecture are distinct.

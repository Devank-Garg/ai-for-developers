# Concepts: Generative Architectures (Optional)

> **Tag: optional** — This module covers generative model families beyond language. Useful context for understanding image generation and multimodal systems (Phase 4 / 06). Not required for building LLM applications.

---

## Why a developer needs this

If you call an image generation API (DALL-E, Stable Diffusion, Midjourney, Imagen), a diffusion model is running underneath. If you use a multimodal LLM that accepts image inputs, a CLIP-based image encoder is producing the image representation. If you work with any generative model that is not a language model, it is almost certainly a diffusion model or a GAN.

This module gives you the conceptual vocabulary to:
- Understand why image generation is slow (diffusion inference requires many sequential denoising steps)
- Understand why Stable Diffusion can run on a laptop but some models cannot (latent diffusion)
- Understand what "text conditioning" means in a text-to-image model
- Interpret API parameters like `steps`, `guidance_scale`, and `negative_prompt`

---

## What generative models do

A generative model learns the underlying distribution of its training data well enough to **sample new examples from that distribution**. A model trained on photos of cats should generate plausible new cat photos. A model trained on human writing should generate plausible human-sounding text.

Three major families of generative architectures have emerged: **VAEs**, **GANs**, and **Diffusion models**. Each approaches the generation task differently.

---

## Variational Autoencoders (VAEs)

A **VAE** compresses input data into a compact **latent space** (a lower-dimensional representation), then reconstructs it from that compressed form.

- **Encoder** — maps an input (e.g. an image) to a point in latent space (a distribution, not just a point)
- **Decoder** — maps a point in latent space back to an output

The trick: by training the encoder to produce a *distribution* rather than a single point, and sampling from that distribution during training, the latent space becomes smooth and continuous. You can then sample a random point in latent space and decode it into a plausible output.

VAEs produce somewhat blurry outputs (optimising for average reconstruction rather than sharpness). They're less used for high-quality image generation but are important conceptually and still used for tasks where a structured latent space is useful (e.g. molecular design, style transfer).

---

## GANs: Generative Adversarial Networks

Introduced by Goodfellow et al. in 2014, **GANs** frame generation as a game between two networks:

- **Generator** — takes random noise as input, tries to generate realistic-looking samples
- **Discriminator** — tries to distinguish real training samples from generated fakes

The two networks train in competition. As the discriminator gets better at catching fakes, the generator is forced to get better at making convincing ones. At convergence, the generator produces samples indistinguishable from real data.

GANs produce sharp, high-quality outputs — they dominated image synthesis from ~2014–2020 (StyleGAN, BigGAN, etc.). Their weaknesses: training instability (the game can collapse), **mode collapse** (the generator learns to produce only a few types of output that fool the discriminator), and no direct way to control what gets generated.

---

## Diffusion Models

**Diffusion models** are now the dominant architecture for image generation (Stable Diffusion, DALL-E 3, Midjourney, Imagen).

The core idea is deceptively simple:

1. **Forward process (training)**: gradually add Gaussian noise to a training image over many steps until it becomes pure noise. This process is mathematically well-defined.
2. **Reverse process (generation)**: train a neural network to *reverse* this process — to predict the noise at each step and denoise slightly. At inference, start from pure noise and iteratively denoise until a clean image emerges.

The neural network at the core is typically a **U-Net** (for pixel-space models) or a Transformer (for newer latent-space models). **Latent diffusion models** (like Stable Diffusion) run the diffusion process in a compressed latent space rather than pixel space, which dramatically reduces computation.

**Text conditioning** — text-to-image models use a text encoder (often a CLIP or language model) to produce a representation of the prompt, which guides the denoising process via cross-attention.

Diffusion models produce high-quality, diverse outputs and are easier to train stably than GANs. Their main drawback: slow inference — generating one image requires many denoising steps.

---

## How these connect to LLMs

LLMs are autoregressive generative models — they generate text token by token, each conditioned on previous tokens. They don't use diffusion or adversarial training for the generation process itself (though RLHF uses a reward model which has connections to discriminator training).

The VAE concept of a latent space reappears in **multimodal models** that encode images, text, and audio into a shared embedding space.

Understanding generative architectures broadly helps when working with multimodal LLMs, image generation APIs, and systems that combine text and visual generation — all covered in Phase 4 / 06.

---

## What this means when you're building

**Diffusion API parameters decoded:**
- `steps` (or `num_inference_steps`) — how many denoising steps to run. More steps = better quality, slower generation. Typical range: 20–50. This is the primary latency dial.
- `guidance_scale` (classifier-free guidance) — how strongly the model follows your text prompt vs. generating freely. Higher values = closer to your prompt but less diverse. Too high = oversaturated, distorted. Typical: 7–12.
- `negative_prompt` — tokens to steer *away* from. Works because the model can condition on "what not to generate" via the same cross-attention mechanism used for positive prompting.

**Latent diffusion = why Stable Diffusion runs locally**: running diffusion in pixel space is expensive — the U-Net must process the full resolution image at each denoising step. Latent diffusion (the Stable Diffusion approach) runs the diffusion process in a compressed latent space (~8× smaller), then decodes the result with a VAE decoder. This is why Stable Diffusion can run on a consumer GPU and why images are decoded, not generated pixel by pixel.

**CLIP as the bridge**: text-to-image generation works because CLIP maps text and images into the same embedding space. Your text prompt is encoded by the CLIP text encoder into a vector; that vector guides the denoising process via cross-attention. Understanding CLIP explains why prompt quality matters — it's not magic, it's a semantic embedding.

---

## Summary

| Architecture | Mechanism | Strengths | Weaknesses |
|-------------|-----------|-----------|------------|
| VAE | Encode to latent distribution; decode | Smooth latent space; controllable | Blurry outputs |
| GAN | Generator vs discriminator adversarial game | Sharp, high-quality outputs | Training instability; mode collapse |
| Diffusion | Iterative denoising from noise | High quality; stable training; diverse | Slow inference |
| LLM | Autoregressive token prediction | Text (and more); scalable | Hallucination; context limits |

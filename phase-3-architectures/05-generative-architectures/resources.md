# Resources: Generative Architectures

---

## Diffusion models (the dominant current paradigm)

**"What are Diffusion Models?" — Lilian Weng, Lil'Log**
https://lilianweng.github.io/posts/2021-07-11-diffusion-models/
The most thorough written explanation of diffusion models available for practitioners. Covers DDPM, score matching, DDIM, and conditional generation with clear derivations. Lil'Log is consistently one of the best ML technical blogs — accurate, well-structured, and regularly updated. Start here for diffusion.

**"Diffusion Models: A Practical Guide" — Hugging Face Blog**
https://huggingface.co/blog/annotated-diffusion
An annotated implementation of a diffusion model in PyTorch, walking through the DDPM paper step by step. Similar in style to the Annotated Transformer. For those who want to go from concept to code.

**"High-Resolution Image Synthesis with Latent Diffusion Models" — Rombach et al. (2021)**
https://arxiv.org/abs/2112.10752
The paper behind Stable Diffusion. Introduces the latent diffusion concept. Abstract and Figure 1 are accessible; the full paper is technical. Worth knowing exists.

---

## GANs

**"Generative Adversarial Networks" — Goodfellow et al. (2014)**
https://arxiv.org/abs/1406.2661
The original GAN paper. Short (9 pages), readable introduction, and the min-max game formulation is clearly stated. A historically important paper worth reading the introduction of.

**"GAN — What is Generative Adversarial Network?" — Towards Data Science**
https://towardsdatascience.com/understanding-generative-adversarial-networks-gans-cd6e4651a29
A clear introductory walkthrough of GANs with diagrams — covers the generator/discriminator dynamic, training challenges, and mode collapse. Good first read before the original paper.

**"Progressive Growing of GANs" — NVIDIA (2018)**
https://arxiv.org/abs/1710.10196
Introduced the technique of training GANs at progressively higher resolutions, leading to the first hyper-realistic face generation. The paper is readable; the generated images made headlines.

---

## VAEs

**"Tutorial on Variational Autoencoders" — Carl Doersch (2016)**
https://arxiv.org/abs/1606.05908
The clearest written introduction to VAEs — explains the latent space, the ELBO objective, and reparameterisation with minimal assumed background. 23 pages; Sections 1–3 cover the core concept.

**"VAE Explained" — by Alfredo Canziani (video)**
Search YouTube for "Alfredo Canziani VAE" — NYU Deep Learning course lectures are well-recorded and clear. A good visual complement to the Doersch paper.

---

## Multimodal connection

**"CLIP: Learning Transferable Visual Models From Natural Language Supervision" — Radford et al., OpenAI (2021)**
https://arxiv.org/abs/2103.00020
Introduced CLIP, the joint image-text embedding model used as the text encoder in most text-to-image systems (including Stable Diffusion and DALL-E). The abstract and introduction explain the core idea clearly. Directly relevant to Phase 4 / Multimodality.

**"Hierarchical Text-Conditional Image Generation with CLIP Latents" — Ramesh et al. (DALL-E 2, 2022)**
https://arxiv.org/abs/2204.06125
The DALL-E 2 paper, which uses CLIP embeddings to condition image generation. Shows how LLM-style text understanding is combined with diffusion for text-to-image generation.

---

## For broader context

**Distill.pub — Generative Models**
https://distill.pub/
Several Distill articles cover generative models with interactive visualisations. Search for "GAN", "VAE", or "diffusion" — the quality bar is exceptionally high. Particularly: "Feature Visualization" and "Activation Atlas" for understanding what generative models have learned.

---

## Going deeper

**"Denoising Diffusion Probabilistic Models" — Ho et al. (DDPM, 2020)**
https://arxiv.org/abs/2006.11239
The paper that established the modern diffusion model formulation. DDPM is the foundation that Stable Diffusion, DALL-E 3, and Imagen are built on. Introduction and Figure 2 are accessible; the math in sections 3–4 is dense but well-motivated.

**"Classifier-Free Diffusion Guidance" — Ho & Salimans (2022)**
https://arxiv.org/abs/2207.12598
Short paper (4 pages) that introduced classifier-free guidance — the mechanism behind the `guidance_scale` parameter in every image generation API. Explains why and how text prompts guide the denoising process. Very practical for understanding text-to-image generation behaviour.

**"Stable Diffusion — How It Works" — Jay Alammar**
https://jalammar.github.io/illustrated-stable-diffusion/
Jay Alammar's illustrated walkthrough of the full Stable Diffusion pipeline: CLIP text encoder → latent diffusion → VAE decoder. Ties together every concept from this module in the context of a real production system. Recommended reading before Phase 4 / 06 (Multimodality).

**"Generative Adversarial Networks — NIPS 2016 Tutorial" — Ian Goodfellow**
https://arxiv.org/abs/1701.00160
Goodfellow's own tutorial on GANs, written for a broad ML audience. Covers the theory, training challenges, and evaluation metrics clearly. More thorough than the original 2014 paper — this is the better starting point for understanding GANs in depth.

**"Score-Based Generative Modeling through Stochastic Differential Equations" — Song et al. (2020)**
https://arxiv.org/abs/2011.13456
The unified SDE framework for diffusion models — the theoretical foundation that explains DDPM, DDIM, and score matching as special cases of the same process. Heavy on mathematics, but the introduction and conceptual sections are accessible. For those who want the full picture.

**Hugging Face Diffusers library documentation**
https://huggingface.co/docs/diffusers/
The practical entry point for running diffusion models in code. The conceptual guides cover pipelines, schedulers, and conditioning clearly. Useful reference if you're working with image generation APIs or self-hosted diffusion models.

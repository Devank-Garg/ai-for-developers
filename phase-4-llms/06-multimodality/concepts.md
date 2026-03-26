# Concepts: Multimodality

> **Tag: core** — Modern frontier models process text, images, audio, and video in a single system. Understanding how multimodality works — and what its current limits are — is essential for building applications that use it effectively.

---

## What multimodality means

A **unimodal** model processes one type of input — text in, text out. A **multimodal** model processes multiple types — text, images, audio, video — and can produce output across multiple types.

The practical significance: you can ask a model to describe an image, answer questions about a chart, transcribe audio, or read a screenshot — without piping data through separate specialised systems.

**Current state (2025):**

| Model | Text | Images | Audio | Video |
|---|---|---|---|---|
| Claude Sonnet/Opus 4.x | ✓ | ✓ | — | — |
| GPT-4o | ✓ | ✓ | ✓ | ✓ |
| Gemini 2.5 Pro | ✓ | ✓ | ✓ | ✓ |
| Llama 3.2 Vision | ✓ | ✓ | — | — |
| Qwen3-VL | ✓ | ✓ | — | — |
| Whisper (OpenAI) | — | — | ✓ → text | — |

"Native multimodal" models like GPT-4o and Gemini 2.5 are trained end-to-end across modalities from the start. Other models bolt on vision after text pre-training — functionally similar for most tasks but architecturally different.

---

## Vision encoders — how images become tokens

Language models operate on tokens. Images are not tokens. To process an image, it must be converted into a representation the transformer can attend to.

**Step 1 — Patch extraction (Vision Transformer / ViT):**

An image is divided into a grid of fixed-size patches (typically 14×14 or 16×16 pixels). Each patch is treated as a single unit, similar to a word in text.

```
224×224 pixel image ÷ 14×14 patches = 256 patches
Each patch → flattened pixel values → linear projection → patch embedding vector
```

**Step 2 — Visual encoding:**

A **vision encoder** (typically a ViT, often pre-trained with CLIP) processes the patch embeddings through transformer layers, producing a sequence of visual feature vectors. These capture both local features (edges, textures) and global semantic content (objects, scenes) through self-attention.

**Step 3 — Projection to language space:**

The visual feature vectors are in "vision space" — a different representation than the LLM's "text space." A **projection layer** (a small MLP or cross-attention module) maps visual features into the same embedding space the LLM uses for text tokens.

```
Patches → [Vision Encoder] → visual features → [Projection Layer] → visual tokens
Text    → [Tokenizer]       → text tokens     →                     text tokens

Combined: [visual_token_1, visual_token_2, ..., visual_token_256, text_token_1, ...]
          ↓
          [Language Model] → text response
```

The LLM then sees a sequence of visual tokens followed by text tokens — and processes them all with the same attention mechanism. From the LLM's perspective, image patches are just tokens in a longer sequence.

---

## CLIP — Contrastive Language-Image Pre-training

**CLIP** (OpenAI, 2021) is the foundational vision encoder model underlying most modern vision-language systems.

**How CLIP was trained:** On 400 million (image, text caption) pairs from the internet. The training objective: make the embedding of an image and the embedding of its correct caption as similar as possible, while making incorrect (image, caption) pairs dissimilar. This is **contrastive learning** — the loss pushes matching pairs together and non-matching pairs apart.

```
Image of a cat + "a photo of a cat" → high similarity
Image of a cat + "a photo of a dog" → low similarity
```

**Why CLIP matters for LLMs:**
- CLIP's image encoder learns representations that are semantically aligned with language — the same semantic content (a cat) produces nearby embeddings whether expressed as an image or as text
- This makes CLIP's visual features directly useful for language models — the semantic alignment is already built in
- Most open-source VLMs (LLaVA, Qwen-VL, InternVL) use CLIP or a CLIP-variant as their visual backbone

---

## Image token count and context window impact

Images consume a significant portion of the context window. A typical 224×224 image produces 256 visual tokens. Larger or higher-resolution images produce more.

**Dynamic resolution (2025 standard):** Modern VLMs no longer resize all images to a fixed 224×224. They tile high-resolution images into multiple sub-images and encode each tile separately, then combine results. A 1024×1024 image might be split into 16 tiles of 256×256 — each producing ~256 tokens = **4,096 visual tokens total**.

```
Image token budget by resolution:
  Low res (224×224):        ~256 tokens
  Medium (512×512):         ~1,024 tokens (via tiling)
  High res (1024×1024):     ~4,096 tokens (via tiling)
  Very high (2048×2048):    ~16,384 tokens — significant context consumption
```

> **Production note:** High-resolution image processing is expensive. If you're sending many images to a multimodal API, check the token cost per image. A single 2048×2048 screenshot at a context window price of $3/M tokens costs ~$0.05 per image — across 10,000 images/day, that's $500/day from images alone. Downscale when resolution isn't needed for the task.

---

## Audio modality

**Audio-to-text (ASR):** The most mature audio application. Models like OpenAI's Whisper transcribe speech to text with high accuracy across many languages. Whisper is open-source and widely deployed self-hosted.

**Native audio understanding (GPT-4o, Gemini 2.5):** Frontier multimodal models can process audio directly — understanding tone, emotion, speaker identity, and content — without first transcribing to text. GPT-4o's "voice mode" processes audio end-to-end, enabling natural conversation with realistic latency.

**How audio is encoded for transformers:** Audio is converted to a **mel spectrogram** — a 2D representation of frequency content over time — and then processed similarly to images (split into patches, encoded by a vision-style encoder). This is the same approach Whisper uses.

**Text-to-speech (TTS):** Separate from LLMs; covered by models like ElevenLabs, OpenAI TTS, and open alternatives like Kokoro. Typically the LLM produces text, and a separate TTS model converts it to audio.

---

## Video modality

**Video as a sequence of frames:** The simplest video processing approach treats video as a sequence of sampled frames, encodes each frame as images, and processes the resulting token sequence. At 1 frame per second, a 60-second video = 60 images = 15,000+ tokens. Context window size becomes the limiting factor.

**Video-native models:** Gemini 2.5 Pro can process up to 1 hour of video. It achieves this by combining temporal subsampling (not every frame), efficient video encoders trained to capture motion and temporal patterns, and a very large context window (1M+ tokens).

**Current limitations:**
- Processing long videos is expensive (many tokens)
- Temporal reasoning (what happened before/after X?) is harder than spatial reasoning
- Most open-source models have limited video capability; frontier-only territory for serious video understanding

---

## Current frontier multimodal models

**GPT-4o (OpenAI):** Natively multimodal — trained on text, image, and audio from the start. Supports image input, image generation (DALL-E integration), real-time audio conversation. As of 2025, remains a benchmark leader on vision tasks.

**Gemini 2.5 Pro (Google):** The most capable multimodal model for long-context tasks. 1M token context window (2M in some configurations). Handles very long videos, large documents with embedded images, and complex cross-modal reasoning. Strongest open benchmark scores on multimodal academic tasks as of early 2026.

**Claude Sonnet/Opus 4.x (Anthropic):** Strong image understanding — document analysis, chart reading, screenshot interpretation, detailed image description. Does not currently support audio or video input. The "computer use" capability allows Claude to interpret and interact with screenshots of computer interfaces.

**Llama 3.2 Vision (Meta):** Open-weight vision-language model (11B and 90B sizes). Competitive with proprietary models on standard vision benchmarks. Deployable self-hosted with vLLM or Ollama. Standard choice for open-source vision applications.

**Qwen3-VL (Alibaba):** The Qwen3-VL-235B-A22B model (released 2025) rivals Gemini 2.5 Pro and GPT-5 on multimodal benchmarks. The 72B variant is practical to self-host. Particularly strong on OCR, document understanding, and multilingual visual content.

---

## What multimodal models are good at

**Document and chart analysis:** Reading PDFs, financial reports, slides, and scientific papers with embedded figures. Models can answer questions about charts, extract tables, and synthesise text + visual information that would require manual effort.

**Screenshot and UI understanding:** Interpreting screenshots of interfaces, web pages, and applications. Claude's "computer use" and GPT-4o's vision capabilities enable automation workflows that previously required custom CV pipelines.

**OCR (Optical Character Recognition):** Extracting text from images, scans, and photos. Frontier multimodal models are now competitive with dedicated OCR systems — and can understand the semantic context of the text, not just extract characters.

**Visual question answering:** Answering questions about image content. "What is the defect visible in the left panel?" "How many red items are in this image?" "What does the error message say?"

**Medical and scientific imaging:** Preliminary analysis of X-rays, histology slides, and scientific visualisations. Significant ongoing research; not a replacement for clinical expertise but useful as a first-pass tool.

---

## What multimodal models struggle with

**Precise spatial reasoning:** "How many pixels from the left edge is the button?" or "What is the exact angle of this line?" — models are better at semantic content than precise spatial measurement.

**Fine-grained counting:** Counting large numbers of small objects in an image reliably. Models often estimate or make errors above ~10–15 objects.

**Video temporal reasoning:** Understanding cause-and-effect relationships across long video timelines. "What caused the machine to stop at 3:42?" is harder than "What is shown in this frame?"

**Hallucination about visual content:** Models can confidently describe image content that isn't there — especially text in images (misreading characters), small details, or content in peripheral areas of the image.

**Consistency across multiple images:** Maintaining consistent understanding across many images in a long context (e.g., "Compare image 3 and image 12") degrades with distance and count.

---

## Multimodal fine-tuning

Vision-language models can be fine-tuned for specific visual tasks using the same PEFT techniques as text models:

- **LoRA on the LLM component** with image input — the vision encoder is typically frozen
- **Visual instruction tuning** — training on (image, instruction, response) triples for domain-specific tasks
- **Tools:** Unsloth, LLaMA-Factory, and TRL all support multimodal fine-tuning as of 2025

Common use cases: adapting a general VLM for medical imaging, industrial inspection, document-specific formats, or domain-specific visual Q&A.

---

## Building with multimodal APIs

**Sending images to an API:**

```python
import anthropic, base64

client = anthropic.Anthropic()

with open("chart.png", "rb") as f:
    image_data = base64.standard_b64encode(f.read()).decode("utf-8")

response = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=1024,
    messages=[{
        "role": "user",
        "content": [
            {
                "type": "image",
                "source": {
                    "type": "base64",
                    "media_type": "image/png",
                    "data": image_data,
                },
            },
            {
                "type": "text",
                "text": "What trend does this chart show? Summarise in 2 sentences."
            }
        ],
    }]
)
```

**Image source types:** APIs accept images as base64-encoded data, URLs (some providers), or file IDs (for uploaded files). Base64 is universal; URL-based access depends on the provider and may require public URLs.

---

## Summary

| Concept | Key point |
|---|---|
| Multimodality | Processing multiple input types (text, image, audio, video) in a single model |
| Vision encoder (ViT) | Splits image into patches → encodes as feature vectors; treats patches like tokens |
| CLIP | Contrastively trained on image-text pairs; produces semantically aligned visual features; foundation of most open VLMs |
| Projection layer | Maps visual features from "vision space" to "text space" so LLM can attend to them |
| Image token cost | 256–16,000+ tokens per image depending on resolution; tiling enables high-res at context cost |
| Dynamic resolution | 2025 standard — tile images into patches for high-res rather than downscaling everything |
| Audio (native) | GPT-4o, Gemini process audio end-to-end; mel spectrogram → encoder → tokens |
| Video | Sequence of sampled frames + temporal encoders; expensive; best support in Gemini 2.5 |
| Frontier VLMs | GPT-4o, Gemini 2.5 Pro (best overall); Claude 4.x (documents, screenshots); Qwen3-VL (open benchmark rival) |
| Open-weight VLMs | Llama 3.2 Vision (90B), Qwen3-VL (72B) — self-hostable, competitive on benchmarks |
| Strengths | Document analysis, OCR, chart reading, screenshot understanding, visual Q&A |
| Limitations | Precise spatial reasoning, counting, temporal video reasoning, hallucination about fine details |

# LLM Fundamentals

> **Tag: core** — This is the foundation for everything in Phase 4 and Phase 5. Do not skip it.

---

## What this module covers

Large language models are the subject of the rest of this course. Before you build anything with them, you need a clear mental model of what they are, how they work, and — critically — what they are not.

This module builds that model from the ground up, from the perspective of a developer who already understands functions, APIs, and data.

---

## Learning objectives

By the end of this module you should be able to:

- Explain what an LLM is as a function: inputs, outputs, and what determines its behaviour
- Describe how autoregressive generation works, step by step
- Explain what tokens are and why they matter for cost and behaviour
- Describe the three-stage training pipeline: pre-training, instruction tuning, and alignment
- Explain what a context window is and what its limits mean in practice
- List what LLMs do not have by default (memory, time awareness, tools, access to the internet)
- Explain what temperature and sampling are and how they affect output
- Describe emergent capabilities and why scale produces qualitatively new behaviour
- Know when to use a hosted API vs other options

---

## Sections in `concepts.md`

1. What an LLM actually is
2. Next-token prediction
3. Autoregressive generation
4. Tokens and tokenisation
5. The training pipeline: pre-training → instruction tuning → alignment
6. Context window
7. Temperature and sampling
8. What LLMs don't have
9. Emergent capabilities
10. The model landscape

---

## Prerequisites

- Phase 3: Transformers (`04-transformers`) — LLMs are decoder-only Transformers; this module assumes you know what attention, layers, and autoregressive decoding are at a conceptual level.

---

## Estimated time

2–3 hours for concepts + quiz. Longer if you explore the linked resources.

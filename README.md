# 🤖 AI for Developers — Zero to Production

> A free, open-source course for software developers and IT professionals who want to **build with AI** — not become AI researchers.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](CONTRIBUTING.md)
[![Status: In Progress](https://img.shields.io/badge/Status-In%20Progress-blue.svg)]()
[![Audience: Developers](https://img.shields.io/badge/Audience-Software%20Developers-purple.svg)]()

---

## 📌 What This Is

This course is built for **software developers, IT leads, and technical professionals** who:

- Know how to write code but are new to AI
- Want to build real AI-powered applications — chatbots, RAG systems, agents
- Don't want to become an ML researcher, but need solid conceptual foundations
- Are tired of "Hello World" AI tutorials that don't prepare them for production

**It is not** a math course. It is not a framework tutorial. Every concept is taught provider-agnostically — the principles work whether you're using Claude, GPT-4, Gemini, Llama, or a model that doesn't exist yet.

---

## 🗺️ Course Roadmap

The course is structured in **6 phases**, progressing from mental models to production systems.

| Phase | Title | Duration | Focus |
|-------|-------|----------|-------|
| 1 | AI Concepts & Orientation | 2–3 weeks | Terminology, data fundamentals |
| 2 | How Models Learn | 2–3 weeks | Training intuition, loss functions |
| 3 | Neural Network Architectures | 2–4 weeks | CNNs, RNNs, Transformers (conceptual) |
| **4** | **LLMs & Generative AI** | **6–8 weeks** | **Core focus — the main module** |
| 5 | Building Real AI Applications | 5–7 weeks | APIs, chatbots, RAG, memory, agents |
| 6 | Responsible AI & The Bigger Picture | 2–3 weeks | Safety, risks, scaling, regulation |

> **Phase 4 is the heart of this course.** It covers LLM fundamentals, training at scale, fine-tuning, inference, prompting, and multimodality in depth.

---

## 📂 Repository Structure

```
ai-for-developers/
│
├── README.md                          ← You are here
├── CONTRIBUTING.md                    ← How to contribute
├── LICENSE                            ← MIT License
├── TRACKER.xlsx                       ← Course content build tracker
│
├── phase-1-concepts/
│   ├── 01-terminology/
│   ├── 02-data-fundamentals/
│   └── 03-classical-ml-reference/
│
├── phase-2-how-models-learn/
│   ├── 01-loss-functions/
│   ├── 02-training-loop/
│   └── 03-optimization-reference/
│
├── phase-3-architectures/
│   ├── 01-neural-network-basics/
│   ├── 02-cnns/                       ← optional
│   ├── 03-rnns-lstms/                 ← optional
│   ├── 04-transformers/               ← core
│   └── 05-generative-architectures/   ← optional
│
├── phase-4-llms/                      ← main focus
│   ├── 01-llm-fundamentals/
│   ├── 02-training-at-scale/
│   ├── 03-efficient-training/
│   ├── 04-inference-deployment/
│   ├── 05-prompting/
│   └── 06-multimodality/
│
├── phase-5-building/
│   ├── 01-api-calls/
│   ├── 02-chatbots/
│   ├── 03-rag/
│   ├── 04-memory/
│   └── 05-agents/
│
├── phase-6-responsible-ai/
│   ├── 01-safety-alignment/
│   ├── 02-responsible-ai-practice/
│   ├── 03-scaling/
│   └── 04-risks-society/
│
└── assets/
    ├── images/
    └── diagrams/
```

Each module folder follows a consistent structure:

```
01-llm-fundamentals/
├── README.md          ← Module overview and learning objectives
├── concepts.md        ← Core theory and explanations
├── examples/          ← Worked examples and illustrations
├── code/              ← Code samples (where applicable)
├── quiz.md            ← Self-check questions
└── resources.md       ← Curated external reading list
```

---

## 🏷️ Content Tags

Every module and topic is tagged to help you navigate:

| Tag | Meaning |
|-----|---------|
| `core` | Essential — everyone should complete this |
| `optional` | Go deeper if you're curious or working in this domain |
| `reference` | Links and overviews only — no deep content required |
| `theory only` | Conceptual understanding, no code |
| `theory + code` | Concept explained, then implemented |
| `theory + examples` | Concept explained with worked examples |

---

## 🚀 Getting Started

### If you're completely new to AI

Start at Phase 1. Don't skip it — the vocabulary built there is used in every subsequent phase.

```
Phase 1 → Phase 2 → Phase 3 (Transformers only) → Phase 4 → Phase 5
```

### If you're a developer who's used ChatGPT but never built with LLM APIs

You can skip Phase 1's terminology module and start at **Phase 1 → Data Fundamentals**, then move straight to Phase 4.

```
Phase 1 (data only) → Phase 4 → Phase 5 → Phase 6
```

### If you want to ship something as fast as possible

Go directly to Phase 4 → Phase 5. Come back and fill gaps as you encounter them.

```
Phase 4 (LLM fundamentals + prompting) → Phase 5 → fill gaps retroactively
```

---

## 💡 Design Principles

This course was built with a specific set of beliefs:

**1. Concepts over syntax**
Every principle is taught in a way that works regardless of which model, framework, or cloud provider you use. Understanding *why* RAG works means you can implement it in any stack.

**2. Developers, not researchers**
No unnecessary math. No derivations. Where math appears, it's because the intuition genuinely requires it — and it's explained visually first.

**3. Production-minded from the start**
We don't just teach you how to make a model respond. We teach you how to handle errors, manage costs, think about latency, and build systems that won't embarrass you in front of users.

**4. Honest about limitations**
Hallucination isn't a bug that gets fixed in the next version. Context windows have real constraints. Agents are unreliable in predictable ways. This course doesn't sell a fantasy.

**5. Provider-agnostic**
Code examples use a simple adapter pattern. Swap Claude for GPT-4 or Llama by changing one line. The concepts work the same everywhere.

---

## 🤝 Contributing

This is a community course. Contributions are welcome and encouraged.

You can contribute by:

- **Writing content** for an unfinished module (check `TRACKER.xlsx` for status)
- **Improving explanations** — clearer is always better
- **Adding code examples** in the `code/` folder of any module
- **Fixing errors** — factual, conceptual, or typographical
- **Translating** modules into other languages
- **Adding resources** to `resources.md` files

Please read [CONTRIBUTING.md](CONTRIBUTING.md) before opening a PR.

### Content quality bar

- Explanations must work without running any code
- Code examples must run without modification (pin dependency versions)
- No framework-specific tutorials — the `code/` folder shows patterns, not library usage
- All claims must be accurate as of the content's last-reviewed date

---

## 📋 Build Tracker

`TRACKER.xlsx` contains the full content plan with status for every topic across all phases. If you're a contributor looking for what to work on, open the tracker and filter by `Status = Not Started` and `Priority = High`.

Columns tracked: Level · Tag · Content Type · Status · Priority · Est. Hours · Actual Hours · % Done · Assigned To · Target Date · Notes

---

## 📚 Recommended Pre-Reading

Before starting Phase 1, these free resources provide useful context:

- [Andrej Karpathy — Intro to LLMs](https://www.youtube.com/watch?v=zjkBMFhNj_g) (1hr YouTube talk — the best single introduction that exists)
- [3Blue1Brown — Neural Networks series](https://www.3blue1brown.com/topics/neural-networks) (visual, no math background needed)
- [Simon Willison — Things I've learned about LLMs](https://simonwillison.net/2023/Aug/3/weird-world-of-llms/) (practitioner perspective)

---

## 📄 License

This course is released under the [MIT License](LICENSE). You are free to use, modify, and distribute this content — including for commercial purposes — with attribution.

If you use this material to run a course or training, a link back to this repository is appreciated but not required.

---

## ✨ Acknowledgements

Built with input from software developers, IT leads, and AI practitioners who were tired of courses that assumed you either knew nothing or already had a PhD.

Special thanks to everyone who has filed issues, submitted PRs, and helped make these explanations clearer.

---

<div align="center">

**If this course helped you, star the repository ⭐ — it helps others find it.**

[Start Learning](phase-1-concepts/) · [View Tracker](TRACKER.xlsx) · [Contribute](CONTRIBUTING.md)

</div>
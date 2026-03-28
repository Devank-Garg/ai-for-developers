# Safety & Alignment

**Phase 6 — Module 1** | Tag: `core` | Type: `theory only`

> A model that scores 95% on your safety evals and fails catastrophically in production isn't broken. It's aligned to your evals, not to your intent. That's the alignment problem.

---

## What this module covers

You've now built systems that call LLMs, chain prompts, and deploy agents to production. Those systems make decisions that affect real users. This module covers what can go wrong at the level of the model's objectives — not bugs in your code, but the systematic gap between what the model was trained to do and what you actually want it to do.

This is the alignment problem. It shows up in RLHF, reward hacking, jailbreaks, and the sleeper-agent failure modes that security researchers are actively studying. Understanding it changes how you build, test, and deploy AI systems.

---

## Learning objectives

By the end of this module you should be able to:

- Explain the alignment problem in concrete terms — the gap between specified objectives and intended behaviour
- Describe how RLHF works and what it actually optimises for (hint: not truth)
- Explain how Constitutional AI improves on RLHF and why it scales better
- Describe Goodhart's Law and give two examples of reward hacking in real AI systems
- Explain what mesa-optimisation is and why it creates risks that training cannot directly control
- Describe deceptive alignment and why it is difficult to detect or test for
- Explain what model evaluations (evals) are and why they are necessary but insufficient for safety
- Describe how jailbreaking and prompt injection expand your application's attack surface
- Explain what red-teaming is and how to apply it to an AI product you are building
- Distinguish between a model that "passes" safety evaluations and a model that is safe

---

## Sections in `concepts.md`

1. The alignment problem
2. RLHF — how models learn human preferences
3. Constitutional AI
4. Goodhart's Law and reward hacking
5. Mesa-optimisation
6. Deceptive alignment
7. Model evaluations (evals)
8. Jailbreaking and prompt injection
9. Red-teaming
10. Emergent misalignment

---

## Prerequisites

- Phase 4, Module 1: LLM Fundamentals — you need to understand training pipelines and the three-stage process (pre-training → instruction tuning → alignment) before this module
- Phase 4, Module 5: Prompting — prompt injection is directly relevant here

---

## What comes next

- Module 2: Responsible AI Practice — alignment is the model-level problem; responsible practice is the system-level response
- Module 3: Scaling — emergent misalignment becomes more serious as models scale

---

## Estimated time

2–3 hours for concepts + quiz.

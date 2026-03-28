# Agents

**Phase 5 — Module 5** | Tag: `core` | Type: `theory + examples`

> An agent doesn't just answer — it acts. That means every decision can have consequences. This module covers how to build agents that are useful, bounded, and safe enough to deploy.

---

## What this module covers

All the prior modules build to this one. An agent is a chatbot (Module 2) that calls tools (defined here), maintains state across a loop (observe → think → act), and can use RAG (Module 3) and memory (Module 4) as tools. What's new in this module is everything that makes that loop production-worthy: defining tools clearly, validating tool selection, bounding the execution loop, recovering from mid-chain failures, adding human approval checkpoints, and understanding the failure modes that are unique to agentic systems.

---

## Learning objectives

By the end of this module you should be able to:

- Write tool definitions with descriptions that reliably guide model selection
- Implement an execution loop with a valid termination condition and iteration limit
- Return structured error data from tools so the model can reason about failures
- Add a human-in-the-loop approval checkpoint for irreversible actions
- Detect and interrupt infinite loops and hallucinated tool calls
- Describe the four main agent failure modes and their mitigations

---

## Module files

| File | What it is |
|------|-----------|
| [`concepts.md`](concepts.md) | Component reference — one card per topic. Start here. |
| [`examples/basic.md`](examples/basic.md) | Pseudocode walkthrough: a research agent that searches and summarizes, one loop cycle |
| [`examples/production.md`](examples/production.md) | Same agent hardened — loop guard, hallucination interception, error recovery, human checkpoint |
| [`quiz.md`](quiz.md) | 7 scenario-based questions. Apply the concepts, don't recite them. |
| [`resources.md`](resources.md) | Papers, reference repos, security blogs, and videos for going deeper |

---

## Prerequisites

- **Module 1: API Calls** — [error handling and retries](../01-api-calls/concepts.md#error-handling-and-retries) and [token budgeting](../01-api-calls/concepts.md#token-budgeting). The agent loop makes many API calls; error handling must account for failures at any step.
- **Module 2: Chatbots** — [conversation history management](../02-chatbots/concepts.md#conversation-history-management) and [turn structure](../02-chatbots/concepts.md#turn-structure). The agent loop extends the messages array with tool calls and results.
- **Module 3: RAG** and **Module 4: Memory** — helpful context, as agents often use both as tools. Not strictly required for the execution loop itself.

---

## What comes next

- **Phase 6 — Responsible AI** — agent safety, alignment, and the risks of autonomous systems. The failure modes in this module are the practical entry point to those bigger questions.

---

## Estimated time

2.5–3 hours: 70 min for concepts, 50 min for examples, 30 min for quiz

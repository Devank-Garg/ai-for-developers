# Chatbots

**Phase 5 — Module 2** | Tag: `core` | Type: `theory + examples`

> A chatbot is a messages array that grows across turns. The hard parts are not the API call — they're deciding what to put in the array, how long to keep it, and what happens when the session ends.

---

## What this module covers

You already know how to make a single API call (Module 1). A chatbot is what happens when you chain those calls together while preserving context. This module covers the six components that separate a working chatbot from a brittle demo: managing the growing message history, designing the system prompt, maintaining turn structure, handling context window pressure, controlling response format, and managing the session lifecycle from start to end.

---

## Learning objectives

By the end of this module you should be able to:

- Build and persist a conversation history that survives server restarts
- Write a system prompt that controls persona, scope, and response style
- Maintain valid turn alternation and handle edge cases (buffered input, injected context)
- Detect when history is approaching the context limit and apply a trimming strategy
- Enforce response format through system prompt instructions and schema validation
- Implement a session lifecycle with start, continue, end, and timeout events

---

## Module files

| File | What it is |
|------|-----------|
| [`concepts.md`](concepts.md) | Component reference — one card per topic. Start here. |
| [`examples/basic.md`](examples/basic.md) | Pseudocode walkthrough: a multi-turn support chatbot, step by step |
| [`examples/production.md`](examples/production.md) | Same chatbot hardened — context trimming, session persistence, format enforcement |
| [`quiz.md`](quiz.md) | 6 scenario-based questions. Apply the concepts, don't recite them. |
| [`resources.md`](resources.md) | Official guides, engineering blogs, and videos for going deeper |

---

## Prerequisites

- **Module 1: API Calls** — specifically [request anatomy](../01-api-calls/concepts.md#request-anatomy) and [token budgeting](../01-api-calls/concepts.md#token-budgeting). Chatbot history management is an extension of the messages array; context window pressure is token budgeting applied across multiple turns.

---

## What comes next

- **Module 3: RAG** — adds a retrieval step before the chat call, injecting retrieved context into the messages array using the same turn structure from this module.
- **Module 4: Memory** — extends conversation history with external storage; short-term memory is the chatbot history from this module, just persisted differently.
- **Module 5: Agents** — runs a chatbot loop where some "turns" are tool calls instead of user messages.

---

## Estimated time

1.5–2 hours: 50 min for concepts, 30 min for examples, 25 min for quiz

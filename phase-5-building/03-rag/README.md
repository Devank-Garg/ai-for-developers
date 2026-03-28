# RAG (Retrieval-Augmented Generation)

**Phase 5 — Module 3** | Tag: `core` | Type: `theory + examples`

> RAG is the pattern for grounding a model's answers in your data — not its training data. Without it, your model knows everything generally and nothing specifically.

---

## What this module covers

LLMs are trained on public data up to a cutoff date. They know nothing about your internal documentation, your product's behavior, or events after their training. RAG solves this by retrieving relevant passages from your own documents at query time and injecting them into the prompt. This module covers the full pipeline: chunking documents, embedding and indexing them, retrieving on a query, injecting context, grounding the answer, and handling the cases where retrieval fails.

---

## Learning objectives

By the end of this module you should be able to:

- Split a document corpus into appropriately sized chunks with overlap
- Build a vector index at document-load time and query it at request time
- Evaluate and improve retrieval quality using query rewriting and re-ranking
- Inject retrieved context into the messages array within a token budget
- Enforce answer grounding so the model uses your documents, not its training data
- Implement a retrieval failure path that returns a useful fallback instead of a hallucinated answer

---

## Module files

| File | What it is |
|------|-----------|
| [`concepts.md`](concepts.md) | Component reference — one card per topic. Start here. |
| [`examples/basic.md`](examples/basic.md) | Pseudocode walkthrough: a knowledge base Q&A system, index to query |
| [`examples/production.md`](examples/production.md) | Same system hardened — re-ranking, token budget guard, fallback path, citation extraction |
| [`quiz.md`](quiz.md) | 6 scenario-based questions. Apply the concepts, don't recite them. |
| [`resources.md`](resources.md) | Reference repos, blog posts, and a full YouTube series for going deeper |

---

## Prerequisites

- **Module 1: API Calls** — [token budgeting](../01-api-calls/concepts.md#token-budgeting) and [request anatomy](../01-api-calls/concepts.md#request-anatomy). Context injection adds tokens to your input; you must account for them within the context limit.
- **Module 2: Chatbots** — [conversation history management](../02-chatbots/concepts.md#conversation-history-management) and [turn structure](../02-chatbots/concepts.md#turn-structure). In a RAG-powered chatbot, retrieved context is injected into the messages array alongside the conversation history.

---

## What comes next

- **Module 4: Memory** — long-term memory uses the same embedding + retrieval pattern as RAG to find relevant past context. If you understand RAG retrieval, you understand the retrieval half of memory.
- **Module 5: Agents** — agents can use RAG as a tool ("search documentation" is a tool call that runs the retrieval pipeline).

---

## Estimated time

2–2.5 hours: 60 min for concepts, 40 min for examples, 25 min for quiz

# Memory

**Phase 5 — Module 4** | Tag: `core` | Type: `theory + examples`

> An LLM has no memory by default. Every session starts blank. This module covers how to build memory that persists beyond the context window and beyond the session.

---

## What this module covers

You've already built short-term memory in Module 2 (conversation history) and the retrieval pattern that underlies long-term memory in Module 3 (RAG). This module combines them into a coherent memory architecture. It covers the two types of memory (in-context and external), how to compress history before it overflows, how to retrieve relevant memories without flooding the context, and how to handle the privacy and deletion requirements that come with storing personal data.

---

## Learning objectives

By the end of this module you should be able to:

- Describe what short-term memory is, what bounds it, and when to use it
- Implement a write path that extracts facts from a conversation and stores them
- Implement a read path that retrieves relevant memories at session start
- Apply compression to in-context history before it hits the context limit
- Score and rank stored memories by both semantic relevance and recency
- Implement a deletion mechanism that satisfies a "forget everything" user request

---

## Module files

| File | What it is |
|------|-----------|
| [`concepts.md`](concepts.md) | Component reference — one card per topic. Start here. |
| [`examples/basic.md`](examples/basic.md) | Pseudocode walkthrough: a personal assistant that remembers preferences across sessions |
| [`examples/production.md`](examples/production.md) | Same assistant hardened — TTL, compression, recency-weighted retrieval, deletion path |
| [`quiz.md`](quiz.md) | 6 scenario-based questions. Apply the concepts, don't recite them. |
| [`resources.md`](resources.md) | Blog posts, reference implementations, and GDPR guidance for going deeper |

---

## Prerequisites

- **Module 2: Chatbots** — [conversation history management](../02-chatbots/concepts.md#conversation-history-management) and [context window pressure](../02-chatbots/concepts.md#context-window-pressure). Short-term memory is the chatbot's message history; this module extends it.
- **Module 3: RAG** — [embedding and vector search](../03-rag/concepts.md#embedding-and-vector-search) and [retrieval quality](../03-rag/concepts.md#retrieval-quality). Long-term memory uses identical retrieval mechanics. If you haven't done Module 3, the read path in this module will be opaque.

---

## What comes next

- **Module 5: Agents** — agents use memory to maintain context across multi-step tasks. An agent's working memory of what it has done so far is an instance of short-term memory; its knowledge of user preferences is long-term memory.

---

## Estimated time

2 hours: 50 min for concepts, 40 min for examples, 30 min for quiz

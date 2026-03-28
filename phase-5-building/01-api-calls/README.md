# API Calls

**Phase 5 — Module 1** | Tag: `core` | Type: `theory + examples`

> The foundation of every LLM application. Before chatbots, RAG, memory, or agents — there is one HTTP request and one response. Getting this right determines everything that follows.

---

## What this module covers

You already know how to call an API. What's different about LLM APIs is that the request carries a conversation, the response costs money per token, and failures are expected under normal load. This module covers the mechanics of that request, how to budget for it, how to handle the failures, and how to avoid the cost surprises.

---

## Learning objectives

By the end of this module you should be able to:

- Construct a valid LLM request with correct message roles and parameters
- Estimate token counts before sending a request and set `max_tokens` appropriately
- Distinguish retryable errors from fatal errors and implement exponential backoff
- Choose between streaming and batch responses based on the use case
- Identify the two main cost levers and apply them in a real application

---

## Module files

| File | What it is |
|------|-----------|
| [`concepts.md`](concepts.md) | Component reference — one card per topic. Start here. |
| [`examples/basic.md`](examples/basic.md) | Pseudocode walkthrough: a single summarization call, step by step |
| [`examples/production.md`](examples/production.md) | Same call hardened for production — retries, token checks, streaming, cost logging |
| [`quiz.md`](quiz.md) | 6 scenario-based questions. Apply the concepts, don't recite them. |
| [`resources.md`](resources.md) | Cookbooks, blogs, and videos for going deeper |

---

## Prerequisites

- **Phase 4 → Module 1: LLM Fundamentals** — specifically [how models generate tokens](../phase-4-llms/01-llm-fundamentals/concepts.md#how-text-generation-works). Token budgeting only makes sense if you understand that the model generates one token at a time until it hits `max_tokens` or a stop condition.

---

## What comes next

- **Module 2: Chatbots** — builds directly on the message array. A chatbot is an application that maintains and grows the messages array across turns.
- **Module 3: RAG** — uses everything here (token budgeting, error handling) plus adds a retrieval step before the call.
- **Module 4: Memory** — extends the message array with external storage.
- **Module 5: Agents** — makes multiple API calls in a loop, making error handling and cost awareness critical.

---

## Estimated time

1–2 hours: 45 min for concepts, 30 min for examples, 20 min for quiz

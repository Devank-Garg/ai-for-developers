# Resources: Memory

Curated external resources for going deeper on memory systems in LLM applications.

---

## Blog posts

**[Lilian Weng — LLM Powered Autonomous Agents](https://lilianweng.github.io/posts/2023-06-23-agent/)**
https://lilianweng.github.io/posts/2023-06-23-agent/
OpenAI researcher Lilian Weng's deep analysis of agent architectures, with a full section on memory taxonomy: sensory memory, short-term memory, and long-term memory. The most cited conceptual treatment of LLM memory. Read the memory section specifically.

**[Pinecone — What is vector search?](https://www.pinecone.io/learn/vector-search/)**
https://www.pinecone.io/learn/vector-search/
The retrieval mechanism powering long-term memory is vector search — the same as RAG. This explainer covers embeddings, approximate nearest neighbor search, and filtering, with clear diagrams. Directly relevant to the long-term memory read path.

---

## Reference code

**[mem0 — Memory layer for AI agents](https://github.com/mem0ai/mem0)**
https://github.com/mem0ai/mem0
An open-source library dedicated specifically to memory management for LLM applications. Useful as a reference implementation: shows how production memory systems handle storage, retrieval, TTL, and multi-user isolation. Read the source even if you don't use the library.

---

## Official documentation

**[GDPR — Right to erasure ("right to be forgotten")](https://gdpr.eu/right-to-be-forgotten/)**
https://gdpr.eu/right-to-be-forgotten/
The specific GDPR article covering user deletion rights. If your memory system stores data about EU users, this is the legal baseline for what "forget everything" means technically and legally.

---

## Video

**[Google — Memory and personalization in AI assistants](https://www.youtube.com/watch?v=0eO9RLTMZHI)**
https://www.youtube.com/watch?v=0eO9RLTMZHI
Google I/O talk on how Gemini's memory and personalization features work at the system level. Covers the same design tradeoffs from concepts.md — what to store, what to surface, how to handle conflicts between stored context and current conversation.

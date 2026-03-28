# Concepts: Memory

> **Tag: core** — Memory is what separates a chatbot that knows you from one that forgets you the moment you close the tab.

---

## Short-Term (In-Context) Memory

**What it is** — The portion of the current conversation history that is included in the messages array for the current API call.

**Why it matters** — Without short-term memory, the model has no awareness of earlier turns in the same conversation. Every response is generated as if the conversation just started.

**How it works** — Short-term memory is the conversation history you already manage in Module 2. It lives entirely inside the messages array sent on each call. The model can "remember" anything in this array because it reads the full array before generating each response.

```
# Short-term memory is the messages array
messages = [
  { role: "system",    content: system_prompt },
  { role: "user",      content: "My name is Alex." },
  { role: "assistant", content: "Nice to meet you, Alex." },
  { role: "user",      content: "What's my name?" }
]
# The model answers "Alex" because the name is in the messages array.
# It is not "remembering" in any durable sense — it is reading.
```

The limit of short-term memory is the context window. As the conversation grows, you must either truncate history (losing early context) or compress it (losing fidelity). (See: Module 2 — Context window pressure.)

Short-term memory is:
- **Ephemeral** — it exists only while the session is active
- **Complete** — the model sees every word of included history
- **Bounded** — constrained by the context window

**Production tip** — Use short-term memory for the current task's working context: the specific problem being solved, decisions made in this session, and clarifications the user gave. Don't try to use it for information that predates the session — that is long-term memory's job.

**Common failure** — Treating short-term memory as permanent. A user who tells the assistant their name in session 1 will have to repeat it in session 2. If your application implies persistence ("I'll remember your preferences") but doesn't implement long-term memory, users will feel misled.

---

## Long-Term (External Store) Memory

**What it is** — A persistent, external database that stores information about the user across sessions, retrieved and injected into the context when relevant.

**Why it matters** — Short-term memory dies when the session ends. Long-term memory gives the model access to information from previous sessions — user preferences, past decisions, established context — without requiring the user to re-explain everything.

**How it works** — Long-term memory is a retrieval problem, identical in structure to RAG (See: Module 3):

**Write path** — extract and store facts during or after a session:
```
def extract_and_store(session_id, user_id, conversation_history):
  # Ask the model to identify memorable facts from the conversation
  extraction_prompt = """
  From this conversation, extract facts worth remembering about the user.
  Format: one fact per line, as a complete statement.
  Example: "User prefers Python over JavaScript."
  """
  facts = model.chat({ messages: [
    { role: "user", content: extraction_prompt + format(conversation_history) }
  ]}).content.split("\n")

  for fact in facts:
    vector = embedding_model.encode(fact)
    memory_store.insert(user_id, fact, vector, timestamp=now())
```

**Read path** — retrieve relevant memories at session start or when context is needed:
```
def load_relevant_memories(user_id, current_query):
  query_vector = embedding_model.encode(current_query)
  memories = memory_store.search(user_id, query_vector, top_k=5)
  return memories

# Inject at session start
memories = load_relevant_memories(user_id, first_message)
memory_context = "\n".join([m.text for m in memories])
system_prompt += "\n\nRelevant context about this user:\n" + memory_context
```

The key difference from RAG: memory is user-scoped (each user has their own memory store), whereas a RAG knowledge base is shared across all users.

**Production tip** — Give memories a TTL (time-to-live) and a freshness score. A memory from 2 years ago ("User was learning Python") may be stale. Weight recent memories more heavily in retrieval, and set an expiration date on facts that are likely to change.

**Common failure** — Storing raw conversation history as long-term memory. Raw conversations are noisy, repetitive, and expensive to retrieve and re-inject. Extract and store discrete, factual statements ("User prefers dark mode", "User's timezone is UTC+9") not conversation transcripts.

---

## Memory Compression and Summarization

**What it is** — The process of condensing long conversation history or accumulated memories into shorter representations that preserve the key information while consuming fewer tokens.

**Why it matters** — Uncompressed history grows without bound. Short-term memory hits the context limit; long-term memory accumulates duplicate and outdated facts. Compression is how you keep both useful over time.

**How it works** — Compression is applied in two contexts:

**In-session summarization** (compresses the current conversation when it grows long):
```
MAX_TOKENS_BEFORE_COMPRESS = 4000

def maybe_compress_history(history):
  if count_tokens(history) < MAX_TOKENS_BEFORE_COMPRESS:
    return history

  # Keep system prompt and recent turns; compress everything in between
  system    = history[0]
  recent    = history[-6:]   # keep last 3 turns (6 messages)
  to_compress = history[1:-6]

  summary_text = model.chat({ messages: [
    { role: "user", content: "Summarize this conversation in 3-5 sentences, capturing key facts and decisions:\n\n" + format(to_compress) }
  ]}).content

  compressed_turn = { role: "user", content: "[Conversation summary: " + summary_text + "]" }
  return [system, compressed_turn] + recent
```

**Long-term memory consolidation** (merges duplicate or related memories):
```
def consolidate_memories(user_id):
  all_memories = memory_store.load_all(user_id)
  # Find and merge duplicates (same fact stated differently)
  # Supersede outdated facts with newer ones
  # Remove memories past their TTL
```

Summarization introduces lossy compression — detail is lost. The tradeoff is acceptable for context that's less immediately relevant, but you should never compress the most recent turns (they contain the active task context).

**Production tip** — Always preserve the last 3–5 turns uncompressed, regardless of total history length. These turns contain the immediate context the model needs to continue the conversation coherently. Compressing the most recent turns is the most common mistake in history management.

**Common failure** — Using a summary to replace all history, then summarizing the summary again on the next compression cycle. Summaries of summaries rapidly lose fidelity — after 2–3 cycles, you're left with a vague paraphrase of the original conversation. Fix: always summarize from the raw turns, not from a prior summary.

---

## Memory Retrieval Relevance

**What it is** — The process of selecting which memories to inject for a given conversation, rather than loading everything the user has ever told the assistant.

**Why it matters** — A user might have 500 stored memories. Injecting all of them would consume the entire context window and bury the relevant information in noise. Selective retrieval keeps the context clean and focused.

**How it works** — Memory retrieval uses the same vector search pattern as RAG (See: Module 3 — Embedding and vector search):

```
def get_relevant_memories(user_id, current_context):
  # Embed the current message or conversation summary
  context_vector = embedding_model.encode(current_context)

  # Retrieve memories most similar to the current context
  memories = memory_store.search(
    user_id=user_id,
    query_vector=context_vector,
    top_k=10,
    filters={ recency_boost: True }
  )

  return memories
```

Relevance has two components:
- **Semantic similarity** — the memory is related to what's being discussed
- **Recency** — recent memories are often more relevant than old ones

A combined score:
```
combined_score = (0.7 × semantic_similarity) + (0.3 × recency_score)
```

The weights are tunable — for a short-term assistant, recency might be less important; for a long-term personal assistant, recency is often the dominant signal.

**Production tip** — Retrieve memories at the start of a session based on the first message, not at the start of every turn. Mid-conversation memory retrieval adds latency and may inject context that contradicts what was established earlier in the session.

**Common failure** — Injecting irrelevant old memories that conflict with what the user has told the assistant in the current session. If the user said "I prefer Python" in a past session and "I'm working in Go now" in this session, injecting the old Python preference creates confusion. Current session context should always take priority over stored long-term memory.

---

## Privacy and Persistence Tradeoffs

**What it is** — The design decisions around what memory to store, how long to keep it, who can access it, and how users can control or delete it.

**Why it matters** — Storing personal information without consent or a deletion mechanism creates legal and ethical liability. Users who discover their data is being retained without their knowledge lose trust. In regulated industries, retention without a legal basis is a compliance violation.

**How it works** — Memory persistence has four dimensions:

**Scope** — what gets stored:
```
# Store: explicit preferences, stated facts, user-provided context
# Don't store: sensitive PII (health, financial, authentication data) without consent
# Never store: content the user intended to be ephemeral ("hypothetically...")
```

**Retention** — how long it's kept:
```
memory = {
  text:       "User prefers concise responses",
  user_id:    user_id,
  created_at: now(),
  expires_at: now() + 90_days,   # TTL — auto-delete after 90 days
  category:   "preference"       # enables category-based deletion
}
```

**Access** — who can read stored memories:
```
# Memories are scoped to user_id — no cross-user access
# Memories are NOT accessible by other users or other assistants
# Memories of minors: consult legal before storing any
```

**Deletion** — user-controlled removal:
```
def delete_user_memories(user_id, category=None):
  if category:
    memory_store.delete_where(user_id=user_id, category=category)
  else:
    memory_store.delete_all(user_id=user_id)
  log_audit("memory_deletion", user_id=user_id, category=category)
```

A "forget everything" request must delete all stored data, not just clear the current session. Users have a legitimate expectation that "forget" means permanent deletion.

**Production tip** — Implement a memory dashboard where users can view and delete what the assistant knows about them. Transparency about stored data is required by GDPR (Europe) and similar regulations. Build the deletion path before you launch, not after you receive a legal demand.

**Common failure** — Storing memories without a TTL or deletion mechanism. When a user asks the assistant to "forget that," the code has no way to comply if deletion was never implemented. Retrofitting deletion after launch requires identifying and removing stored data across all records, which is significantly more complex than building it in from the start.

---

## Further reading

**Memory systems in AI (conceptual)** — [https://lilianweng.github.io/posts/2023-06-23-agent/](https://lilianweng.github.io/posts/2023-06-23-agent/)
Lilian Weng's overview of agent architectures includes a taxonomy of memory types (sensory, short-term, long-term) that maps well to the cards above.

**GDPR and AI systems** — [https://gdpr.eu/what-is-gdpr/](https://gdpr.eu/what-is-gdpr/)
If you store personal data about EU users, GDPR applies. This overview covers the key requirements relevant to memory systems (consent, retention limits, right to erasure).

**Vector database comparison** — [https://benchmark.vectorview.ai/vectordbs.html](https://benchmark.vectorview.ai/vectordbs.html)
Benchmarks for choosing a vector store for memory retrieval at scale.

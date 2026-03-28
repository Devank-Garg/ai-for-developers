# Production Example: Memory

> Same scenario as `basic.md` — a personal productivity assistant — now hardened
> for real use. Failure modes are injected deliberately; read the annotations.

---

## What changes from the basic example

- **TTL on stored memories** — facts expire automatically after 90 days
- **Recency-weighted retrieval** — recent memories are ranked higher than old ones
- **In-session compression** — long history is summarized before hitting the context limit
- **Priority ordering** — current session context beats long-term memory when they conflict
- **Deletion path** — user-controlled "forget everything" that actually works

---

## The full example

```
MEMORY_TTL_DAYS         = 90
MAX_HISTORY_TOKENS      = 5000
COMPRESSION_KEEP_RECENT = 6   # keep last 3 turns (6 messages) uncompressed
RECENCY_WEIGHT          = 0.3
SEMANTIC_WEIGHT         = 0.7


# --- WRITE PATH ---

def end_session(user_id, history, session_id):
  # Extract and store memorable facts from this session
  facts = extract_memories_from_history(history)

  for fact in facts:
    vector = embedding_model.encode(fact)
    memory_store.insert({
      user_id:    user_id,
      session_id: session_id,
      text:       fact,
      vector:     vector,
      created_at: now(),
      # WHY: TTL ensures stale memories auto-expire.
      # "User is learning Python" might be wrong a year from now.
      expires_at: now() + (MEMORY_TTL_DAYS * 86400)
    })


# --- READ PATH ---

def start_session(user_id, first_message):
  query_vector = embedding_model.encode(first_message)

  # WHY: retrieve with recency boosting — recent memories are more likely
  # to be accurate than memories from years ago.
  raw_memories = memory_store.search(
    user_id=user_id,
    query_vector=query_vector,
    top_k=20,                    # retrieve broadly first
    exclude_expired=True         # filter out TTL-expired memories
  )

  # Combined scoring: semantic similarity + recency
  scored = []
  for m in raw_memories:
    age_days = (now() - m.created_at) / 86400
    recency_score = max(0, 1 - (age_days / MEMORY_TTL_DAYS))  # 1.0 for new, 0.0 at TTL
    combined = (SEMANTIC_WEIGHT * m.similarity) + (RECENCY_WEIGHT * recency_score)
    scored.append((combined, m))

  # Sort by combined score, take top 5
  scored.sort(reverse=True)
  top_memories = [m for _, m in scored[:5]]

  memory_text = "\n".join(["- " + m.text for m in top_memories])

  history = [{
    role: "system",
    content: """You are a personal productivity assistant.

What you know about this user (from past sessions):
""" + memory_text + """

IMPORTANT: If the user states something in this conversation that contradicts
the above, trust what they say NOW over stored memory.
"""
  }]

  history.append({ role: "user", content: first_message })
  return history


# --- IN-SESSION COMPRESSION ---

def handle_message(history, user_message):
  history.append({ role: "user", content: user_message })

  # WHY: check before sending — context overflow errors are silent data loss.
  if count_tokens(history) > MAX_HISTORY_TOKENS:
    history = compress_history(history)

  response = model.chat({
    model: "claude-sonnet-4-6",
    messages: history,
    max_tokens: 500
  })

  history.append({ role: "assistant", content: response.content })
  return history, response.content


def compress_history(history):
  system   = history[0]    # always preserve system prompt
  recent   = history[-COMPRESSION_KEEP_RECENT:]  # always preserve recent turns
  to_compress = history[1:-COMPRESSION_KEEP_RECENT]

  if not to_compress:
    raise FatalError("Cannot compress — only recent turns remain")

  # WHY: summarize from raw turns, not from a previous summary.
  # Summarizing summaries loses fidelity rapidly over multiple cycles.
  summary = model.chat({
    model: "small-fast-model",
    messages: [{ role: "user", content:
      "Summarize this conversation concisely, preserving key facts and decisions:\n\n" +
      format(to_compress)
    }],
    max_tokens: 200
  }).content

  summary_turn = {
    role: "user",
    content: "[Earlier conversation summary: " + summary + "]"
  }

  log_info("history_compressed", turns_compressed=len(to_compress)//2)
  return [system, summary_turn] + recent


# --- DELETION PATH ---

def delete_user_memories(user_id, category=None):
  """
  Called when user requests 'forget everything' or selective deletion.
  WHY: must delete from persistent storage, not just clear session state.
  Clearing session history without deleting the memory store means
  the next session re-injects the "forgotten" memories.
  """
  if category:
    count = memory_store.delete_where(user_id=user_id, category=category)
  else:
    count = memory_store.delete_all(user_id=user_id)

  # WHY: audit log is required for compliance (GDPR right to erasure).
  log_audit("memory_deletion", user_id=user_id, count=count, category=category)
  return count
```

---

## Failure mode walkthrough

### Failure 1: Stale memory contradicts current session context

**What happens:** A user said "I use Python" 6 months ago. That memory is retrieved and injected. The user opens a new session and says "I'm working in TypeScript now." The assistant keeps referring to Python throughout the session because the stored memory is heavily weighted.

**Why it happens:** The system injects long-term memory into the system prompt but has no mechanism to deprioritize it when the current conversation contradicts it. The model is reading both signals and doesn't know which to trust.

**How the example handles it:** The system prompt explicitly states: "If the user states something in this conversation that contradicts the above, trust what they say NOW over stored memory." This instruction tells the model how to resolve conflicts. It also uses recency-weighted scoring to reduce the influence of old memories over time.

**What would break without this:** The assistant behaves inconsistently — it applies outdated preferences and the user must repeat corrections every session.

---

### Failure 2: "Forget everything" doesn't actually forget

**What happens:** A user asks the assistant to forget all personal information. The app clears the current session's history. The next session starts, retrieves memories from the memory store, and injects them — the user's "forgotten" data is back.

**Why it happens:** The deletion only cleared the in-memory session state, not the persistent memory store. Clearing `history` in memory does not affect the database.

**How the example handles it:** `delete_user_memories()` calls `memory_store.delete_all(user_id)` — deleting from the persistent store, not from session state. The audit log records the deletion for compliance. The next session starts with no retrievable memories.

**What would break without this:** Users who request data deletion still have their data retained. In GDPR-regulated markets, this is a compliance violation. In all markets, it destroys user trust.

---

### Failure 3: Summarizing a summary, accumulating information loss

**What happens:** In session 1, turns 1–20 are compressed into a summary. In session 2, that summary is loaded as context, and when history grows long again, the summary is compressed into another summary. After several sessions, the "summary" is a vague sentence that barely represents the original conversation.

**Why it happens:** If the compression function reads prior summaries from history and summarizes them again, each cycle removes more detail. The model summarizing a summary has less source material to work with.

**How the example handles it:** The `compress_history()` function only compresses `to_compress` — the raw turns between the system prompt and the recent turns. A stored `[Earlier conversation summary]` message is treated as a past "user" turn and is also eligible for compression in a future session, but the function always summarizes from the raw text in the array, not by calling itself recursively.

**What would break without this:** After 3–4 sessions, the long-term context degrades to near-nothing. The assistant appears to "forget" more and more over time despite having a memory system.

---

## Production checklist for this module

- [ ] Assign TTL to every stored memory — no memory should be permanent by default
- [ ] Weight memories by recency, not just semantic similarity
- [ ] Explicitly instruct the model to prioritize current session context over stored memory
- [ ] Compress history before sending when token count approaches the limit
- [ ] Always keep the last 3–5 turns uncompressed — they carry the active task context
- [ ] Implement and test the deletion path before launch; don't retrofit it after a user complaint

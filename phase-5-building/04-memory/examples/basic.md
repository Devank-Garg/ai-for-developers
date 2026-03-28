# Basic Example: Memory

> Pseudocode walkthrough — no real SDK imports. The goal is to understand
> the read and write paths for a memory system, not to copy runnable code.

---

## Scenario

A personal productivity assistant that remembers user preferences and past decisions across sessions. The user sets preferences once and the assistant applies them in every subsequent session.

---

## Phase 1: Write path — extracting and storing memories after a session

This runs once at session end, not on every turn.

### Step 1: Extract memorable facts from the conversation

```
def extract_memories(user_id, conversation_history):
  # WHY: don't store raw conversation history — extract discrete facts.
  # Raw history is noisy, large, and hard to retrieve selectively.
  extraction_prompt = """
  From this conversation, extract facts worth remembering about the user.
  Include: preferences, stated goals, decisions made, relevant personal context.
  Exclude: ephemeral task details, things the user said hypothetically.
  Format: one fact per line, as a complete statement.
  Example outputs:
    "User prefers responses in bullet points."
    "User is building a Python web app using FastAPI."
  """

  response = model.chat({
    model: "small-fast-model",  # extraction is a simple task — no need for large model
    messages: [
      { role: "user", content: extraction_prompt + "\n\nConversation:\n" + format(conversation_history) }
    ],
    max_tokens: 300
  })

  facts = response.content.strip().split("\n")
  facts = [f.strip() for f in facts if f.strip()]  # remove empty lines
  return facts
```

### Step 2: Store extracted facts with embeddings

```
def store_memories(user_id, facts):
  for fact in facts:
    # WHY: embed each fact so it can be retrieved by semantic similarity.
    # Same pattern as RAG indexing — one embedding per stored item.
    vector = embedding_model.encode(fact)

    memory_store.insert({
      user_id:    user_id,
      text:       fact,
      vector:     vector,
      created_at: now()
    })

  print("Stored " + len(facts) + " memories for user " + user_id)
```

---

## Phase 2: Read path — retrieving relevant memories at session start

### Step 3: Load relevant memories based on the first message

```
def start_session_with_memory(user_id, first_message):
  # WHY: retrieve memories relevant to what the user is asking about now.
  # Don't load all memories — that floods the context with irrelevant history.
  query_vector = embedding_model.encode(first_message)

  relevant_memories = memory_store.search(
    user_id=user_id,
    query_vector=query_vector,
    top_k=5   # inject at most 5 memories
  )

  return relevant_memories
```

### Step 4: Inject memories into the system prompt

```
def build_session(user_id, first_message):
  memories = start_session_with_memory(user_id, first_message)

  # WHY: inject memories into the system prompt, not as user messages.
  # They represent standing context about the user, not a conversation turn.
  if memories:
    memory_text = "\n".join(["- " + m.text for m in memories])
    memory_section = "\n\nWhat you know about this user:\n" + memory_text
  else:
    memory_section = ""

  history = [
    {
      role: "system",
      content: "You are a personal productivity assistant." + memory_section
    },
    { role: "user", content: first_message }
  ]

  return history
```

### Step 5: Continue the session normally

```
def continue_session(history, user_message):
  history.append({ role: "user", content: user_message })

  response = model.chat({
    model: "claude-sonnet-4-6",
    messages: history,
    max_tokens: 500
  })

  history.append({ role: "assistant", content: response.content })
  return history, response.content


# At session end: extract and store new memories
def end_session(user_id, history):
  new_facts = extract_memories(user_id, history)
  store_memories(user_id, new_facts)
```

---

## What this example shows

This walkthrough demonstrates three of the five component cards from `concepts.md`:

- **Short-term (in-context) memory** — the `history` array within a session (Steps 4–5)
- **Long-term (external store) memory** — the write path (extract + store) and read path (retrieve + inject) across sessions (Steps 1–4)
- **Memory retrieval relevance** — retrieving only the top-5 semantically relevant memories, not all stored facts (Step 3)

It deliberately leaves out:

- **Memory compression** — this example has no history compression. In a long session, history will grow until the context limit is hit. See `production.md`.
- **Privacy and persistence tradeoffs** — no TTL, no deletion mechanism. See `production.md` for the full lifecycle including user-controlled deletion.

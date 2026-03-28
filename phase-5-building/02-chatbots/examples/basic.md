# Basic Example: Chatbots

> Pseudocode walkthrough — no real SDK imports. The goal is to understand
> the structure of a multi-turn chatbot, not to copy runnable code.

---

## Scenario

A software product support chatbot that answers questions about installation, billing, and known bugs. Users can ask multiple questions in one session and reference things they said earlier.

---

## Step 1: Initialize the session

```
def start_session(session_id):
  # WHY: The system prompt is set once at session start and persists for
  # the entire conversation. It never changes mid-session.
  system_prompt = """
  You are a customer support agent for Acme software.
  Answer questions about installation, billing, and known bugs.
  Do not answer questions outside this scope.
  Be direct. Use numbered steps for instructions.
  """

  history = [
    { role: "system", content: system_prompt }
    # WHY: The system message is the only content here at session start.
    # User and assistant turns will be appended as the conversation progresses.
  ]

  # WHY: Persist the history immediately, even though it's just the system prompt.
  # The session exists from this point forward.
  db.save(session_id, history)

  return session_id
```

---

## Step 2: Receive a user message and append it

```
def handle_user_message(session_id, user_message):
  # WHY: Load history from storage, not from memory.
  # If this function runs on a different server process than the last call,
  # in-memory history would be empty.
  history = db.load(session_id)

  # WHY: Append the user turn before calling the model.
  # The model needs to see the new message as part of the array.
  history.append({
    role: "user",
    content: user_message
  })
```

---

## Step 3: Send the full history to the model

```
  # WHY: We send the ENTIRE history on every call, not just the latest message.
  # The model has no memory — it only knows what's in the messages array.
  response = model.chat({
    model: "claude-sonnet-4-6",
    messages: history,
    max_tokens: 500,
    temperature: 0.3
  })

  assistant_reply = response.content
```

---

## Step 4: Append the assistant reply and persist

```
  # WHY: Append the assistant's reply so it becomes part of the next call's context.
  # Without this, the model has no memory of its own previous answers.
  history.append({
    role: "assistant",
    content: assistant_reply
  })

  # WHY: Save after every turn. If the server crashes between turns,
  # the next call can resume from the last saved state.
  db.save(session_id, history)

  return assistant_reply
```

---

## Step 5: What the history looks like after three turns

```
# After the user has asked three questions:
[
  { role: "system",    content: "You are a customer support agent..." },
  { role: "user",      content: "How do I install Acme on Windows?" },
  { role: "assistant", content: "1. Download the installer from acme.com/download..." },
  { role: "user",      content: "It says 'missing DLL'. What does that mean?" },
  { role: "assistant", content: "That error means a required library is missing..." },
  { role: "user",      content: "I tried reinstalling and still get it." }
]

# The model sees all 7 messages and generates the next assistant turn.
# Notice: the user's third message implicitly refers to "it" (the DLL error)
# and "reinstalling" — context the model can only understand because of turns 2-4.
```

---

## What this example shows

This walkthrough demonstrates four of the six component cards from `concepts.md`:

- **Conversation history management** — loading, appending, and persisting the history array across turns
- **System prompt design** — initializing the history with a system message that persists for the session
- **Turn structure** — maintaining the system → user → assistant → user → assistant alternation
- **Session lifecycle** — start (Step 1), continue (Steps 2–4), and implicit persistence on every turn

It deliberately leaves out:

- **Context window pressure** — this example has no trimming logic; after enough turns it will hit the context limit. See `production.md` for the trimming strategy.
- **Response formatting** — the system prompt sets basic formatting, but there's no schema enforcement or JSON parsing. See `production.md`.

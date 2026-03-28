# Concepts: Chatbots

> **Tag: core** — A chatbot is not a special API mode — it is what happens when you maintain the messages array across multiple turns and manage it carefully.

---

## Conversation History Management

**What it is** — The practice of building, storing, and updating the messages array that represents the full conversation between a user and the model.

**Why it matters** — LLMs are stateless. Every call is independent. The model has no memory of previous turns unless you explicitly include them in the messages array. Conversation history is the mechanism that creates the illusion of continuity.

**How it works** — Each user turn appends a `user` message to the array. After the model responds, the assistant's reply is appended as an `assistant` message. The next call sends the entire array from the beginning.

```
history = []

def chat(user_message):
  # Add the new user turn
  history.append({ role: "user", content: user_message })

  # Send the full history
  response = model.chat({
    model: "claude-sonnet-4-6",
    messages: history,
    max_tokens: 500
  })

  # Add the model's reply so it's part of the next call
  history.append({ role: "assistant", content: response.content })

  return response.content
```

After three turns, the messages array might look like:
```
[
  { role: "system",    content: "You are a helpful assistant." },
  { role: "user",      content: "What is recursion?" },
  { role: "assistant", content: "Recursion is when a function calls itself..." },
  { role: "user",      content: "Can you give me a Python example?" },
  { role: "assistant", content: "Sure. Here's a factorial function..." },
  { role: "user",      content: "What's the maximum recursion depth in Python?" }
]
```

The model reads this array and generates the next assistant turn. Without the prior turns, the third question has no context.

**Production tip** — Store the history in a data structure you control, not in memory. In-memory history dies when the server restarts, when a user opens a second tab, or when you deploy. Persist history to a database keyed by session ID.

**Common failure** — Mutating the history array in place while a concurrent request is reading it. In any environment with concurrent users, each session needs its own isolated history object. Sharing a single global `history` array across users merges their conversations.

---

## System Prompt Design

**What it is** — The `system` role message that sets the model's behavior, persona, and constraints for the entire conversation.

**Why it matters** — Without a system prompt, the model defaults to a generic assistant persona. With one, you control tone, task scope, output format, and what the model should refuse — before any user input arrives.

**How it works** — The system message is always the first entry in the messages array. It is not shown to the user and does not count as a turn. It persists across every turn of the conversation.

A useful system prompt answers four questions:
1. What is this assistant for? ("You are a customer support agent for Acme software.")
2. What should it do? ("Answer questions about billing, installation, and known bugs.")
3. What should it avoid? ("Do not answer questions outside of Acme software support.")
4. How should it respond? ("Be concise. Use numbered lists for step-by-step instructions.")

```
system_prompt = """
You are a customer support agent for Acme software.

Your scope:
- Answer questions about billing, installation, and known bugs
- Direct users to docs.acme.com for feature requests
- Escalate to human support if the user is frustrated or the issue is unresolved after 3 turns

Response style:
- Be direct and concise
- Use numbered steps for instructions
- Do not speculate about future product features
"""

messages = [
  { role: "system", content: system_prompt },
  ...user turns...
]
```

**Production tip** — Keep system prompts version-controlled and treat them like code. A change to the system prompt changes the behavior of every session that starts after the deployment. Include the prompt version in your logs so you can correlate behavioral changes to deployments.

**Common failure** — Placing the system prompt after user messages in the array. While some models tolerate this, most treat the system role as authoritative only when it appears first. A user message before the system prompt can cause the model to partially ignore the system instructions.

---

## Turn Structure

**What it is** — The required alternating pattern of `user` and `assistant` messages in the conversation array.

**Why it matters** — LLMs are trained to expect alternating turns. Consecutive messages of the same role (two `user` messages in a row, or two `assistant` messages) produce unpredictable behavior — the model may ignore turns, merge them, or hallucinate responses to fill the gap.

**How it works** — The valid pattern is:
```
system → user → assistant → user → assistant → user → assistant → ...
```

The system message appears once at the start. After that, the conversation alternates strictly between user and assistant. The model always generates the next `assistant` turn.

Common cases where turn structure breaks:
- User sends multiple messages before getting a response (buffering required)
- System needs to inject information mid-conversation (inject into a `user` turn or as a new system message, depending on the provider)
- Prefilling the assistant's response (some providers allow starting an `assistant` message to steer format)

```
# Buffering multiple user inputs before sending
pending_messages = []

def on_user_input(text):
  pending_messages.append(text)

def on_submit():
  # Combine buffered inputs into one user turn
  combined = "\n".join(pending_messages)
  history.append({ role: "user", content: combined })
  pending_messages.clear()
  send_and_respond()
```

**Production tip** — Validate the turn structure before sending. Assert that the last message in the array is a `user` message (or a `user`-role injected context message). Sending a request ending in `assistant` will fail or produce nonsense.

**Common failure** — Injecting tool output or retrieved context as a bare `user` message. This works but pollutes the conversation history with retrieval artifacts that the user never typed. Providers that support a `tool` role should use it; otherwise, wrap injected content in a labeled block so the model — and your logs — can distinguish it from user input.

---

## Context Window Pressure

**What it is** — The gradual increase in input token count as conversation history grows, which eventually reaches the model's context limit.

**Why it matters** — A conversation with enough turns will exceed the context limit and cause a hard API error. In a long support session or a daily-use assistant, this is not a hypothetical — it will happen.

**How it works** — Every turn adds tokens. A typical conversational turn is 50–200 tokens. With a 128,000-token context limit, you can have roughly 300–800 turns before the limit is reached. With older models at 4,096 tokens, you might hit it after 10–20 turns if the user pastes in large blocks of text.

(See: Module 01 — Token budgeting for how to count tokens and set limits.)

When history grows too long, you have three strategies:

**Truncation** — drop the oldest turns:
```
MAX_HISTORY_TOKENS = 3000

def trim_history(history, system_prompt):
  while count_tokens(history) > MAX_HISTORY_TOKENS:
    # Remove oldest user+assistant pair (indices 1 and 2, after system)
    history.pop(1)
    history.pop(1)
  return history
```

**Summarization** — compress old turns into a summary message:
```
if count_tokens(history) > MAX_HISTORY_TOKENS:
  old_turns = history[1:10]  # oldest non-system turns
  summary = model.chat(summarize_conversation(old_turns))
  history = [history[0]] + [{ role: "user", content: "[Context summary: " + summary + "]" }] + history[10:]
```

**Sliding window** — keep only the last N turns plus the system prompt.

Truncation is the simplest; summarization preserves more context; sliding window is the easiest to implement and reason about.

**Production tip** — Track context window usage proactively, not reactively. Check token count before every call and trim before it fails. Handling a context overflow error after the fact means the user's message was lost.

**Common failure** — Removing only user turns when trimming. If you remove a `user` message without removing the subsequent `assistant` message, you break the alternating turn structure. Always remove pairs.

---

## Response Formatting

**What it is** — Controlling the structure and style of the model's output — whether it responds in plain prose, markdown, JSON, or another format.

**Why it matters** — The model generates whatever format seems appropriate given the conversation. Without explicit instructions, a technical question might get markdown headers in a context where markdown isn't rendered, or JSON might include explanatory prose that breaks a parser.

**How it works** — Format is controlled through the system prompt and, for structured outputs, through explicit instructions in the user turn:

**Plain prose:**
```
system: "Respond in plain text. Do not use markdown formatting."
```

**Markdown (for rendered interfaces):**
```
system: "Use markdown formatting. Use headers for sections, bullet points for lists."
```

**Structured JSON:**
```
system: "Respond only with valid JSON. No explanation, no preamble."
user:   "Extract the name, date, and amount from: 'Invoice from Acme, dated 2025-03-01, total $450'"
# Expected: { "name": "Acme", "date": "2025-03-01", "amount": 450 }
```

Some providers offer a native "JSON mode" or "structured outputs" parameter that enforces schema-valid JSON. When available, use it — it is more reliable than prompt instructions alone.

**Production tip** — For JSON extraction, always validate the response against a schema before using it. Even with JSON mode enabled, unexpected inputs can produce malformed output. A schema validation step catches this before downstream code crashes.

**Common failure** — Asking the model to return JSON but receiving a response like `"Here is the JSON you requested:\n\n```json\n{...}\n```"`. The model adds helpful preamble by default. Fix this by being explicit: "Respond ONLY with valid JSON. No preamble, no explanation, no code blocks."

---

## Session Lifecycle

**What it is** — The events that mark the beginning, continuation, and end of a user's conversation, and the code that manages state across those events.

**Why it matters** — Without explicit session lifecycle management, conversations bleed between users, history accumulates infinitely, and there's no clean way to end a conversation and start a new one.

**How it works** — A session has four lifecycle events:

```
START      → create session ID, initialize empty history, inject system prompt
CONTINUE   → load history by session ID, append turn, call model, persist updated history
END        → archive or delete history, release session ID
TIMEOUT    → auto-end after N minutes of inactivity
```

Implementation:
```
def start_session(user_id):
  session_id = generate_id()
  history = [{ role: "system", content: system_prompt }]
  db.save(session_id, history, user_id=user_id)
  return session_id

def continue_session(session_id, user_message):
  history = db.load(session_id)
  if history is None:
    raise SessionNotFoundError(session_id)
  history.append({ role: "user", content: user_message })
  response = model.chat({ messages: history, ... })
  history.append({ role: "assistant", content: response.content })
  db.save(session_id, history)
  return response.content

def end_session(session_id):
  db.archive(session_id)
  db.delete_active(session_id)
```

**Production tip** — Set an inactivity timeout at the session level. A session that hasn't received a message in 30 minutes is almost certainly abandoned. Auto-archiving it prevents history accumulation and reduces storage costs.

**Common failure** — Reusing the same session ID across different users because the ID was derived from a shared value (e.g., the user's email address without hashing, or a constant in the code). One user's conversation bleeds into another's. Session IDs must be unguessable, unique per conversation, and bound to a single user's auth context.

---

## Further reading

**Prompt engineering guide (system prompts)** — [https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/system-prompts](https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/system-prompts)
Anthropic's guidance on structuring system prompts. Provider-specific but the principles apply broadly.

**Context window sizes by model** — [https://artificialanalysis.ai/](https://artificialanalysis.ai/)
Track current context limits across providers — they change as models are updated.

**Session management patterns** — [https://12factor.net/processes](https://12factor.net/processes)
The 12-Factor App principle on stateless processes. The same logic applies to why chatbot session state belongs in a database, not in application memory.

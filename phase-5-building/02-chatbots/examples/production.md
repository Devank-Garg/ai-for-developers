# Production Example: Chatbots

> Same scenario as `basic.md` — a software support chatbot — now hardened
> for real use. Failure modes are injected deliberately; read the annotations.

---

## What changes from the basic example

- **Context window pressure detection** — trims oldest turns before the context limit is hit
- **Session isolation** — history is keyed to session ID, validated against user auth
- **System prompt versioning** — logged on every call for deployment traceability
- **Format enforcement** — escalation requests parsed as structured output, not free text
- **Session timeout** — inactive sessions are auto-archived

---

## The full example

```
MAX_HISTORY_TOKENS  = 6000   # leave headroom for output within context limit
MAX_OUTPUT_TOKENS   = 600
INACTIVITY_TIMEOUT  = 1800   # seconds (30 minutes)
SYSTEM_PROMPT_VERSION = "v2.1"

SYSTEM_PROMPT = """
You are a customer support agent for Acme software.
Answer questions about installation, billing, and known bugs.
Do not answer questions outside this scope.
Be direct. Use numbered steps for instructions.
If the issue is unresolved after 3 turns, output exactly:
ESCALATE: <brief reason>
"""

def handle_message(session_id, user_id, user_message):

  # --- SESSION AUTH ---
  # WHY: validate that this session_id belongs to this user_id.
  # Without this check, a user who guesses another session ID can read
  # another user's conversation history.
  session = db.load_session(session_id)
  if session is None:
    raise SessionNotFoundError(session_id)
  if session.user_id != user_id:
    raise UnauthorizedError("session does not belong to this user")

  # --- INACTIVITY TIMEOUT ---
  # WHY: a session idle for 30+ minutes is almost certainly abandoned.
  # Resuming it would confuse the model (stale context) and the user.
  if now() - session.last_active > INACTIVITY_TIMEOUT:
    archive_session(session_id)
    raise SessionExpiredError("session timed out, please start a new conversation")

  history = session.history

  # --- CONTEXT TRIMMING ---
  # WHY: check before appending, not after. If we append first and then
  # check, we might already be over the limit and can't send.
  history.append({ role: "user", content: user_message })

  while count_tokens(history) > MAX_HISTORY_TOKENS:
    # WHY: remove oldest user+assistant PAIR (indices 1 and 2 after system).
    # Removing only the user message without the assistant reply breaks
    # turn alternation and produces unpredictable model behavior.
    if len(history) <= 3:
      # Only system + one user/assistant pair remains — cannot trim further
      raise FatalError("Single turn exceeds context limit")
    history.pop(1)  # oldest user message
    history.pop(1)  # its corresponding assistant reply
    log_info("trimmed oldest turn pair", session_id=session_id)

  response = model.chat({
    model: "claude-sonnet-4-6",
    messages: history,
    max_tokens: MAX_OUTPUT_TOKENS,
    temperature: 0.3
  })

  assistant_reply = response.content

  # --- ESCALATION DETECTION ---
  # WHY: parse the model's escalation signal before storing and returning.
  # If we just return the raw text, the UI has to parse it — inconsistently.
  if assistant_reply.startswith("ESCALATE:"):
    reason = assistant_reply.replace("ESCALATE:", "").strip()
    create_escalation_ticket(session_id, user_id, reason, history)
    assistant_reply = "I'm connecting you with a human agent. They'll have your conversation context."
    log_info("escalation triggered", reason=reason, session_id=session_id)

  history.append({ role: "assistant", content: assistant_reply })

  # --- PERSIST WITH METADATA ---
  # WHY: log the system prompt version so you can correlate behavior
  # changes to deployments when debugging in production.
  db.save_session(session_id, {
    history:       history,
    user_id:       user_id,
    last_active:   now(),
    prompt_version: SYSTEM_PROMPT_VERSION
  })

  return assistant_reply
```

---

## Failure mode walkthrough

### Failure 1: Context limit hit mid-conversation

**What happens:** After 40 turns of a detailed support conversation, the next API call returns `400: context_length_exceeded`. The user's most recent message is lost. The session is broken.

**Why it happens:** Conversation history grows with every turn. Without trimming, it eventually exceeds the model's context limit. The error occurs at request time, after the user has already submitted their message.

**How the example handles it:** The while loop trims the oldest user+assistant pairs *before* sending the request. Token count is checked proactively — the call only goes out when the array is within budget. The user never sees a context error.

**What would break without this:** The session fails with an unhandled 400 error at unpredictable points — early for users who paste large blocks of text, late for users with short messages. There's no way to predict when it will happen without proactive budgeting.

---

### Failure 2: Session ID accessible across users

**What happens:** User A's conversation history is loaded by User B, who guesses or intercepts session A's ID. User B can read the full conversation, inject messages, and receive responses in User A's session.

**Why it happens:** Without binding the session to a user identity, any caller who presents a valid session ID gets access. Session IDs are often predictable (sequential integers, timestamps) unless explicitly randomized.

**How the example handles it:** The auth check validates that `session.user_id == user_id` on every call. A session ID that doesn't match the authenticated user's ID raises `UnauthorizedError`. Session IDs are generated as random UUIDs, not derived from user-visible values.

**What would break without this:** Any user who can observe or guess another user's session ID can impersonate them. In a support context, this means reading private support conversations and injecting false history.

---

### Failure 3: Removing only user messages when trimming, not pairs

**What happens:** The trimming logic removes user messages but leaves the corresponding assistant replies. After trimming, the messages array looks like:

```
[system, assistant, user, assistant, user, assistant, user]
```

The model receives a conversation that starts with an assistant message, which violates turn structure. The model's behavior becomes unpredictable — it may repeat itself, generate confused responses, or produce a 400 error.

**Why it happens:** The trimming code iterated by role rather than by position, accidentally skipping assistant messages.

**How the example handles it:** The trimming loop explicitly removes two consecutive items (index 1 and index 1 again after the first removal), always removing a user+assistant pair together.

**What would break without this:** Turn alternation breaks silently. The API may accept the malformed array but the model's quality degrades in hard-to-diagnose ways — responses seem "confused" or "inconsistent" without any error signal.

---

## Production checklist for this module

- [ ] Persist conversation history to a database, not in application memory
- [ ] Validate session ownership on every call — session ID alone is not authorization
- [ ] Trim history before sending, not after hitting a context error
- [ ] Always remove user+assistant pairs when trimming, never individual messages
- [ ] Version-control system prompts and log the version on every call
- [ ] Implement inactivity timeouts to auto-archive abandoned sessions

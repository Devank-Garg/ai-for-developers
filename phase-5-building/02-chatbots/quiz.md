# Quiz: Chatbots

These questions are scenario-based. The goal is to apply what you know, not recite it. Attempt each before reading the answer.

---

## Questions

**1.** Your chatbot is giving inconsistent answers to the same question asked in different sessions. Two users ask "How do I reset my password?" and get different responses. The system prompt hasn't changed. Name two component cards from this module that could be causing this, and explain how each one would produce the inconsistency.

---

**2.** A user's session has been active for 2 hours and involves detailed troubleshooting. The next API call fails with `400: context_length_exceeded`. You have three recovery strategies available: truncation (drop oldest turns), summarization (compress old turns into a summary), and sliding window (keep only the last N turns). For this specific scenario — a long technical troubleshooting session — which strategy would you choose and why? What would you lose with each of the other two?

---

**3.** You're reviewing a colleague's chatbot implementation and notice the history array is stored in a module-level variable:

```
history = []  # global, initialized once when the module loads

def chat(user_message):
  history.append({ role: "user", content: user_message })
  ...
```

The app has been running fine in local testing with one user. Describe two failure modes that will occur in production, and name the component card that covers each one.

---

**4.** Your system prompt is 800 tokens. Your max_tokens for output is 500. Your model has a 4,096-token context limit. A user pastes a 2,500-word document into the chat and asks a question about it. Will this call succeed? Show your calculation, and describe what will happen if it fails mid-conversation.

---

**5.** You deploy a chatbot with a system prompt that instructs the model to "respond in plain text, no markdown." Users report that the chatbot is responding with markdown headers and bullet points anyway. You haven't changed the system prompt. What are two plausible causes, and how would you investigate each?

---

**6.** A user of your chatbot says: "I know I asked about this in our last conversation, but I can't remember what you told me." Your current implementation stores session history in a database but deletes it when the session ends. The user wants continuity across sessions. Name the component card in this module that is most relevant, explain its limit, and describe what additional module you'd need to implement true cross-session memory.

---

## Answers

**1.**

Two component cards that could cause inconsistent answers across sessions:

**System prompt design** — if the system prompt is not stored in version control and is constructed dynamically (from a template, from a database, or from environment variables that differ between deployments), different sessions might receive different system prompts. A change to the prompt during the day means morning and afternoon sessions have different instructions, producing different responses to the same question.

**Conversation history management** — if any prior context is leaking into new sessions (e.g., the history array is not cleared between users, or a shared object is mutated), a session that starts with leftover context from a previous user will behave differently from a fresh session. The model's response to "How do I reset my password?" will differ based on what's already in the history.

Check: log the full messages array on both divergent calls and compare them. If the system prompts differ, the problem is prompt versioning. If prior turns are present in one but not the other, the problem is history isolation.

---

**2.**

For a long technical troubleshooting session, **summarization** is the right choice. The early turns in a technical session often contain crucial context — the user described the initial error, the system configuration, and what they've already tried. Dropping those turns (truncation or sliding window) risks losing the exact details the model needs to continue giving coherent advice.

**What you lose with truncation:** The oldest turns are removed outright. If the user described a specific error in turn 2 and the model has been referring to it, the model loses that reference. Responses may become generic or contradict earlier advice.

**What you lose with sliding window:** Same as truncation, but even more aggressively — a sliding window of the last N turns may drop the original problem description entirely if the session is long enough.

**What you lose with summarization:** Summarization costs an extra API call and introduces latency. The summary may not capture every technical detail accurately — the model summarizing the conversation might paraphrase a specific error code incorrectly. It also adds implementation complexity.

In practice: use summarization for troubleshooting and long-task sessions; use truncation or sliding window for casual, stateless Q&A where early turns are less critical. (See: **Context window pressure**)

---

**3.**

Two failure modes from a global history variable:

**Failure 1 — Conversation bleed across users:** In production, multiple users make requests concurrently. All of them append to the same `history` array. User A's messages appear in User B's conversation. User B's messages appear in User A's API call. Responses are incoherent and expose private conversation content across users. This is a **conversation history management** failure — each session must have its own isolated history object.

**Failure 2 — History lost on server restart:** If the server restarts (deployment, crash, auto-scaling scale-down), the in-memory `history` is wiped. Every user's conversation starts from scratch on their next message. If a user is mid-troubleshooting when the restart happens, they lose the entire context and get a response from the model that has no knowledge of what was discussed. This is also a **session lifecycle** failure — the history must be persisted to storage, not kept in memory.

In testing with one user on one process, neither failure appears. Both are guaranteed at scale.

---

**4.**

Calculation:
- System prompt: ~800 tokens (as stated)
- User's document: ~2,500 words × 1.3 ≈ 3,250 tokens
- User's question: ~20 tokens
- Overhead: ~10 tokens
- `max_tokens` (output): 500 tokens

Total: 800 + 3,250 + 20 + 10 + 500 = **4,580 tokens** — this exceeds the 4,096-token context limit.

The call will fail with `400: context_length_exceeded` before any generation occurs.

If this failure happens mid-conversation (not on the first turn), the user's message has already been appended to the history. The failed call means the message was "sent" from the user's perspective but never got a response. On the next attempt, the history has an unmatched user turn at the end. This doesn't cause a crash but leads to the model seeing a dangling user message that may cause confusion.

The correct fix is a token pre-check before appending to history (see: **Token budgeting** in Module 01). If the document exceeds your input budget, reject it with a clear message before appending it — not after the API call fails. (See: **Context window pressure**)

---

**5.**

Two plausible causes:

**Cause 1 — System prompt position:** The system prompt might not be the first message in the array. If a user message appears before the system message, many models deprioritize or partially ignore the system instructions. Check the messages array order — the system role must always be index 0. (See: **System prompt design**)

**Cause 2 — The model is selecting a format that matches the conversational context:** If users are asking questions that the model's training associates with markdown responses (e.g., "Can you give me a list of..." or "Explain the steps for..."), the model may override a weak format instruction. The fix is to make the instruction more emphatic and specific: "You MUST respond in plain text only. Never use markdown. Do not use asterisks, headers, or bullet symbols under any circumstances." Weaker instructions like "respond in plain text" are treated as suggestions, not rules.

To investigate: check the messages array server-side and confirm (a) system message is index 0, and (b) the system prompt text hasn't changed. If both are correct, the issue is instruction strength — tighten the wording.

---

**6.**

The most relevant component card is **session lifecycle**. In the current implementation, the session lifecycle ends with deletion of the history. Once a session ends, its context is gone. There's no mechanism to connect a new session to a previous one.

The limit of **session lifecycle** management alone: it gives you continuity within a session, not across sessions. Ending a session is designed to be a clean break.

What you'd need for true cross-session memory: **Module 4 — Memory**, specifically the **Long-term (external store) memory** component. That module covers persisting information about the user in an external store that survives session boundaries — so when a new session starts, relevant context from previous sessions can be retrieved and injected. This is architecturally different from conversation history: it's not the raw message array, but curated, relevant facts extracted from past conversations.

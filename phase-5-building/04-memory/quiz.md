# Quiz: Memory

These questions are scenario-based. The goal is to apply what you know, not recite it. Attempt each before reading the answer.

---

## Questions

**1.** Your assistant keeps contradicting itself within a single long session. In turn 3, the user says they prefer concise responses. By turn 25, the assistant is writing lengthy paragraphs again. No stored long-term memories are involved. Name the most likely cause using component cards from this module, and describe what a fix looks like.

---

**2.** A user asks your assistant to "forget everything I ever told you." Your current implementation: (a) clears the current session's `history` array in memory, and (b) stores a `cleared` flag in the database but keeps the memory records. The user starts a new session and notices the assistant still knows their name and preferences. What went wrong at each layer, and what does a correct implementation look like?

---

**3.** You're building a personal finance assistant. A user mentions their salary, account balances, and investment portfolio over several sessions. Your memory system extracts and stores these as long-term memories. Name two component cards from this module that are directly relevant to this scenario, and describe the specific risk each one raises.

---

**4.** After 6 sessions with your assistant, a user reports that it seems to know less about them than it did after session 2. They were expecting the assistant to get smarter over time. You look at the code and find that compression runs mid-session and compresses everything including any existing `[Earlier summary]` messages in the history. What is the specific failure here, and how do you fix it?

---

**5.** Your memory system retrieves the top 5 memories by semantic similarity. A user asks "what should I work on today?" The retrieved memories are all from 8 months ago, when they were working on a different project. Recent sessions from the last 2 weeks contain much more relevant context, but they score slightly lower on semantic similarity. Name the component card this falls under, explain why it's happening, and describe the fix.

---

**6.** A colleague argues that you should store the full conversation transcript as long-term memory instead of extracting facts. "More data is always better," they say. Under what conditions would you agree, and under what conditions would you push back? What are the concrete tradeoffs?

---

## Answers

**1.**

The most likely cause is a **short-term (in-context) memory** degradation from context window pressure. In a long session, once history exceeds the context limit, turns must be trimmed or compressed. If the trimming strategy removes the earliest turns first — including turn 3 where the user stated their preference — the model no longer has that instruction in its context. It reverts to default behavior (longer responses).

The fix has two parts: (a) in the system prompt, reinforce the preference as a standing instruction so it persists even after turn-by-turn history is trimmed; and (b) implement compression that preserves the preference instruction either in the system prompt or as a summary turn before trimming.

A standing system-prompt instruction like "The user has requested concise responses. Always be brief." survives history trimming because the system prompt is never trimmed. (See: **Short-term (in-context) memory** and **Memory compression and summarization**)

---

**2.**

Two layers of failure:

**Layer 1 — Clearing only session state:** Clearing `history` removes the in-memory conversation array, but this only affects the current session. It has no effect on the persistent memory store. The next session's read path calls `memory_store.search()` and retrieves the stored memories as if nothing happened.

**Layer 2 — Storing a flag instead of deleting:** Keeping the memory records and setting a `cleared` flag is the wrong pattern. If the flag check logic ever fails, or if the flag isn't checked consistently in the read path, the memories are accessible. Deletion must remove the records, not mark them.

A correct implementation:
```
def delete_user_memories(user_id):
  memory_store.delete_all(user_id=user_id)  # permanently delete records
  log_audit("memory_deletion", user_id=user_id)
  # Do NOT just set a flag — delete the rows/documents
```

After deletion, the next session's `memory_store.search()` returns nothing because there's nothing to find. (See: **Privacy and persistence tradeoffs**)

---

**3.**

Two relevant component cards:

**Privacy and persistence tradeoffs** — financial data (salary, account balances, portfolio) is sensitive personal financial information. The risk: storing it without consent or a deletion mechanism creates legal exposure. In many jurisdictions, financial data has specific handling requirements. The practical risk: if this data is leaked (via a security breach, an unauthorized query, or a bug that returns another user's memories), the damage is severe. Mitigation: don't store sensitive financial figures as plain text long-term memories. If you must store context, store categories ("User is focused on retirement savings") not values ("Balance: $47,000").

**Memory retrieval relevance** — financial figures change. A salary from 18 months ago, an account balance from last quarter, a portfolio value from before a market move — all of these become stale quickly. The risk: the assistant gives advice based on outdated figures without flagging that the data might be old. Mitigation: apply short TTLs (30 days) to financial facts, and include the date the fact was captured in the memory text so the model can reason about staleness. (See: **Privacy and persistence tradeoffs** and **Memory retrieval relevance**)

---

**4.**

The specific failure is summarizing a summary — the pattern called out in the **Memory compression and summarization** component card.

What's happening: in session 2, the history already contains `[Earlier summary]` from session 1. When the compression runs, it includes that summary message in `to_compress`. The model summarizes the summary. In session 3, the same thing happens again. By session 6, the "summary" is a distillation of a distillation of a distillation — most of the original detail is gone.

The fix: when building `to_compress`, exclude prior summary messages and start fresh from the raw turns that haven't been summarized yet. If the history contains only a prior summary and recent turns (no raw history left to compress), don't compress again.

```
# Separate raw turns from prior summaries
raw_turns   = [m for m in history[1:-recent] if not m.content.startswith("[Earlier")]
prior_summaries = [m for m in history[1:-recent] if m.content.startswith("[Earlier")]

if not raw_turns:
  return history  # nothing new to compress

summary = summarize(raw_turns)
# Keep prior summaries as-is; add the new summary alongside them
return [system] + prior_summaries + [{ role: "user", content: "[Earlier: " + summary + "]" }] + recent
```

(See: **Memory compression and summarization**)

---

**5.**

This falls under **Memory retrieval relevance**. The problem is over-weighting semantic similarity and under-weighting recency.

Why it's happening: semantic similarity alone measures how closely the stored memories match the topic of the current query. "What should I work on today?" is a general productivity question that matches old memories about past projects about as well as new ones — the general topic (work, tasks) is similar regardless of when the memory was created.

The fix is recency weighting in the retrieval score:
```
combined_score = (0.7 × semantic_similarity) + (0.3 × recency_score)
```

With recency weighting, memories from 2 weeks ago score significantly higher than memories from 8 months ago, even if the semantic similarity is slightly lower. The weights (0.7/0.3) are tunable — for a productivity assistant where current projects are heavily relevant, you might weight recency even more (0.5/0.5).

You should also consider storing session-level "active project" metadata separately and loading it unconditionally at session start, regardless of query similarity — some context is always relevant regardless of what the user is asking about today. (See: **Memory retrieval relevance**)

---

**6.**

**When you'd agree (full transcript):**

If the primary use case is detailed recall of past conversations — "What did we decide about X in our last meeting?" — then full transcripts preserve the nuance that fact extraction loses. Fact extraction is lossy: a model extracting facts might miss an important caveat, a specific number, or a conditional decision. For use cases that require verbatim recall, transcripts are the right choice.

**When you'd push back (fact extraction is better):**

For most personal assistant use cases, **long-term (external store) memory** — specifically discrete extracted facts — is superior to raw transcripts because:

1. **Retrieval precision**: Embedding a 2,000-token transcript produces a vector representing the average meaning of the whole conversation. It retrieves when the current query is broadly related to anything in the conversation. Discrete facts produce focused vectors — much higher retrieval precision.

2. **Token budget**: Injecting 5 extracted facts costs ~100 tokens. Injecting 5 transcript chunks from past sessions might cost 2,000–5,000 tokens, leaving little room for the current conversation.

3. **Staleness handling**: You can apply TTL to individual facts. You can't selectively expire individual statements from a transcript.

4. **Privacy**: Transcripts contain everything the user said, including things they may not want retained. Fact extraction gives you control over what is stored.

The honest answer: use fact extraction for personal context, and offer explicit transcript storage (user-initiated, labeled, deletable) for users who need verbatim recall. Don't store transcripts by default. (See: **Long-term (external store) memory**)

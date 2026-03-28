# Quiz: Agents

These questions are scenario-based. The goal is to apply what you know, not recite it. Attempt each before reading the answer.

---

## Questions

**1.** Your agent has been running for 3 minutes and hasn't returned a result. You look at the logs and see it has called the same tool 47 times with slightly different inputs — each search query is a minor variation of the previous one. Which component card describes this failure, what is the root cause, and what are two fixes?

---

**2.** Your agent is supposed to: (1) search the knowledge base, (2) draft a report, (3) request approval, (4) publish. Users report that sometimes the report gets published without going through approval. You look at the code and find that approval is enforced only by the system prompt instruction: "You must call request_approval before publish_report." Why is this insufficient, and what is the correct fix?

---

**3.** A colleague argues that human-in-the-loop checkpoints "defeat the purpose of agents." Write the strongest version of their argument. Then describe the specific conditions under which you would agree to remove the checkpoint, and the specific conditions under which adding a checkpoint would be non-negotiable.

---

**4.** You're reviewing an agent's execution trace and find this sequence:

```
Iteration 1: tool_call = search("refund policy")
Iteration 2: tool_call = search("refund policy")   ← identical to iteration 1
Iteration 3: tool_call = search("return policy")
Iteration 4: tool_call = summarize_result()         ← this tool doesn't exist
Iteration 5: error returned: "tool not found"
Iteration 6: tool_call = summarize_result()         ← same error
Iteration 7: MAX_ITERATIONS reached
```

Name the two failure modes visible in this trace, explain what caused each one, and describe the fix for each.

---

**5.** You're building an agent that can send emails on behalf of users. A user asks: "Send a follow-up email to everyone who attended last week's meeting." The agent searches the calendar for attendees, drafts an email, and is about to send it. The user is not present to approve. Describe what should happen at each step, and what the checkpoint should show the human reviewer.

---

**6.** Your agent returns a different answer each time a user asks the same question. The tool results are deterministic (the knowledge base hasn't changed). What is the most likely cause, and what can you do to make the agent more consistent?

---

**7.** An agent has access to three tools: `read_file`, `write_file`, and `delete_file`. A user asks: "Clean up my project folder — remove all the old backup files." The agent calls `delete_file` 15 times without asking for confirmation. The user didn't realize how many files would be deleted. Which two component cards cover this situation? For each, describe what should have been in place and why it wasn't sufficient.

---

## Answers

**1.**

The failure is described by the **Agent failure modes** component card, specifically the "infinite loop" pattern — the agent calls the same (or very similar) tools repeatedly without converging on an answer.

**Root cause:** The tool returns low-quality or sparse results. The model interprets this as "insufficient information" and retries with slightly varied queries. Without an explicit instruction to move on when results are inadequate, and without loop detection in the execution layer, the model continues refining queries indefinitely.

**Fix 1 — System prompt instruction:** "If you search more than 3 times and still cannot find sufficient information, proceed with what you have and note the gaps in your output." This gives the model an explicit exit condition for insufficient retrieval.

**Fix 2 — Execution-layer loop detection:** Track recent `(tool_name, input_hash)` pairs. When the model calls a tool with the same name and similar input as a recent call, return a structured message: "You have already tried this. Use a different query or proceed with available information." This catches the failure regardless of what the system prompt says. (See: **Agent failure modes**)

---

**2.**

The system prompt instruction is insufficient because models do not reliably follow procedural instructions under all conditions. The model may judge that the sequence is "obvious" and skip a step, or it may complete steps in a different order based on how it reasons about the task. System prompt instructions are guidance, not enforcement.

The correct fix is enforcement in the tool implementation itself:

```
elif tool_name == "publish_report":
  if not approval_store.is_approved(user_id, "publish_report"):
    return { status: "blocked", message: "Approval required. Call request_approval first." }
  external_system.publish(...)
```

When `publish_report` is called without prior approval, the tool blocks the action and returns a structured error. The model receives the error and is forced to call `request_approval` before trying again. This works even if the model never read, forgot, or ignored the system prompt instruction.

The principle: human-in-the-loop requirements must be enforced in code, not trusted to model compliance. (See: **Human-in-the-loop checkpoints**)

---

**3.**

**The strongest version of the argument:**

"Agents exist to automate tasks that humans would otherwise do manually. Every checkpoint reintroduces the human bottleneck you were trying to eliminate. If you require approval before every significant action, the agent is just a more expensive way to draft things for humans to rubber-stamp. The latency of waiting for human approval may be worse than the latency of doing the task manually. An agent that can't act autonomously on non-trivial decisions isn't an agent — it's an autocomplete feature."

**When you would agree (remove the checkpoint):**

- The action is fully reversible. If you can undo it in under 30 seconds with no external consequences, autonomous execution is reasonable.
- The domain is low-stakes and bounded. Formatting a document, generating a draft, reorganizing files — actions where a mistake is inconvenient but not damaging.
- You have extensive observability. If you can audit every action and rollback reliably, autonomous execution is acceptable because you can catch and fix mistakes quickly.

**When a checkpoint is non-negotiable:**

- The action is irreversible or difficult to reverse (send email, charge payment, delete data, post publicly, notify external parties).
- The action affects people outside the conversation (sending to third parties, modifying shared systems).
- The downstream consequences are high (financial transactions, legal documents, anything that triggers external workflows).
- The domain is regulated (healthcare, legal, financial services).

The heuristic: if a human doing this task would pause and double-check before proceeding, the agent should too. (See: **Human-in-the-loop checkpoints**)

---

**4.**

Two failure modes visible in the trace:

**Failure 1 — Repeated tool call (loop pattern):** Iteration 2 is identical to iteration 1. The model called `search("refund policy")` twice in a row. This is the early stage of the loop failure mode described in **Agent failure modes**. The root cause: the search returned sparse or unsatisfying results, and the model retried with the same query. Fix: loop detection that intercepts repeat calls and redirects the model, plus a system prompt instruction to vary queries or move on after N attempts.

**Failure 2 — Hallucinated tool call:** `summarize_result()` doesn't exist in the tools array. The model generated a plausible-sounding tool name, received an error, and called it again on the next iteration — repeating the same wrong tool call. This is the "hallucinated tool call" failure mode from **Agent failure modes**. The root cause: the model extrapolated a tool name from its general knowledge rather than from the provided tool definitions. Fix: the execution layer checks tool names against `VALID_TOOLS` before executing, returns a structured error with the list of valid tool names, and the model should self-correct on the next iteration. Iterating on a tool that doesn't exist twice in a row suggests the error message wasn't clear enough or didn't include the valid names. (See: **Tool selection by the model** and **Agent failure modes**)

---

**5.**

What should happen at each step:

1. **Search calendar** — no checkpoint needed. Reading is non-destructive and reversible.
2. **Draft email** — no checkpoint needed. A draft in memory has no external effect.
3. **Request approval (checkpoint)** — before sending any external communication, the agent calls `request_approval`. The loop pauses. The human reviewer is notified.

**What the checkpoint should show the reviewer:**

```
Action: Send follow-up email
To: 12 recipients (list all names and email addresses)
Subject: [the draft subject line]
Body: [the full draft email text]
Sent from: [user's email address]

This action will send 12 emails immediately. It cannot be undone once sent.
[Approve] [Reject] [Edit]
```

The reviewer must see the full list of recipients (not just "12 people"), the complete email content, and a clear statement that the action is irreversible. An "Edit" option reduces the friction of approving slightly-wrong drafts.

4. **Send email** — only after explicit approval. If rejected, the agent returns to the user with the rejection reason.

Without the checkpoint at step 3, the agent sends 12 emails on behalf of the user based on a query that contained ambiguity ("everyone who attended" — does that include the organizer? people who RSVPed but didn't attend?). The user discovers the mistake only after recipients respond. (See: **Human-in-the-loop checkpoints**)

---

**6.**

The most likely cause is temperature. If the agent's calls use a temperature above 0, the model's reasoning about which tool to call and how to construct responses will vary between runs even with identical inputs. Different tool selection leads to different tool results, which leads to different final answers — even though the tools themselves are deterministic.

What you can do to make it more consistent:

1. **Lower temperature** to 0 or 0.1 for the agent's model calls. This makes tool selection more deterministic.

2. **More specific tool descriptions** — if two tools have overlapping descriptions, the model chooses between them with noise at higher temperatures. Sharper differentiation reduces variance regardless of temperature.

3. **Constrain the system prompt** — specify the exact sequence of steps the agent should follow. "Step 1: call X. Step 2: call Y only if X returns Z." Structured step instructions reduce the number of decisions left to probabilistic sampling.

Note: temperature 0 is not bit-for-bit identical (see: Module 1 — Request anatomy). For absolute consistency, you need to cache responses for identical tool inputs. But for practical purposes, temperature 0 with good tool descriptions gets you very close to consistent behavior. (See: **Tool selection by the model**)

---

**7.**

Two relevant component cards:

**Human-in-the-loop checkpoints** — the agent should have called `request_approval` before executing bulk delete operations. "Remove all backup files" is an irreversible action with unclear scope. The user said "backup files" but may not have known how many there were. A checkpoint before the first `delete_file` call — showing the full list of files to be deleted — would have given the user the opportunity to review the scope before any deletion occurred.

What should have been in place: the `delete_file` tool (or the agent system prompt) should have required approval before any delete operation, or at minimum before bulk operations exceeding 1 file. What wasn't sufficient: the absence of any checkpoint at all. The agent interpreted "clean up" as "proceed without confirmation" because nothing in the tool definition or system prompt required it to pause.

**Tool definition** — the `delete_file` tool description should have included a warning about irreversibility and a trigger for approval: "Deletes a file permanently. This action cannot be undone. You MUST call `request_approval` before calling this tool if more than 1 file will be deleted, or if the files have not been explicitly confirmed by the user." A vague description ("Deletes a file") provides no guidance on when caution is required.

What wasn't sufficient: a tool description that described the action but not its consequences or the conditions under which approval is required. The model's default behavior is to use tools that match the task — it needs explicit instructions to know when to pause. (See: **Tool definition** and **Human-in-the-loop checkpoints**)

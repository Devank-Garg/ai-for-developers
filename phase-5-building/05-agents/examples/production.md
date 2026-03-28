# Production Example: Agents

> Same scenario as `basic.md` — a research and summary agent — now hardened
> for real use. Failure modes are injected deliberately; read the annotations.

---

## What changes from the basic example

- **Hallucinated tool call interception** — unknown tool names are caught and returned as errors
- **Structured error returns** — tool failures return data, not exceptions that crash the loop
- **Loop detection** — repeated identical calls are caught and interrupted
- **Human-in-the-loop checkpoint** — approval required before writing to external systems
- **Iteration limit with partial result** — agent summarizes what it found before terminating

---

## The full example

```
MAX_ITERATIONS = 10
recent_tool_calls = []   # tracks (tool_name, input_hash) for loop detection

VALID_TOOLS = {
  "search_knowledge_base",
  "write_summary",
  "publish_report",          # new: writes to an external system
  "request_approval"         # checkpoint tool
}

def run_research_agent(topic, user_id):
  messages = [
    {
      role: "system",
      content: """You are a research assistant. Your task:
1. Search the knowledge base to gather information about the given topic
2. Search at least twice with different queries
3. Write a structured summary using write_summary
4. Before publishing with publish_report, you MUST call request_approval first

If you cannot complete a step, explain what you found and what blocked you."""
    },
    { role: "user", content: "Research this topic: " + topic }
  ]

  for iteration in range(MAX_ITERATIONS):
    response = model.chat({
      model: "claude-sonnet-4-6",
      messages: messages,
      tools: build_tool_list(),
      max_tokens: 1000
    })

    if response.stop_reason == "end_turn":
      return { status: "complete", result: response.content, iterations: iteration + 1 }

    elif response.stop_reason == "tool_use":
      tool_call = response.tool_use

      # --- HALLUCINATED TOOL NAME CHECK ---
      # WHY: the model occasionally generates a tool name that doesn't exist.
      # Without this check, the execution layer crashes with an unhandled error.
      if tool_call.name not in VALID_TOOLS:
        tool_result = {
          status: "error",
          message: "Tool '" + tool_call.name + "' does not exist. "
                   "Available tools: " + list(VALID_TOOLS)
        }
      else:
        # --- LOOP DETECTION ---
        # WHY: detect when the model is calling the same tool with the same
        # input repeatedly — the hallmark of an infinite loop.
        call_signature = (tool_call.name, hash(str(tool_call.input)))
        if call_signature in recent_tool_calls:
          tool_result = {
            status: "loop_detected",
            message: "You have already called this tool with this input. "
                     "Do not repeat it. Either use a different query, "
                     "or proceed to write_summary with the information gathered so far."
          }
        else:
          recent_tool_calls.append(call_signature)
          # Keep only the last 5 to avoid false positives across long runs
          recent_tool_calls = recent_tool_calls[-5:]

          # --- EXECUTE TOOL WITH STRUCTURED ERROR RETURN ---
          tool_result = execute_tool_safe(tool_call.name, tool_call.input, user_id)

      # --- APPEND AND CONTINUE ---
      # WHY: always append model's turn first, then the tool result.
      messages.append({ role: "assistant", content: response.content })
      messages.append({ role: "tool", content: str(tool_result), tool_use_id: tool_call.id })

      log_info("agent_iteration", iteration=iteration, tool=tool_call.name, status=tool_result.get("status"))

  # --- ITERATION LIMIT HIT ---
  # WHY: don't just raise an error — ask the model to summarize what it found.
  # A partial result is more useful than an error message.
  messages.append({ role: "user", content:
    "You have reached the maximum number of steps. "
    "Please summarize everything you have gathered so far, even if incomplete."
  })
  final = model.chat({ model: "claude-sonnet-4-6", messages: messages, max_tokens: 500 })
  return { status: "partial", result: final.content, iterations: MAX_ITERATIONS }


def execute_tool_safe(tool_name, tool_input, user_id):
  """All tool failures return structured data — no exceptions propagate to the loop."""
  try:
    if tool_name == "search_knowledge_base":
      results = knowledge_base.search(tool_input.query, top_k=3)
      if not results:
        return { status: "empty", message: "No results found for: " + tool_input.query }
      return { status: "success", results: format_results(results) }

    elif tool_name == "write_summary":
      # This is a local operation — no approval needed
      summary_id = summary_store.save(tool_input.title, tool_input.content)
      return { status: "success", summary_id: summary_id }

    elif tool_name == "publish_report":
      # WHY: this writes to an external system — verify the model called
      # request_approval before this call. If not, block it.
      if not approval_store.is_approved(user_id, "publish_report"):
        return {
          status: "blocked",
          message: "Approval required before publishing. Call request_approval first."
        }
      report_id = external_system.publish(tool_input)
      return { status: "success", report_id: report_id }

    elif tool_name == "request_approval":
      # Pause and wait for human review
      approval_id = approval_store.create(
        user_id=user_id,
        action=tool_input.action_description,
        details=tool_input.action_details
      )
      notify_reviewer(approval_id)
      approved = wait_for_approval(approval_id, timeout_seconds=300)
      approval_store.record(user_id, "publish_report", approved)

      if approved:
        return { status: "approved", message: "Approved. You may now call publish_report." }
      else:
        return { status: "rejected", message: "Rejected. Do not publish this report." }

  except ExternalServiceError as e:
    # WHY: return as data, not exception — the model can reason about errors
    return { status: "error", message: "Service error: " + e.message + ". Try again or use an alternative approach." }

  except Exception as e:
    log_error("unexpected_tool_error", tool=tool_name, error=e)
    return { status: "error", message: "Unexpected error in " + tool_name + ". Try a different approach." }
```

---

## Failure mode walkthrough

### Failure 1: Agent loops on the same search query 47 times

**What happens:** The model searches for "climate change agriculture" and gets sparse results. It searches again with the same query. And again. The loop runs until `MAX_ITERATIONS` is hit, consuming budget and producing nothing.

**Why it happens:** The model received low-quality results but was not given guidance on how to proceed. Without instructions to vary the query or move on, it retries the same approach indefinitely.

**How the example handles it:** Loop detection checks `call_signature` before executing. On the second identical call, the model receives a structured message: "You have already called this tool with this input. Do not repeat it." This redirects the model to either vary the query or proceed with what it has. The `MAX_ITERATIONS` guard is the final backstop.

**What would break without this:** The agent burns the entire iteration budget on one failing query. The user waits minutes and receives either an error or a partial result with no explanation.

---

### Failure 2: Model calls `publish_report` without approval

**What happens:** The model gathers good research, writes a summary, and decides to publish it directly — skipping the `request_approval` step despite the system prompt instruction.

**Why it happens:** Models don't always follow procedural instructions reliably, especially when they have the information needed to proceed and the task seems complete. The system prompt says "you must call request_approval first" but the model may judge that the instruction doesn't apply to its current situation.

**How the example handles it:** `execute_tool_safe` checks `approval_store.is_approved()` before executing `publish_report`. Regardless of whether the model followed the approval instruction, the tool itself enforces the requirement. The model receives a `blocked` status and instructions to call `request_approval` first.

**What would break without this:** An unapproved report is published to an external system. Depending on the content, this could be a minor inconvenience or a significant incident (publishing confidential research, an incorrect report, or a draft marked "do not distribute").

---

### Failure 3: Hallucinated tool name crashes the execution layer

**What happens:** The model generates `tool_call.name = "summarize_findings"` — a name it invented that doesn't exist in the tools array. The execution layer does a dictionary lookup: `tools["summarize_findings"]` and raises `KeyError`. The loop terminates with an unhandled exception.

**Why it happens:** Tool names that sound plausible can be generated by the model, especially if tool names are similar to common concepts. The model knows what `summarize` means and may generate a plausible-sounding variant.

**How the example handles it:** The `VALID_TOOLS` check at the top of the tool execution block catches unknown names before any lookup is attempted. The model receives a structured error listing the valid tool names, which gives it the information needed to self-correct on the next iteration.

**What would break without this:** The agent crashes with a stack trace. The user sees a 500 error. The partial work done in prior iterations is lost or requires manual inspection of the logs to recover.

---

## Production checklist for this module

- [ ] Validate tool names against the known set before execution — never assume the model sends a valid name
- [ ] Return all tool errors as structured data, not exceptions — the model needs to reason about failures
- [ ] Detect repeated identical tool calls and redirect the model, not just count iterations
- [ ] Set MAX_ITERATIONS and return a partial result at the limit — never return just an error
- [ ] Enforce human approval in the tool implementation, not only in the system prompt instruction
- [ ] Log every iteration with tool name, inputs, and result status — agents are impossible to debug without execution traces

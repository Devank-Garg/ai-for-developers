# Concepts: Agents

> **Tag: core** — An agent is a loop where the model decides what to do next, does it, observes the result, and decides again. Everything that makes this safe and reliable is your responsibility to build.

---

## Tool Definition

**What it is** — A structured description of a capability you give to the model: its name, what it does, and the parameters it accepts.

**Why it matters** — The model can only call tools it knows about. Without a well-defined tool specification, the model either ignores the capability or hallucinates a tool call with the wrong name or parameters.

**How it works** — Tools are defined as a schema attached to the API request. Each tool has a name, description, and parameter schema. The model reads these definitions and decides which tool to call based on the task.

```
tools = [
  {
    name: "search_documentation",
    description: "Search the product documentation for information about a topic. Use this when the user asks a question that requires looking up specific product behavior.",
    parameters: {
      query: {
        type: "string",
        description: "The search query. Be specific.",
        required: true
      },
      max_results: {
        type: "integer",
        description: "Maximum number of results to return. Default 5.",
        required: false
      }
    }
  },
  {
    name: "create_support_ticket",
    description: "Create a support ticket in the ticketing system. Use this only when the user's issue cannot be resolved through documentation and requires human support.",
    parameters: {
      summary: { type: "string", required: true },
      priority: { type: "string", enum: ["low", "medium", "high"], required: true }
    }
  }
]

response = model.chat({
  model: "claude-sonnet-4-6",
  messages: messages,
  tools: tools
})
```

The description is the model's only guide for when and how to use a tool. Vague descriptions produce unreliable tool selection. Specific descriptions ("Use this when X, not when Y") produce consistent behavior.

**Production tip** — Write tool descriptions as if you're explaining them to a new team member, not documenting them for a computer. Include when to use the tool, when NOT to use it, and any side effects (e.g., "This creates a real ticket and notifies the support team").

**Common failure** — Tools with similar names or descriptions cause the model to choose the wrong one. If you have `search_docs` and `search_tickets`, the model will confuse them on edge cases. Differentiate descriptions explicitly: "search_docs searches product documentation only. Do not use it for ticket history."

---

## Tool Selection by the Model

**What it is** — The model's decision about which tool to call, what parameters to pass, and whether any tool call is needed at all.

**Why it matters** — The model makes this decision based solely on the task and the tool descriptions. Incorrect tool selection — calling the wrong tool, passing wrong parameters, or calling a tool when none is needed — cascades through the rest of the execution loop.

**How it works** — When the model decides to use a tool, it returns a tool call in its response instead of (or alongside) a text response. You execute the tool, pass the result back, and the model continues.

```
response = model.chat({ messages: messages, tools: tools })

if response.stop_reason == "tool_use":
  # Model wants to call a tool
  tool_call = response.tool_use
  tool_name = tool_call.name        # e.g., "search_documentation"
  tool_input = tool_call.input      # e.g., { query: "how to reset password" }

  # You validate and execute the tool
  result = execute_tool(tool_name, tool_input)

  # Add the tool call and result to the messages array
  messages.append({ role: "assistant", content: response.content })
  messages.append({ role: "tool",      content: result, tool_use_id: tool_call.id })

elif response.stop_reason == "end_turn":
  # Model is done — no more tool calls needed
  return response.content
```

The model does not execute tools. It only requests them. You run the tool and return the result. The model then decides whether to call another tool or generate a final answer.

**Production tip** — Validate tool inputs before execution. The model may pass a parameter that is technically valid JSON but semantically incorrect (a negative number where only positives make sense, an empty string where content is required). Validate at the application layer before calling the underlying system.

**Common failure** — The model calls a tool that doesn't exist — it generates a tool name that isn't in the `tools` array. This happens when tool names are similar to common concepts the model knows from training. Your execution layer must handle unknown tool names explicitly: check the name before calling anything, and return a descriptive error to the model if the tool doesn't exist.

---

## Execution Loop (Observe → Think → Act)

**What it is** — The repeating cycle that drives an agent: the model observes its current state (messages + tool results), thinks about what to do next (generates a response or tool call), and acts (executes a tool or returns an answer).

**Why it matters** — Without a well-structured execution loop, an agent either stops too early (exits before completing the task), runs forever (no termination condition), or loses track of state (tool results aren't properly added to the context).

**How it works** — The loop runs until the model returns a final answer or the maximum iteration limit is reached:

```
MAX_ITERATIONS = 10

def run_agent(task):
  messages = [
    { role: "system", content: agent_system_prompt },
    { role: "user",   content: task }
  ]

  for iteration in range(MAX_ITERATIONS):
    response = model.chat({ messages: messages, tools: tools })

    if response.stop_reason == "end_turn":
      # Model is done
      return response.content

    elif response.stop_reason == "tool_use":
      # Execute the requested tool
      tool_call = response.tool_use
      tool_result = execute_tool(tool_call.name, tool_call.input)

      # Add both the model's turn and the tool result to messages
      # WHY: the model needs to see its own tool call in the history
      # to make sense of the result
      messages.append({ role: "assistant", content: response.content })
      messages.append({ role: "tool", content: tool_result, tool_use_id: tool_call.id })

      # Continue the loop — model will see the result and decide next step

  # WHY: if we hit MAX_ITERATIONS without an end_turn, the agent is stuck.
  # Return an error, not silence.
  raise AgentLoopError("Agent did not complete within " + MAX_ITERATIONS + " iterations")
```

Each iteration: the model reads the full message history (including all prior tool calls and results), generates the next action, you execute it, and the result is appended before the next iteration.

**Production tip** — Log every iteration with the tool call, input, and result. Agent behavior is hard to debug without this trace. When an agent produces a wrong answer, the trace tells you exactly where it went wrong.

**Common failure** — Not appending the model's own tool_call response to the messages array before adding the tool result. The next iteration's model call contains a tool result with no corresponding tool request in the history, which confuses the model and often causes it to generate another (duplicate) tool call.

---

## Error Recovery in Multi-Step Chains

**What it is** — The strategy for handling failures that occur mid-execution in a multi-tool, multi-step agent task.

**Why it matters** — A single tool failure in a multi-step chain doesn't necessarily mean the task is impossible. An agent with good error recovery can retry a step, try an alternative approach, or gracefully report what failed and why — instead of crashing or silently producing a wrong answer.

**How it works** — Error recovery operates at two levels:

**Tool-level recovery** — handle errors from individual tool calls:
```
def execute_tool_with_recovery(tool_name, tool_input):
  try:
    result = tools[tool_name](tool_input)
    return { status: "success", result: result }
  except ToolNotFoundError:
    return { status: "error", message: "Tool '" + tool_name + "' does not exist." }
  except ValidationError as e:
    return { status: "error", message: "Invalid input: " + e.message }
  except ExternalServiceError as e:
    return { status: "error", message: "External service failed: " + e.message + ". Try an alternative approach." }
```

Return errors as structured data back to the model — don't raise exceptions that terminate the loop. The model reads the error and can decide to retry, try a different tool, or escalate.

**Agent-level recovery** — limit retries and handle unrecoverable states:
```
retry_counts = {}

def execute_with_retry_limit(tool_name, tool_input, max_retries=2):
  key = tool_name + str(tool_input)
  retry_counts[key] = retry_counts.get(key, 0) + 1

  if retry_counts[key] > max_retries:
    return { status: "fatal_error", message: "Tool " + tool_name + " failed after " + max_retries + " retries. Cannot complete this step." }

  return execute_tool_with_recovery(tool_name, tool_input)
```

**Production tip** — Always return error messages in the tool result, not as exceptions. When a tool fails, the model gets the error as context and can reason about it ("the search returned no results — I'll try a broader query"). An exception that crashes the loop gives the model no opportunity to adapt.

**Common failure** — An agent that retries the same tool call with identical inputs after an error. If a tool fails on input X, calling it again with X will fail again. The model must be instructed (in the system prompt or via the error message) to modify its approach on retry — not just repeat.

---

## Human-in-the-Loop Checkpoints

**What it is** — Explicit pauses in the agent execution loop where a human must review and approve before the agent proceeds.

**Why it matters** — Agents that can take irreversible actions (sending emails, modifying databases, charging accounts, deleting files) should not execute those actions autonomously without confirmation. A single wrong parameter in a tool call can cause damage that is expensive or impossible to reverse.

**How it works** — Checkpoints are implemented as special tool calls that pause the loop:

```
tools = [
  ...,
  {
    name: "request_approval",
    description: "Request human approval before taking an irreversible action. "
                 "Use this before sending emails, modifying production data, "
                 "charging customers, or deleting records.",
    parameters: {
      action_description: { type: "string", required: true },
      action_details:     { type: "string", required: true }
    }
  }
]

def execute_tool(tool_name, tool_input):
  if tool_name == "request_approval":
    # Pause the loop — notify the human and wait for response
    approval_id = create_approval_request(
      description = tool_input.action_description,
      details     = tool_input.action_details
    )
    notify_human(approval_id)
    approved = wait_for_human_response(approval_id, timeout=300)

    if approved:
      return { status: "approved", message: "Human approved. Proceed." }
    else:
      return { status: "rejected", message: "Human rejected. Do not proceed with this action." }
```

The agent requests approval; the human reviews the description; the loop continues or halts based on the decision.

**Production tip** — Define which tools require approval in your system prompt, not by hardcoding checks in each tool. "Before calling `send_email`, `delete_record`, or `process_payment`, you MUST call `request_approval` first." This is more maintainable and the model enforces it across its own decisions.

**Common failure** — Implementing checkpoints only for the final action but not for intermediate steps that are also irreversible. An agent that approves "send the summary email" but doesn't checkpoint "update the database with the calculated totals" (which happens earlier and is also irreversible) has a false sense of safety.

---

## Agent Failure Modes

**What it is** — The characteristic ways that agents go wrong: infinite loops, hallucinated tool calls, goal drift, and task abandonment.

**Why it matters** — Agent failures are qualitatively different from chatbot failures. A chatbot gives a wrong answer; an agent can take multiple wrong actions in sequence, each compounding the error, sometimes against external systems. The damage is real, not just conversational.

**How it works** — The four main failure modes:

**Infinite loop** — the agent calls the same (or similar) tools repeatedly without converging:
```
# Detection:
if same_tool_called_with_similar_input(history, current_call, threshold=0.9):
  return { status: "loop_detected", message: "Stopping: same action repeated. Summarize what you've found so far." }
```

**Hallucinated tool call** — the model invokes a tool that doesn't exist:
```
# Defense:
valid_tool_names = {t.name for t in tools}
if tool_call.name not in valid_tool_names:
  return { status: "error", message: "Tool '" + tool_call.name + "' does not exist. Available tools: " + list(valid_tool_names) }
```

**Goal drift** — the agent pursues a sub-goal that diverges from the original task:
```
# Mitigation: include the original task in every iteration's system prompt
# so the model can re-anchor to the goal if it drifts.
```

**Task abandonment** — the agent gives up or returns a partial result without communicating clearly:
```
# Mitigation: in the system prompt: "If you cannot complete the task,
# explicitly state what you were able to do, what you were not able to do,
# and why."
```

The `MAX_ITERATIONS` guard (see: Execution loop) is the backstop for all loop-based failures. It ensures no agent runs indefinitely regardless of the failure mode.

**Production tip** — Test your agent against adversarial tasks: tasks that are ambiguous, tasks that require a tool that always fails, and tasks with no valid solution. Observing failure behavior before production is the only way to know what your agent will do when real users hit these cases.

**Common failure** — Relying solely on `MAX_ITERATIONS` as the only safety mechanism. MAX_ITERATIONS stops the loop but doesn't return a useful result. A loop-detected agent should generate a partial summary of what it accomplished before terminating — not just throw an error.

---

## Further reading

**ReAct: Reasoning + Acting** — [https://arxiv.org/abs/2210.03629](https://arxiv.org/abs/2210.03629)
The paper that formalized the observe-think-act loop. Short and readable. The pseudocode in this module's execution loop card is a direct implementation of ReAct.

**Anthropic's guide to tool use** — [https://docs.anthropic.com/en/docs/build-with-claude/tool-use](https://docs.anthropic.com/en/docs/build-with-claude/tool-use)
Provider-specific but the concepts (tool schemas, tool_use responses, tool results) map to the patterns in this module.

**Prompt injection in agents** — [https://simonwillison.net/2023/Apr/14/worst-that-could-happen/](https://simonwillison.net/2023/Apr/14/worst-that-could-happen/)
Simon Willison's analysis of prompt injection risks in agentic systems. Required reading before shipping any agent that processes external content.

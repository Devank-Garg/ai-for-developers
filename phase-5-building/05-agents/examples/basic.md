# Basic Example: Agents

> Pseudocode walkthrough — no real SDK imports. The goal is to understand
> one complete observe → think → act cycle in an agent execution loop.

---

## Scenario

A research agent that helps users gather information from a knowledge base and produce a summary. The user gives the agent a research topic; the agent searches, reads results, and writes a summary — without the user needing to run each step manually.

---

## Step 1: Define the tools

```
tools = [
  {
    name: "search_knowledge_base",
    # WHY: the description tells the model when to use this tool.
    # "Use this when you need to find information" is vague.
    # This description specifies the input type and what the tool returns.
    description: "Search the internal knowledge base for documents related to a topic. "
                 "Returns a list of document excerpts. Use this to gather information "
                 "before writing a summary. Do not use it for general web search.",
    parameters: {
      query: {
        type: "string",
        description: "The search query. Be specific and use relevant terminology.",
        required: true
      }
    }
  },
  {
    name: "write_summary",
    description: "Write a structured summary of findings. "
                 "Use this only after you have gathered sufficient information "
                 "from the knowledge base. This produces the final output.",
    parameters: {
      title:   { type: "string", required: true },
      content: { type: "string", required: true }
    }
  }
]
```

---

## Step 2: Initialize the agent loop

```
def run_research_agent(topic):
  messages = [
    {
      role: "system",
      content: "You are a research assistant. Your task: search the knowledge base "
               "to gather information about the given topic, then write a structured summary. "
               "Search at least twice with different queries before writing the summary."
    },
    { role: "user", content: "Research this topic: " + topic }
  ]

  MAX_ITERATIONS = 10  # WHY: prevents the agent from running indefinitely

  for iteration in range(MAX_ITERATIONS):
    response = model.chat({
      model: "claude-sonnet-4-6",
      messages: messages,
      tools: tools,
      max_tokens: 1000
    })
```

---

## Step 3: Handle the model's response

```
    if response.stop_reason == "end_turn":
      # WHY: the model finished without a tool call — it either gave a final
      # answer in text or couldn't proceed. Return what we have.
      return response.content

    elif response.stop_reason == "tool_use":
      tool_call = response.tool_use
      tool_name = tool_call.name
      tool_input = tool_call.input
```

---

## Step 4: Execute the tool

```
      if tool_name == "search_knowledge_base":
        # Run the actual search
        results = knowledge_base.search(tool_input.query, top_k=3)
        tool_result = format_results(results)  # returns a readable string

      elif tool_name == "write_summary":
        # Write and return the summary — this is the terminal action
        summary = {
          title:   tool_input.title,
          content: tool_input.content
        }
        # WHY: append the tool result before returning,
        # so the loop can handle any follow-up if needed
        tool_result = "Summary written successfully."
```

---

## Step 5: Append tool call and result, continue the loop

```
      # WHY: append the model's response (which contains the tool call)
      # BEFORE appending the tool result.
      # The model needs to see its own request to make sense of the result.
      messages.append({
        role: "assistant",
        content: response.content  # includes the tool call
      })
      messages.append({
        role: "tool",
        content: tool_result,
        tool_use_id: tool_call.id  # links this result to the tool call above
      })

      # The loop continues — the model reads the result and decides next step

  # If we exit the loop without returning:
  return "Agent did not complete within " + MAX_ITERATIONS + " iterations."
```

---

## What a full run looks like

```
# Iteration 1:
#   Model → calls search_knowledge_base(query="climate change impact agriculture")
#   App   → runs search, returns 3 excerpts
#   Messages array now has: system, user, assistant(tool_call), tool(results)

# Iteration 2:
#   Model reads the results, decides to search again with a different angle
#   Model → calls search_knowledge_base(query="food security drought adaptation")
#   App   → runs search, returns 3 more excerpts

# Iteration 3:
#   Model has enough information
#   Model → calls write_summary(title="...", content="...")
#   App   → records summary

# Iteration 4:
#   Model → stop_reason = "end_turn" (no more tool calls needed)
#   App   → returns final response
```

---

## What this example shows

This walkthrough demonstrates three of the six component cards from `concepts.md`:

- **Tool definition** — two tools with specific, differentiated descriptions
- **Tool selection by the model** — the model reads descriptions to decide when to search vs. summarize
- **Execution loop** — the observe → think → act cycle with correct message appending and MAX_ITERATIONS guard

It deliberately leaves out:

- **Error recovery** — if `knowledge_base.search()` throws an exception, this loop crashes. See `production.md` for structured error returns.
- **Human-in-the-loop** — this agent has no approval checkpoint. See `production.md`.
- **Agent failure modes** — no loop detection, no hallucinated tool name handling. See `production.md`.

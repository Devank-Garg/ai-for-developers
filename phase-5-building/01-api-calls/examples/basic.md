# Basic Example: API Calls

> Pseudocode walkthrough — no real SDK imports. The goal is to understand
> the structure of an LLM call, not to copy runnable code.

---

## Scenario

A developer needs to summarize user-submitted support tickets. Each summary is a one-off call: send the ticket text, get back a short summary, display it.

---

## Step 1: Build the messages array

```
ticket_text = get_user_input()  # the text to summarize

messages = [
  {
    role: "system",
    # WHY: The system prompt sets the model's behavior for this entire call.
    # Keep it short and specific — vague instructions produce vague behavior.
    content: "You are a support ticket summarizer. Summarize in 2-3 sentences.
              Focus on: the reported issue, its impact, and any steps already tried."
  },
  {
    role: "user",
    # WHY: The user message carries the actual content to process.
    # We label it so the model knows what role this content plays.
    content: "Please summarize this ticket:\n\n" + ticket_text
  }
]
```

Notice: the system message comes first. The model treats it as standing instructions for the call. The user message is the specific task.

---

## Step 2: Choose parameters

```
params = {
  model: "claude-sonnet-4-6",  # or whichever model you're using

  # WHY: max_tokens caps the output length.
  # A summary doesn't need more than 150 tokens; setting this prevents
  # the model from writing an essay when you asked for a summary.
  max_tokens: 150,

  # WHY: temperature 0.2 keeps summaries factual and consistent.
  # Higher temperature would make the same ticket produce different summaries
  # on repeated calls, which is undesirable for a support tool.
  temperature: 0.2
}
```

---

## Step 3: Send the request

```
request = { messages: messages, ...params }

# WHY: model.chat() is the blocking call — it waits for the full response.
# For a background summarization job, this is fine.
# For a user-facing interface, you'd switch to model.stream() in Step 4.
response = model.chat(request)
```

---

## Step 4: Read the response

```
summary = response.content

# WHY: The response always includes token usage. Log it now —
# you'll need this data to understand costs and catch outliers.
input_tokens  = response.usage.input_tokens
output_tokens = response.usage.output_tokens

log("tokens used: input=" + input_tokens + " output=" + output_tokens)

display(summary)
```

---

## What this example shows

This walkthrough demonstrates three of the five component cards from `concepts.md`:

- **Request anatomy** — building the messages array with correct roles and parameters
- **Token budgeting** — setting `max_tokens` to prevent over-generation and logging usage
- **Streaming vs. batch** — using batch mode (`model.chat()`) for a background job

It deliberately leaves out:

- **Error handling and retries** — shown in `production.md`. This example has no retry logic; a rate limit or server error would crash it.
- **Cost awareness** — partially shown via the usage log, but no routing or spend cap. See `production.md` for the full pattern.

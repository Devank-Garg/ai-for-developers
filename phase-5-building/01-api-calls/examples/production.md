# Production Example: API Calls

> Same scenario as `basic.md` — summarizing support tickets — now hardened
> for real use. Failure modes are injected deliberately; read the annotations.

---

## What changes from the basic example

- **Token pre-check** before sending — rejects inputs that would overflow the context
- **Exponential backoff with jitter** on rate limit and server errors
- **Streaming** instead of batch — first token appears in ~100ms instead of ~10 seconds
- **Per-call cost logging** using actual token counts from the response
- **Model routing** — short tickets use a smaller, cheaper model

---

## The full example

```
MAX_INPUT_TOKENS  = 3000  # leave room for output within the context limit
MAX_OUTPUT_TOKENS = 200
CONTEXT_LIMIT     = 4096
BASE_RETRY_DELAY  = 1     # seconds
MAX_RETRIES       = 3

def summarize_ticket(ticket_text):

  # --- TOKEN PRE-CHECK ---
  # WHY: count tokens BEFORE sending. A 400 "context overflow" error from
  # the API gives you no information about which part overflowed.
  # Checking locally means you can log the ticket_id and input length.
  input_token_count = count_tokens(ticket_text)
  if input_token_count > MAX_INPUT_TOKENS:
    log_error("ticket too long", tokens=input_token_count)
    raise FatalError("Input exceeds token limit: " + input_token_count)

  # --- MODEL ROUTING ---
  # WHY: short tickets don't need the most capable model.
  # This check costs nothing and can reduce per-call cost by 10-60×.
  if input_token_count < 500:
    model_id = "small-fast-model"
  else:
    model_id = "large-capable-model"

  messages = [
    {
      role: "system",
      content: "You are a support ticket summarizer. Summarize in 2-3 sentences.
                Focus on: the reported issue, its impact, and any steps already tried."
    },
    {
      role: "user",
      content: "Please summarize this ticket:\n\n" + ticket_text
    }
  ]

  request = {
    model: model_id,
    messages: messages,
    max_tokens: MAX_OUTPUT_TOKENS,
    temperature: 0.2,
    stream: true  # WHY: streaming — user sees output start within ~100ms
  }

  # --- RETRY LOOP ---
  for attempt in range(MAX_RETRIES):
    try:

      # --- STREAMING RESPONSE ---
      buffer = ""
      stream = model.stream(request)
      for chunk in stream:
        # WHY: accumulate into buffer — each chunk is only the delta,
        # not the full response so far.
        buffer += chunk.delta
        display(chunk.delta)  # show each token as it arrives

      summary = buffer

      # WHY: the final chunk carries usage stats — log them every call.
      # This is the source of truth for cost monitoring.
      usage = stream.final_usage()
      log_cost({
        model:         model_id,
        input_tokens:  usage.input_tokens,
        output_tokens: usage.output_tokens,
        cost_usd:      estimate_cost(model_id, usage)
      })

      return summary

    except RateLimitError:
      # WHY: retryable — rate limits are temporary.
      if attempt == MAX_RETRIES - 1:
        raise FatalError("Rate limit persists after " + MAX_RETRIES + " retries")
      # WHY: jitter prevents all retry attempts from hitting the API at the same instant.
      delay = BASE_RETRY_DELAY * (2 ** attempt) + random_jitter(0, 1)
      log_warning("rate limited, retrying in " + delay + "s (attempt " + attempt + ")")
      sleep(delay)

    except ServerError:
      # WHY: retryable — transient provider-side failure.
      if attempt == MAX_RETRIES - 1:
        raise FatalError("Server error persists after retries")
      delay = BASE_RETRY_DELAY * (2 ** attempt) + random_jitter(0, 1)
      log_warning("server error, retrying in " + delay + "s")
      sleep(delay)

    except BadRequestError as e:
      # WHY: NOT retryable — bad requests fail identically every time.
      # Retrying wastes time and obscures the real problem.
      raise FatalError("Bad request (not retryable): " + e.message)
```

---

## Failure mode walkthrough

### Failure 1: Context overflow with no pre-check

**What happens:** The user pastes a 5,000-word ticket. The API returns a `400 Bad Request` with a cryptic message. The app crashes. The user sees a generic error.

**Why it happens:** The messages array plus the requested `max_tokens` exceeds the model's context limit. The model never starts generating — the error is returned immediately.

**How the example handles it:** The token pre-check at the top of the function counts input tokens before the request is sent. Inputs over `MAX_INPUT_TOKENS` are rejected locally with a meaningful error message, not after a round-trip to the API.

**What would break without this:** The API call fails with a 400. Because 400 is not retried (correctly), the error surfaces immediately — but the message is `"context_length_exceeded"` with no indication of which ticket triggered it or by how much.

---

### Failure 2: Rate limit with no backoff

**What happens:** Under load, the app is sending 50 requests per second. The API starts returning `429 Too Many Requests`. With no retry logic, every queued request fails immediately.

**Why it happens:** Rate limits are per-account or per-minute quotas. They trigger under normal production load, not just abuse.

**How the example handles it:** The retry loop catches `RateLimitError` and waits with exponential backoff: 1s, 2s, 4s (plus jitter). It retries up to `MAX_RETRIES` times before giving up and logging the failure.

**What would break without this:** Every request during the rate-limited period fails with an unhandled exception. Users see errors. The load spikes again when requests are re-submitted, creating a feedback loop.

---

### Failure 3: Streaming chunks displayed without accumulation

**What happens:** The developer prints each chunk as it arrives. The output looks like: `"The"` ... `" ticket"` ... `" describes"` ... instead of building up a coherent sentence.

**Why it happens:** Each chunk contains only the new tokens since the last event — the delta — not a running total. Without accumulation, each displayed chunk replaces or appends incorrectly.

**How the example handles it:** `buffer += chunk.delta` accumulates every token into a complete string. The display call shows the delta for the streaming effect; `buffer` holds the final response for logging and return.

**What would break without this:** The function returns only the last chunk (a few tokens) instead of the complete summary. Downstream code that depends on the full response gets garbage.

---

## Production checklist for this module

- [ ] Set `max_tokens` explicitly on every call — never rely on the model default
- [ ] Count input tokens before sending; reject or truncate oversized inputs locally
- [ ] Implement exponential backoff with jitter for `429` and `5xx` errors
- [ ] Never retry `400`, `401`, or `404` errors — they won't resolve
- [ ] Use streaming for any user-facing generation
- [ ] Log token counts from every response — `response.usage` is your cost ledger

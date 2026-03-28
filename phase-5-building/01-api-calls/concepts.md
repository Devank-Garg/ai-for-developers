# Concepts: API Calls

> **Tag: core** — Every LLM application begins here: sending a request, reading a response, and doing both reliably without breaking the bank.

---

## Request Anatomy

**What it is** — The structured payload you send to an LLM: a model identifier, a list of messages, and optional parameters that control how the model responds.

**Why it matters** — Malformed requests return errors or produce unpredictable output; understanding the structure lets you debug failures immediately instead of guessing.

**How it works** — Every request has three required pieces:

1. **Model** — which model to use (`"gpt-4o"`, `"claude-sonnet-4-6"`, `"gemini-1.5-pro"`). Different models have different context limits, costs, and capabilities. The model is not optional.

2. **Messages** — an ordered array of turns. Each message has a `role` (`"system"`, `"user"`, or `"assistant"`) and `content` (the text). The array is the conversation: the model reads it top-to-bottom and generates the next turn.

3. **Parameters** — optional controls on the output:
   - `max_tokens`: upper bound on response length in tokens
   - `temperature`: randomness (0 = deterministic, 1 = creative, 2 = chaotic)
   - `stop`: one or more strings that end generation early
   - `stream`: whether to stream tokens back as they're generated

```
request = {
  model: "claude-sonnet-4-6",
  messages: [
    { role: "system",    content: "You are a helpful assistant." },
    { role: "user",      content: "Summarize this article: ..." }
  ],
  max_tokens: 500,
  temperature: 0.3
}

response = model.chat(request)
text = response.content
```

The model never sees your code, your variable names, or your intent. It only sees the messages array. Everything you want the model to know must be in there.

**Production tip** — Always set `max_tokens` explicitly. Without it, a single call can consume an entire context window worth of tokens and run for minutes, burning budget and blocking your user.

**Common failure** — Sending `temperature: 0` and expecting perfectly identical responses across calls. Temperature 0 is nearly deterministic but not bit-for-bit identical — floating-point differences in inference infrastructure introduce variation. Never rely on exact response reproducibility.

---

## Token Budgeting

**What it is** — The process of accounting for how many tokens a request will consume before sending it, so you avoid hitting limits or overspending.

**Why it matters** — Every model has a maximum context length; exceeding it returns a hard error. Tokens also determine cost — an unbudgeted app can cost 10× more than expected at scale.

**How it works** — Tokens are not words. A rough rule: 1 token ≈ 4 characters in English, but punctuation, code, and non-Latin scripts tokenize differently. A 1,000-word article is roughly 1,300 tokens.

Your budget must cover three categories:

| Category | What it includes |
|----------|-----------------|
| Input tokens | System prompt + entire message history + any injected context |
| Output tokens | The model's response (up to `max_tokens`) |
| Overhead | Formatting tokens added by the model provider (usually 3–10 tokens) |

The formula:
```
available_for_output = context_limit - input_tokens - overhead_buffer
set max_tokens = min(desired_output_length, available_for_output)
```

If `input_tokens + max_tokens > context_limit`, the call will fail with a 400-class error before any generation occurs.

Most providers return the actual token counts in the response object. Log them on every call — this is the cheapest form of cost monitoring.

```
response = model.chat(request)
log({
  input_tokens:  response.usage.input_tokens,
  output_tokens: response.usage.output_tokens,
  total_cost:    estimate_cost(response.usage)
})
```

**Production tip** — Budget tokens on the input side before sending, not just on the output side. Use a tokenizer library to count the message array before the call — catching an oversized request locally avoids a round-trip error and a confusing 400 response.

**Common failure** — Allocating `max_tokens: 4096` on a model with a 4096-token context limit. The model cannot generate 4096 output tokens if the input already consumed 2000 — the call returns a context overflow error or silently truncates the response.

---

## Error Handling and Retries

**What it is** — The logic that decides what to do when an API call fails, including which errors to retry and which to surface immediately.

**Why it matters** — LLM APIs fail regularly under normal operation: rate limits, transient server errors, and network timeouts are expected, not exceptional. An app without retry logic will fail visibly to users under normal load.

**How it works** — Errors fall into two categories:

**Retryable errors** — temporary conditions that will likely resolve:
- `429 Too Many Requests` — rate limit hit; wait and retry
- `500 Internal Server Error` — provider-side transient failure
- `503 Service Unavailable` — overloaded or deploying
- Network timeouts and connection resets

**Fatal errors** — problems you caused that retrying will not fix:
- `400 Bad Request` — malformed request (wrong model name, context overflow, invalid parameter)
- `401 Unauthorized` — bad API key
- `404 Not Found` — wrong endpoint or model name

The standard retry strategy is **exponential backoff with jitter**:
```
max_attempts = 3
base_delay = 1  # seconds

for attempt in range(max_attempts):
  try:
    response = model.chat(request)
    return response
  except RateLimitError:
    if attempt == max_attempts - 1:
      raise FatalError("Rate limit exceeded after retries")
    delay = base_delay * (2 ** attempt) + random_jitter(0, 1)
    sleep(delay)
  except BadRequestError as e:
    raise FatalError(e)  # don't retry — it will fail identically
```

Jitter (a small random offset) prevents the "thundering herd" problem where all your instances retry at the same second and re-trigger the rate limit together.

**Production tip** — Log every retry attempt with the error code, attempt number, and delay. Silent retries make incidents invisible — you want to know when your app is experiencing elevated error rates before users notice.

**Common failure** — Retrying `400 Bad Request` errors. A bad request will fail identically every time; retrying it wastes time, burns your retry budget, and delays surfacing the real error.

---

## Streaming vs. Batch Responses

**What it is** — The choice between receiving a complete response when generation finishes (batch) or receiving tokens incrementally as they are generated (streaming).

**Why it matters** — For interactive interfaces, batch mode means the user sees nothing for 5–30 seconds; streaming makes the app feel fast by starting to display text within milliseconds.

**How it works** — In batch mode, the API holds the connection until the model finishes, then sends the full response. In streaming mode, the API sends small token chunks as a stream of events.

**Batch mode:**
```
response = model.chat(request)
display(response.content)  # user waits until this line
```

**Streaming mode:**
```
buffer = ""
stream = model.stream(request)
for chunk in stream:
  buffer += chunk.delta   # accumulate
  display(chunk.delta)    # show each token as it arrives

final_text = buffer
```

The tradeoff:

| | Batch | Streaming |
|---|-------|-----------|
| Time to first token | Full generation time | ~100ms |
| Perceived speed | Slow | Fast |
| Code complexity | Simple | Requires accumulation |
| Error handling | Clean | Must handle mid-stream errors |

Streaming does not speed up generation — the model still generates at the same rate. It only changes when tokens arrive at the client.

**Production tip** — Use streaming for any user-facing generation. Use batch for background jobs, classification, or any call where you need the complete response before doing anything with it (e.g., JSON parsing, structured extraction).

**Common failure** — Displaying streaming chunks without accumulating them. Each chunk contains only the delta — the new tokens since the last event. Displaying each chunk individually shows fragmented words rather than a building sentence. Always accumulate chunks into a buffer.

---

## Cost Awareness

**What it is** — The discipline of understanding and controlling what your LLM calls cost, measured in tokens consumed and dollars per day at scale.

**Why it matters** — LLM costs are not fixed — they scale with input length, model choice, usage volume, and retry count. An unmonitored app can generate a bill 100× larger than expected with no bugs and no misuse.

**How it works** — Cost has two levers:

**Token efficiency** — reducing tokens consumed per call:
- Shorter system prompts (bullet points, not paragraphs)
- Truncating conversation history when it grows long
- Removing irrelevant context before injection
- Using `stop` sequences to end generation early

**Model selection** — matching model capability to task complexity:
```
# Rough pricing ratio (verify current rates on provider pages):
# Small/fast model: ~$0.25 per 1M input tokens
# Large/capable model: ~$15 per 1M input tokens
# The difference is ~60×

def route(task):
  if task.type in ["classification", "extraction", "formatting"]:
    return small_fast_model   # 60× cheaper
  else:
    return large_capable_model
```

The cheapest optimization is usually model selection, not prompt rewriting.

Monitor actual spend using the token counts in every response. Log them, aggregate by day, and set budget alerts at the provider level before shipping.

**Production tip** — Set a hard spending cap at the API account level. Every provider supports this. It is a backstop against runaway costs from bugs or infinite loops — and it takes two minutes to configure.

**Common failure** — Using a frontier model for every task. Classification, short-text summarization, format conversion, and extraction tasks rarely need the most capable model. Routing simple tasks to a smaller model at 1/60th of the cost is not a compromise — it is correct design.

---

## Further reading

**Interactive tokenizer** — [https://platform.openai.com/tokenizer](https://platform.openai.com/tokenizer)
Paste any text to see how it splits into tokens. Use this to calibrate estimates before building your budgeting logic.

**Exponential backoff and jitter** — [https://aws.amazon.com/blogs/architecture/exponential-backoff-and-jitter/](https://aws.amazon.com/blogs/architecture/exponential-backoff-and-jitter/)
The canonical explanation of why jitter matters in distributed retry logic. The examples are for DynamoDB but the math applies directly to LLM retries.

**HTTP status codes** — [https://developer.mozilla.org/en-US/docs/Web/HTTP/Status](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status)
Provider error codes map to standard HTTP semantics. If you know the HTTP category, you know whether to retry.

**LLM pricing and performance comparison** — [https://artificialanalysis.ai/](https://artificialanalysis.ai/)
Tracks pricing benchmarks across providers. Useful for model routing decisions.

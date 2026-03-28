# Quiz: API Calls

These questions are scenario-based. The goal is to apply what you know, not recite it. Attempt each before reading the answer.

---

## Questions

**1.** You set `max_tokens: 100` on a call to generate a product description. The response always cuts off mid-sentence. A colleague suggests "just raise it to 500." Before doing that, what two things should you check — and why might raising the number not actually be the fix?

---

**2.** Your app makes API calls as users submit forms. It works fine in development. In production with 100 concurrent users, about 15% of calls fail with `429 Too Many Requests`. You add retry logic that immediately retries the failed call up to 3 times. The error rate drops briefly, then returns to the same level. What did you do wrong, and what should you do instead?

---

**3.** You're building a document analysis tool. Users upload PDFs, your app extracts the text (often 10,000–50,000 words), and sends it to the model for analysis. The API call sometimes succeeds and sometimes fails with `400 Bad Request: context_length_exceeded`. The failures seem random. What is the actual cause of the inconsistency, and what is the correct fix?

---

**4.** Your app sends the same prompt to the model with `temperature: 0` every time a user asks the same question. A user reports that they got two different answers to identical questions submitted 5 minutes apart. A teammate says this is impossible with temperature 0 and wants to file a bug with the API provider. Should you file the bug? Why or why not?

---

**5.** You're reviewing costs for a new app after its first week. The bill is 8× higher than your estimate. You look at the logs and find: (a) the app is using the largest available model for all tasks including simple yes/no classification calls, and (b) `max_tokens` is not set on any call. Which of these two issues is likely causing more of the cost overrun, and why?

---

**6.** A user submits a long support ticket. Your app generates a streaming summary. The function returns the summary to downstream code, which tries to parse it as JSON but gets a JSON parse error. You look at the streaming code and see:

```
for chunk in stream:
  display(chunk.delta)
return chunk.delta   # returns last chunk only
```

What is wrong, what does the returned value actually contain, and how do you fix it?

---

## Answers

**1.**

Two things to check before raising `max_tokens`:

First, check whether the model is actually hitting the token limit or stopping for a different reason (a `stop` sequence, a formatting instruction in the system prompt, or a natural sentence ending that the model mistook for completion). Look at `response.finish_reason` — if it's `"max_tokens"`, the limit is the cause. If it's `"stop"`, the model stopped intentionally.

Second, check whether raising `max_tokens` is actually possible given your context limit. The formula is: `available_for_output = context_limit - input_tokens - overhead`. If your input is already consuming most of the context window, raising `max_tokens` won't work — you'll hit the context limit error instead. (See: **Token budgeting**)

Raising the number blindly without checking `finish_reason` or the input token count is the wrong instinct — you may fix nothing and introduce a different error.

---

**2.**

The problem is retrying immediately without any delay. When 100 concurrent users all hit the rate limit at the same time, they all retry at the same time, which generates another burst of requests that re-triggers the rate limit. This is the "thundering herd" problem.

The fix is **exponential backoff with jitter**: wait 1 second before the first retry, 2 seconds before the second, 4 seconds before the third — and add a small random offset (jitter) to each delay. The jitter staggers the retries so they don't all hit the API simultaneously. (See: **Error handling and retries**)

Without jitter, synchronised retries from multiple instances can be as damaging as the original load spike.

---

**3.**

The inconsistency is caused by variable document length. Short PDFs produce small input token counts that fit within the context limit; long PDFs produce large counts that don't. The failure is not random — it is deterministic based on token count, but because document length varies per upload, it appears random.

The correct fix has two parts:

1. Count tokens before sending (using a tokenizer library). If the input exceeds your budget, either reject it with a clear error ("Document too long — maximum X words") or truncate/chunk it before sending.
2. Set a maximum input token cap in your application layer so you catch oversize inputs before they reach the API.

Relying on the API to tell you the input was too long is the wrong place to catch this — you want to handle it before the round-trip. (See: **Token budgeting**)

---

**4.**

Do not file the bug. Temperature 0 is nearly deterministic, but not guaranteed to be bit-for-bit identical across calls. Floating-point arithmetic in distributed GPU inference can introduce tiny differences that cause the model to select a different token at certain positions.

This is documented behavior, not a bug. The correct mental model is: temperature 0 gives you the most deterministic output the system can provide, not a hash function that always returns the same bytes. If your application requires identical responses to identical inputs, you need to cache the response — not fix the temperature. (See: **Request anatomy**)

---

**5.**

Both issues are real, but model selection is likely causing more of the overrun. The pricing ratio between a small model and a large model is roughly 60:1. If 80% of your calls are simple classification tasks running on the large model, switching them to a small model reduces those calls' costs by ~98%. That's structural — it doesn't matter how well you optimize `max_tokens`.

Unset `max_tokens` is also a real problem — the model can generate a full context window's worth of output — but for classification calls, the output is usually short and the model stops naturally. For long-form generation, it compounds the cost.

Fix model routing first. Then audit `max_tokens` for calls that are actually generating long outputs. (See: **Cost awareness** and **Token budgeting**)

---

**6.**

The bug is on the last line: `return chunk.delta` returns only the delta from the final streaming chunk — typically a few characters or an empty string, not the complete response.

Each chunk's `.delta` contains only the tokens generated since the previous chunk. To get the full response, you must accumulate all deltas into a buffer throughout the stream.

The fix:
```
buffer = ""
for chunk in stream:
  buffer += chunk.delta  # accumulate
  display(chunk.delta)
return buffer  # return the complete accumulated response
```

Returning `chunk.delta` from the last chunk is a very common streaming bug — the downstream JSON parser gets a fragment (or an empty string) instead of a complete response. (See: **Streaming vs. batch responses**)

# Resources: API Calls

Curated external resources for going deeper on LLM API fundamentals. Each link is chosen for quality and stability.

---

## Official cookbooks and reference code

**[OpenAI Cookbook](https://github.com/openai/openai-cookbook)**
https://github.com/openai/openai-cookbook
The canonical collection of worked examples for the OpenAI API — rate limit handling, streaming, token counting, batching, and more. Provider-specific but the patterns apply across APIs.

**[Anthropic Cookbook](https://github.com/anthropics/anthropic-cookbook)**
https://github.com/anthropics/anthropic-cookbook
Anthropic's equivalent: worked examples for tool use, streaming, vision, and prompt caching. Well-maintained and frequently updated.

---

## Official documentation

**[Anthropic API error reference](https://docs.anthropic.com/en/api/errors)**
https://docs.anthropic.com/en/api/errors
Complete list of error codes with explanations. Bookmark this when debugging — it tells you exactly which errors are retryable and which aren't.

**[OpenAI rate limits guide](https://platform.openai.com/docs/guides/rate-limits)**
https://platform.openai.com/docs/guides/rate-limits
How rate limits work, how to request increases, and the full breakdown of limit types (RPM, TPM, RPD). Provider-specific but the concepts are universal.

---

## Engineering blogs

**[How we handle rate limiting at scale — Stripe Engineering](https://stripe.com/blog/rate-limiters)**
https://stripe.com/blog/rate-limiters
Stripe's deep dive on rate limiter design. Not LLM-specific, but the retry strategy and backoff math applies directly to LLM API clients. One of the best treatments of the subject online.

---

## Video

**[Andrej Karpathy — Intro to Large Language Models](https://www.youtube.com/watch?v=zjkBMFhNj_g)**
https://www.youtube.com/watch?v=zjkBMFhNj_g
1-hour talk covering how LLMs work at the level a developer needs. Not API-specific, but gives the mental model behind token generation, context windows, and temperature that makes the API concepts in this module make sense.

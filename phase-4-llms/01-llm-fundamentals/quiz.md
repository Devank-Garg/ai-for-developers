# Quiz: LLM Fundamentals

Attempt these questions before checking the answers. The goal isn't to pass a test — it's to find the gaps before they become problems later.

---

## Questions

**1.** At its core, what function does an LLM perform? Describe it in one sentence without using the words "AI" or "intelligence."

---

**2.** A developer says: "I sent the model my message and it wrote back a full paragraph in one go." What is technically inaccurate about this description?

---

**3.** You are building a customer support chatbot. A user sends a message in Japanese. Your context window is 8,000 tokens and the conversation has been going for a while. Why might you run out of context faster than expected, and what is the root cause?

---

**4.** A pre-trained language model — trained only on next-token prediction, never instruction-tuned — is given the prompt: "What is the capital of France?" What would it most likely produce, and why?

a) "Paris" — it learned factual knowledge during pre-training
b) Something that continues the text in a plausible way, not necessarily a direct answer
c) An error, because it hasn't learned to answer questions
d) "I don't know" — it knows the limits of its knowledge

---

**5.** Explain the three stages of training a production LLM (pre-training, instruction tuning, alignment) in plain terms. What does each stage add that the previous one lacked?

---

**6.** You call an LLM API with `temperature = 0`. Your colleague calls the same API with the same prompt but `temperature = 1.2`. Describe how their outputs will likely differ and why.

---

**7.** A user asks a deployed chatbot: "What happened in the news today?" The model responds with a confident, detailed summary of recent events. Should you trust this response? Explain what is actually happening.

---

**8.** You are building an application that needs to remember user preferences across sessions — for example, "this user prefers formal language and metric units." How would you implement this? What does the model provide, and what must you build yourself?

---

**9.** Your application sends an LLM a document to summarise. The document is 50,000 words. Your model has a 128,000-token context window. Will this fit? Show your reasoning.

---

**10.** What is in-context learning? Give a concrete example of how you would use it. Why does it work in large models but not small ones?

---

**11.** A colleague proposes using `temperature = 0` for all LLM calls in your application to "make the model more accurate." Is this good advice? When would you agree and when would you push back?

---

**12.** Rank these use cases from lowest to highest recommended temperature setting, and briefly justify your ranking:

- Generating creative marketing taglines
- Extracting structured data (names and dates) from a contract
- Writing a conversational reply in a customer chat
- Answering a factual question about a product's return policy

---

## Answers

**1.** An LLM is a function that takes a sequence of tokens as input and outputs a probability distribution over the next possible token.

The key insight is the word "distribution" — the model doesn't output *a* word, it outputs the probability of *every possible* next word. What you observe as output is the result of sampling from that distribution repeatedly.

---

**2.** The model did not write the paragraph "in one go." It generated one token at a time, autoregressively — each token produced by a separate forward pass, with that token then appended to the input for the next step. A 100-token response involved 100 sequential forward passes. This is why generation has latency that scales with output length, and why streaming (receiving tokens as they are produced) is common in production systems.

---

**3.** Japanese text tokenises much more densely than English. A message that would use 100 tokens in English may use 200–400 tokens in Japanese, depending on the model's tokeniser. The conversation will consume the context window roughly 2–4× faster than an equivalent English conversation. The root cause is that tokenisation is designed primarily around English text — characters and scripts from other languages are split into smaller sub-word pieces, each consuming a token. This also directly affects API cost.

---

**4.** b) Something that continues the text in a plausible way, not necessarily a direct answer.

A raw pre-trained model has not been trained to behave like an assistant. It's a next-token predictor. Given "What is the capital of France?", it might continue with "...is a question that appears in many geography quizzes. The answer, of course..." or it might generate another question, depending on what patterns in its training data best match that input. Instruction tuning is what teaches a model to recognise a question and respond with a direct answer.

---

**5.**

**Pre-training**: The model is trained on trillions of tokens using next-token prediction. It learns language, facts, reasoning patterns, and world knowledge — but it behaves like a text predictor, not an assistant. It will "complete" your input rather than answer it. Very expensive; done by labs with GPU clusters.

**Instruction tuning (supervised fine-tuning)**: The model is fine-tuned on a dataset of prompt-response pairs, teaching it to respond *as an assistant*. It learns the format: "when given a question, give a direct answer; when given an instruction, follow it." This is what transforms a text predictor into something you can have a useful conversation with.

**Alignment (RLHF / RLAIF)**: Even an instruction-tuned model can give harmful, dishonest, or unhelpful outputs. The alignment stage uses human or AI feedback to reward outputs that are helpful, harmless, and honest — and penalise outputs that aren't. This is what produces the guardrails and values you observe in deployed models like Claude or GPT-4.

---

**6.** With `temperature = 0`, the model deterministically picks the highest-probability token at every step. Calling it twice with the same prompt produces identical output. The response tends to be conservative, direct, and may be repetitive.

With `temperature = 1.2`, the probability distribution is flattened — lower-probability tokens become meaningfully more likely to be sampled. The output will be more varied, may include unexpected word choices, and will produce different results on repeated calls with the same prompt. At 1.2, responses may be more creative but are also more likely to drift, repeat, or lose coherence for factual tasks.

---

**7.** Do not trust this response. The model has no access to current news. Its knowledge comes entirely from training data with a fixed cutoff date — it cannot browse the internet, query live sources, or know what happened today. What you are observing is the model generating a plausible-sounding news summary based on patterns in its training data. It may be describing past events it trained on, fabricating plausible-sounding events entirely, or some mixture of both. If the application requires current information, you must retrieve it separately (e.g., via a news API) and provide it in the prompt. This is covered in the RAG module in Phase 5.

---

**8.** The model provides nothing for cross-session memory — each API call is stateless. The model has no knowledge of any previous conversation unless you include it in the current context.

To implement persistent preferences, you must:
1. **Store preferences yourself** — in a database, file, or key-value store, keyed to the user's identity
2. **Inject them at call time** — include the user's preferences in the system prompt or context for every call: `"User preferences: formal language, metric units."`

The model is a stateless function. Anything that feels like memory is state you maintain and re-inject into every call. This pattern — retrieving relevant information and placing it into the context — is the foundation of the memory and RAG patterns covered in Phase 5.

---

**9.** 50,000 words × (1 token / 0.75 words) ≈ **66,700 tokens** for the document alone.

A 128,000-token context window can hold the document — but barely. You also need to budget for:
- The system prompt
- Your summarisation instruction
- The model's output (a summary could be 500–2,000 tokens)

Total usage: ~70,000–72,000 tokens, which fits within 128K. However:
- If the document is not plain English (e.g., technical jargon, tables, code), token count may be higher
- Cost scales with input length — processing 67K tokens in the prompt is significantly more expensive than a typical call
- Model quality can degrade when attending to information deep in very long contexts

For production use on large documents, chunked summarisation or retrieval-based approaches (Phase 5) are often more reliable and cost-effective than stuffing the full document into context.

---

**10.** In-context learning is the ability of a large language model to perform a new task from a few examples provided in the prompt — without any weight updates or fine-tuning.

Concrete example: you want the model to extract prices from product descriptions in a specific JSON format. Rather than fine-tuning, you include two or three worked examples in the prompt:

```
Input: "The Pro plan costs $49 per month."
Output: {"price": 49, "currency": "USD", "period": "month"}

Input: "Enterprise pricing starts at €200/year."
Output: {"price": 200, "currency": "EUR", "period": "year"}

Input: "Basic tier is free for the first 30 days, then $12 weekly."
Output:
```

The model infers the pattern from the examples and completes the third one correctly, having never been explicitly trained on this format.

This works in large models because they developed sufficient representational capacity during pre-training to recognise and extend patterns from context. Small models lack this capacity — they can memorise training patterns but cannot flexibly generalise to new patterns seen only at inference time.

---

**11.** It depends on the task. `temperature = 0` is not "more accurate" in a general sense — it is deterministic and conservative. For factual, structured, or extraction tasks, this is usually the right choice. For conversational, creative, or open-ended tasks, it produces flat, repetitive, formulaic output that often feels unnatural to users.

Agree with `temperature = 0` for: classification, data extraction, factual Q&A, structured output generation.

Push back for: conversation interfaces, creative generation, summarisation where variety is acceptable, any task where multiple correct answers exist.

A blanket policy of `temperature = 0` would make a customer-facing chatbot feel robotic and produce the exact same response to the same question every time — which users notice and dislike.

---

**12.** Lowest to highest recommended temperature:

| Rank | Use case | Temperature range | Why |
|------|----------|-------------------|-----|
| 1 (lowest) | Extracting structured data from a contract | 0 – 0.1 | Exact, deterministic extraction. One correct answer. Randomness only introduces errors. |
| 2 | Answering a factual question about a return policy | 0.1 – 0.3 | Still factual, but a slightly higher temperature allows for natural phrasing without risking wrong facts. |
| 3 | Writing a conversational reply in a customer chat | 0.7 – 1.0 | Conversation benefits from natural variation. Too low feels robotic; this range produces warm, human-feeling replies. |
| 4 (highest) | Generating creative marketing taglines | 1.0 – 1.4 | The goal is variety and novelty. A low temperature would produce predictable, safe, similar outputs — defeating the purpose. Higher temperature generates unexpected combinations that can be genuinely creative. |

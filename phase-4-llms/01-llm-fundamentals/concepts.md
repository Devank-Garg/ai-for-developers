# Concepts: LLM Fundamentals

> **Tag: core** — This is the foundation for everything in Phase 4 and Phase 5.

---

## What an LLM actually is

You already understand functions. An LLM is a function:

```
f(tokens_in) → probability_distribution_over_next_token
```

That's the whole thing. Everything else — the chat interface, the reasoning, the code generation — is built on top of that single operation: given a sequence of tokens, predict the probability of every possible next token.

What makes it useful isn't the function form; it's what was done to define that function. A language model with billions of parameters, trained on hundreds of billions of tokens of text, develops a representation of language — and, incidentally, of much of what language talks about — that turns out to be remarkably general.  

When you call an LLM API, you are calling this function repeatedly. The model doesn't "think" in any human sense. It doesn't plan ahead or look anything up. It runs a forward pass through a very large neural network and produces a probability distribution. Your application then samples a token from that distribution, appends it to the input, and calls the function again.

> You might think the model reads your message and then writes a response. Actually, the model reads your message and the response *one token at a time*, each token produced by a separate call to the same function.

----

## Next-token prediction

The core task an LLM is trained on is deceptively simple: **given all the tokens so far, predict the next one**.

During pre-training, the model reads a sentence like:

```
The capital of France is ___
```

The correct next token is `Paris`. The model produces a probability distribution over its entire vocabulary (often 50,000–100,000+ tokens). If it assigns high probability to `Paris`, the loss is low. If it assigns high probability to `London`, the loss is high. Gradient descent nudges the weights to do better next time.

This is repeated hundreds of billions of times, across text from the internet, books, code, scientific papers, and more. What emerges is a model that has compressed an enormous amount of knowledge about language and the world into its weights — not because it was programmed with that knowledge, but because predicting the next token well requires it.

A model that can reliably predict the next token in a chemistry textbook must, in some sense, "know" chemistry. A model that can predict the next token in Python source code must understand code structure. The task is simple; the knowledge required to do it well at scale is vast.

---

## Autoregressive generation

When you prompt an LLM, it generates a response one token at a time. This process is called **autoregressive generation**:

```
Step 1: input = [your_prompt]
        → model outputs distribution → sample token_1

Step 2: input = [your_prompt, token_1]
        → model outputs distribution → sample token_2

Step 3: input = [your_prompt, token_1, token_2]
        → model outputs distribution → sample token_3

... (repeat until stop token or max length)
```

Each token is appended to the growing input before the next step. The model re-processes the entire sequence each time. This means:

- **Generation is sequential** — you can't generate token 50 until you have tokens 1–49
- **Each step is a full forward pass** — at full context length, each step is expensive
- **Earlier tokens influence all later tokens** — but later tokens cannot influence earlier ones (causal masking)

This is why generating a long response takes longer than a short one, and why APIs charge for both input and output tokens separately (output tokens require more forward passes).

---

## Tokens and tokenisation

Models do not process characters or words — they process **tokens**.

Tokenisation splits text into chunks before it enters the model. The exact scheme varies by model, but the general pattern for English:

- Common short words are one token: `the`, `is`, `cat`
- Longer or less common words are split: `tokenisation` → `token` + `isation`
- Punctuation and spaces are their own tokens
- Numbers may be split digit by digit: `2024` → `20` + `24`
- Code and non-English text often use more tokens per word

**Why this matters for you:**

| Concern | Why tokens are the cause |
|---------|--------------------------|
| API cost | Priced per input + output token, not per character or word |
| Context window | Measured in tokens, not words (a 128K context ≠ 128K words) |
| Non-English content | May use 2–5× more tokens than equivalent English |
| Code | Token count can vary significantly by language and style |
| Counting | ~4 characters per token, ~0.75 words per token — rough rules of thumb only |

**Try it yourself**: Tiktokenizer (https://tiktokenizer.vercel.app/) shows you exactly how any text is tokenised for OpenAI models. The OpenAI tokenizer (https://platform.openai.com/tokenizer) does the same for GPT models specifically.

> **Production note:** When estimating cost or context usage, always count tokens — not words or characters. A "short" message in Japanese may be 3× as expensive as the same message in English.

---

## The training pipeline

A model you call via API today went through (at minimum) three stages of training:

### Stage 1: Pre-training

The model is trained on a massive dataset — trillions of tokens from the internet, books, code, academic papers, and other sources — using next-token prediction. This is the most compute-intensive stage. GPT-3 used roughly 300 billion tokens and cost an estimated $4M in compute at the time.

After pre-training, the model can predict text very well. But it isn't yet helpful in the way you'd want. It might respond to "How do I make coffee?" by continuing the text in the style of a recipe blog, or generating another question, depending on what patterns in the training data match.

### Stage 2: Instruction tuning (supervised fine-tuning)

The pre-trained model is fine-tuned on a smaller dataset of prompt-response pairs, written or curated by humans. This teaches the model to respond *as an assistant* — to answer questions, follow instructions, and engage in dialogue rather than just continue text.

This stage transforms a "predict the next token" engine into something that behaves like a conversational assistant.

### Stage 3: Alignment (RLHF / RLAIF)

Instruction-tuned models can still produce harmful, dishonest, or unhelpful outputs. The alignment stage uses feedback — from humans (RLHF: Reinforcement Learning from Human Feedback) or from another model (RLAIF: Reinforcement Learning from AI Feedback) — to shape the model toward outputs that are helpful, harmless, and honest.

A simplified version of RLHF:
1. Generate several responses to the same prompt
2. Have humans rank which responses are better
3. Train a **reward model** to predict human preference scores
4. Fine-tune the LLM to produce outputs the reward model rates highly

```
Pre-training              → knows language and world knowledge
Instruction tuning        → knows how to behave as an assistant
Alignment (RLHF/RLAIF)   → shaped toward helpful, harmless, honest outputs
```

When you use Claude, GPT-4, or Gemini, you are using a model that has gone through all three stages.

---

## Context window

The **context window** is the maximum number of tokens the model can process in a single call — input plus output combined.

Everything inside the context window is what the model "sees." Everything outside it does not exist.

```
[system prompt] [conversation history] [current message] [response so far]
|←————————————————— context window ————————————————————→|
```

If the conversation grows larger than the context window, you must truncate or summarise earlier parts. The model has no ability to "remember" what fell outside the window — there is no hidden memory store.

**Practical implications:**

| Context size | What fits (rough) |
|---|---|
| 8K tokens | ~6,000 words — a long document or short story |
| 32K tokens | ~24,000 words — a short book chapter or large codebase |
| 128K tokens | ~96,000 words — a full novel or a large codebase |
| 1M tokens | ~750,000 words — an entire codebase or multiple books |

Context windows have expanded dramatically — from 4,096 tokens in early GPT-3 to over 1 million tokens in some 2024–2025 models. But larger contexts are more expensive (cost typically scales linearly or superlinearly with input length), and model quality can degrade at very long ranges depending on architecture.

> **Production note:** Don't assume that "long context = problem solved." Models can have difficulty attending to information buried in the middle of very long contexts. Retrieval (Phase 5) is often better than stuffing everything in.

---

## Temperature and sampling

The model outputs a **probability distribution** over all possible next tokens — not a single token. How you sample from that distribution determines how the output behaves.

**Temperature** controls how "peaky" or "flat" the distribution is:

- `temperature = 0` — always pick the single highest-probability token. Deterministic, but can be repetitive or formulaic.
- `temperature = 1` — sample proportionally to the model's probabilities. The default for most models.
- `temperature > 1` — flatten the distribution, making lower-probability tokens more likely. More random and creative, but also more likely to be wrong or incoherent.

```
Low temperature (0.1):      High temperature (1.5):
"Paris" → 95%               "Paris" → 40%
"Lyon"  →  3%               "Lyon"  → 20%
"Rome"  →  1%               "Rome"  → 15%
...                         "Berlin"→ 12%
                            ...
```

**Top-p (nucleus sampling)** is an alternative: instead of a fixed temperature, you sample only from the smallest set of tokens whose cumulative probability exceeds `p`. `top_p = 0.9` means: only consider tokens until you've covered 90% of the probability mass.

**Practical guidance:**

| Use case | Temperature |
|----------|-------------|
| Factual Q&A, classification | 0 – 0.3 |
| Summarisation, analysis | 0.3 – 0.7 |
| Conversation, general use | 0.7 – 1.0 |
| Creative writing, brainstorming | 1.0 – 1.5 |

---

## What LLMs don't have

This is where most developers get burned. These things are **not** present in a base LLM call unless explicitly added:

| What's missing | What this means |
|---|---|
| **Memory** | Each API call starts fresh. The model has no recollection of previous conversations unless you include them in the context. |
| **Current date/time** | The model doesn't know what day it is. If you don't tell it, it will guess — often wrong. |
| **Internet access** | The model cannot browse the web, look up current information, or check facts against live sources. Its knowledge comes from training data with a cutoff date. |
| **Tools** | The model cannot run code, query databases, or call APIs on its own. Tool use must be explicitly architected by you (covered in Phase 5). |
| **Persistent identity** | A "Claude" you talked to yesterday and the "Claude" you're talking to now share trained weights but have no shared conversational memory unless you provide it. |
| **Certainty about itself** | The model doesn't know what it doesn't know. It will answer confidently even when it's wrong. |

> Building an application that *feels* like the model has memory, access to current info, or the ability to take action means you, the developer, are implementing those capabilities — using the context window, retrieval, and tool-calling patterns. The model is a capable function; the architecture around it is your job.

---

## Emergent capabilities

One of the more surprising findings in LLM research is **emergence**: capabilities that appear suddenly as models scale, rather than improving gradually.

Small models (a few hundred million parameters) mostly interpolate from training data. As models scale into the billions, qualitatively new capabilities appear — multi-step reasoning, analogical thinking, code generation, in-context learning — that weren't explicitly trained for and weren't present in smaller models.

**In-context learning** is the clearest example. If you provide a few examples of a task in the prompt, a large enough model can perform that task without any weight updates. It has "learned" the pattern from the examples in context. Small models cannot do this reliably.

This is why scale matters so much and why the difference between a 7B and 70B parameter model isn't just "a bit better" — it can be qualitatively different for reasoning-heavy tasks.

Emergence also means that model behaviour can surprise you — including in undesirable ways. Models sometimes generalise patterns from training data in ways their creators didn't anticipate or intend.

---

## The model landscape

A brief orientation to avoid confusion:

**Closed / hosted models** — accessed via API, weights not public. Examples: GPT-4o (OpenAI), Claude (Anthropic), Gemini (Google). You pay per token. You don't control the model.

**Open-weight models** — weights released publicly. Examples: Llama 3 (Meta), Mistral, Qwen (Alibaba), Gemma (Google). You can run them yourself, fine-tune them, or host them.

**Quantised models** — open-weight models compressed to run on consumer hardware (via tools like llama.cpp, Ollama). Quality/performance tradeoff for accessibility.

**Multimodal models** — accept inputs beyond text (images, audio, video). Covered in module 06.

For most developers building applications, the choice is between closed APIs (easier, more capable, costs money) and self-hosted open models (more control, more ops burden, free to run). That decision is covered in depth in Phase 4 module 04 and Phase 5.

---

## Summary

| Concept | Key point |
|---------|-----------|
| LLM as function | Maps token sequence → probability over next token. One forward pass per token generated. |
| Next-token prediction | The training task. Simple objective; enormous knowledge required to do it well. |
| Autoregressive generation | Tokens generated one at a time, each fed back as input for the next step. |
| Tokenisation | Text split into chunks (~4 chars each). Affects cost, context usage, and behaviour across languages. |
| Pre-training | Massive-scale next-token prediction on internet-scale data. Expensive. Done once. |
| Instruction tuning | Fine-tuning on prompt-response pairs to teach assistant behaviour. |
| Alignment (RLHF) | Feedback-based training to steer toward helpful, harmless, honest outputs. |
| Context window | The model's full "working memory" for one call. Anything outside it doesn't exist. |
| Temperature | Controls randomness in sampling. 0 = deterministic; higher = more creative/risky. |
| What LLMs lack | No memory, no current knowledge, no tools, no certainty — unless you build those in. |
| Emergence | Capabilities that appear at scale but not in smaller models. Reasoning, in-context learning. |

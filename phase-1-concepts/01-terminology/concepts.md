# Concepts: Terminology

---

## The landscape: AI, ML, and deep learning

These three terms are often used interchangeably. They shouldn't be — they describe different scopes of the same field.

**Artificial intelligence (AI)** is the broadest term. It refers to any technique that enables a machine to mimic behaviour we would consider intelligent if a human did it. This includes everything from a chess engine that uses hand-coded rules to a neural network that learned to generate images. The field has existed since the 1950s, and the definition has shifted over time — tasks we consider "solved" tend to stop feeling like AI.

**Machine learning (ML)** is a subset of AI. Instead of programming explicit rules, you expose a system to data and let it figure out the rules itself. A spam filter that you hand-coded with keyword rules is AI. A spam filter that learned what spam looks like by reading millions of emails is machine learning. The key distinction is that the system's behaviour comes from the data, not from explicit instructions written by a programmer.

**Deep learning** is a subset of ML that uses a specific architecture: neural networks with many layers. "Deep" refers to the depth of those layers, not to any philosophical profundity. Deep learning is responsible for most of the dramatic progress in AI since roughly 2012 — image recognition, speech-to-text, language models, and generative AI all run on deep learning architectures.

Think of it as nested circles:

```
AI
└── Machine Learning
    └── Deep Learning
        └── (Large Language Models, diffusion models, etc.)
```

When people say "AI" in a 2024 context, they almost always mean deep learning, and usually large language models specifically. But the broader terms still matter — not everything is deep learning, and understanding the hierarchy helps you read the field accurately.

---

## What is a model?

In everyday language, a "model" is a simplified representation of something — a model aeroplane, a model of the economy. In ML, the meaning is similar but more precise.

A **model** is a mathematical function that takes an input and produces an output. That's it. The interesting part is how that function is defined — not by a programmer writing `if/else` rules, but by a training process that adjusts millions (or billions) of numerical values until the function reliably produces the right kind of output.

A language model takes text as input and produces text (or a probability distribution over possible next tokens) as output. An image classifier takes an image as input and produces a label as output. A recommendation model takes a user's history as input and produces a ranked list of items as output.

When someone says "the model got it wrong" or "we deployed the model", they mean this mathematical function — along with all the numerical values that define its behaviour.

---

## Parameters and weights

A model's behaviour is determined by its **parameters** — sometimes called **weights**. These are the numerical values that the training process adjusts.

A simple analogy: imagine you're tuning a mixing board with thousands of dials. Each dial is a parameter. Training is the process of figuring out where to set each dial so the output sounds right. A large model has billions of dials.

When you hear "GPT-4 has hundreds of billions of parameters" or "this is a 7B model", the number refers to how many of these adjustable values the model contains. More parameters generally means more capacity to learn complex patterns — but also more compute required to train and run it.

**Weights** and **parameters** mean the same thing in most contexts. Some literature distinguishes them (weights refer specifically to the connections in a neural network, parameters is the broader term), but in practice they're used interchangeably.

---

## Training

**Training** is the process of adjusting a model's parameters so that it produces good outputs. It works roughly like this:

1. Show the model an input.
2. The model produces an output.
3. Compare that output to the correct answer (the label or target).
4. Calculate how wrong the output was — this is the **loss**.
5. Adjust the parameters slightly in the direction that would have reduced the loss.
6. Repeat millions or billions of times.

This process — called **gradient descent** — is covered in depth in Phase 2. For now, what matters is the concept: training is an iterative numerical optimisation process, not something mystical or biological.

After training, the parameters are fixed. The model "knows" what it learned and won't change unless you train it again.

A few related terms you'll encounter:

**Training data** — the examples the model learns from. The quality and quantity of training data is one of the most important factors in model quality.

**Epoch** — one full pass through the training dataset. Models are typically trained for multiple epochs.

**Checkpoint** — a saved snapshot of a model's parameters at a particular point during training. If training crashes, you can resume from the last checkpoint rather than starting over.

**Pre-training** — training a model from scratch on a large general dataset. For language models, this means reading an enormous amount of text from the internet, books, code, and other sources.

**Fine-tuning** — taking a pre-trained model and training it further on a smaller, more specific dataset to adapt its behaviour. A model pre-trained on general text might be fine-tuned on customer service conversations to make it better at that specific task.

---

## Inference

**Inference** is when you use a trained model to produce outputs — as opposed to training, which is when you're adjusting the model's parameters.

When you type a message to an AI assistant and it responds, that's inference. The model's parameters are frozen; it's just running your input through the mathematical function to produce a response.

The distinction matters for a few reasons:

- Training is expensive and slow. Inference is cheaper and fast (relatively).
- You train a model once (or periodically). You run inference constantly — every user request is an inference call.
- Training requires storing gradients and intermediate values, which uses a lot of memory. Inference is lighter.

In production systems, when people talk about "serving" a model or "deploying" a model, they mean setting up infrastructure to handle inference requests at scale.

---

## Datasets

A **dataset** is a collection of examples used to train, evaluate, or test a model.

Datasets come in a few varieties:

**Labelled datasets** have inputs paired with correct outputs. An image dataset might contain photos labelled with what's in them ("cat", "dog", "car"). These are used for **supervised learning** — training where the model learns from explicit correct answers.

**Unlabelled datasets** have inputs but no labels. Most of the text on the internet is unlabelled. Language models learn from unlabelled data using a technique called **self-supervised learning** — the model predicts what comes next in a sequence, which creates implicit labels from the data itself.

**Benchmark datasets** are standardised datasets used to compare model performance. If someone says "our model achieved 90% on ImageNet", they're reporting performance on a specific well-known benchmark dataset. Benchmarks allow the field to track progress over time.

The three standard splits of a dataset:

- **Training set** — what the model learns from
- **Validation set** — used during training to check progress and tune settings, but not trained on directly
- **Test set** — held out entirely until evaluation; gives a true measure of performance on unseen data

A model that performs well on the training set but poorly on the test set has **overfit** — it memorised the training examples rather than learning general patterns.

---

## Generative AI

**Generative AI** refers to models that produce new content — text, images, audio, code, video — rather than just classifying or predicting from existing inputs.

A traditional classifier takes an image and outputs "cat" or "dog". A generative model takes a text prompt ("a cat sitting on a moon") and outputs a new image.

Generative models learn the underlying distribution of their training data well enough to produce new samples that plausibly came from that distribution. A language model trained on human writing produces text that reads like human writing. An image model trained on photos produces images that look like photos.

The term became mainstream with the launch of ChatGPT in late 2022, but the underlying techniques — particularly **diffusion models** for images and **transformers** for text — had been developing for years before that.

---

## Large Language Models (LLMs)

A **large language model** is a generative model trained on text at massive scale. "Large" refers to the number of parameters (billions to trillions) and the scale of training data (hundreds of billions to trillions of tokens).

LLMs are trained to predict the next token in a sequence. Through doing this at enormous scale, they develop capabilities that weren't explicitly trained: reasoning, summarisation, translation, code generation, question answering, and more. This emergent capability from scale is one of the more surprising findings in the field.

The "language" in LLM is slightly misleading — modern LLMs also handle images, audio, and other modalities. But they originated as text models, and text remains their primary medium.

Examples: GPT-4, Claude, Gemini, Llama, Mistral.

---

## Tokens

Models don't process text as words or characters — they process **tokens**.

A token is a chunk of text that the model treats as a single unit. Tokenisation splits input text into these chunks before it enters the model. The exact tokenisation scheme varies by model, but as a rough guide:

- Common English words are often a single token ("the", "model", "training")
- Longer or less common words may be split ("tokenisation" → "token" + "isation")
- Punctuation, spaces, and special characters are their own tokens
- For English text, a token is roughly 3–4 characters, or about 0.75 words on average

Tokens matter practically because:

- API pricing is based on tokens consumed (input and output)
- Models have a **context window** — a maximum number of tokens they can process at once
- Different languages tokenise differently; the same content in a non-English language often uses more tokens than in English

---

## Context window

The **context window** is the maximum amount of text a model can "see" at once, measured in tokens.

When you have a conversation with an AI assistant, everything the model considers — the system prompt, all previous messages, your current message, and its response — must fit within the context window. Anything outside the window doesn't exist to the model.

This is a hard constraint. A model with a 100,000-token context window cannot consider a conversation longer than that, no matter how important earlier parts were. Techniques like **RAG** (covered in Phase 5) exist partly to work around this limitation.

Context windows have grown dramatically — from 4,096 tokens a few years ago to over 1 million tokens for some models today. Larger context windows are generally better, but they're also more expensive to process.

---

## Embeddings

An **embedding** is a numerical representation of something — a word, a sentence, an image, a user — as a point in high-dimensional space.

The core idea: things that are semantically similar should be close together in this space. The embedding for "cat" should be near the embedding for "kitten", further from "dog", and very far from "turbine".

Embeddings allow models (and search systems) to work with meaning rather than just exact string matches. They're foundational to retrieval systems, recommendation engines, and how language models understand relationships between concepts.

In practice, embeddings are just arrays of floating-point numbers. A sentence embedding might be a list of 1,536 numbers. The numbers themselves aren't interpretable — it's their geometric relationships that carry meaning.

---

## Prompt

A **prompt** is the input you give to a language model. Everything you type into an AI assistant is a prompt.

The term has spawned its own discipline: **prompt engineering** is the practice of crafting inputs to elicit better, more accurate, or more useful outputs. Because LLMs are sensitive to how a question is phrased — and because they can follow instructions within the prompt itself — how you write a prompt meaningfully affects the output you get.

A **system prompt** is a special prompt that shapes the model's behaviour for an entire conversation, typically set by the application developer rather than the end user. It might instruct the model to respond in a certain style, focus on certain topics, or take on a specific persona.

---

## Hallucination

**Hallucination** is when a model produces output that is confidently stated but factually wrong or entirely fabricated.

A model might invent citations to papers that don't exist, state false historical dates, or describe events that never happened — all with the same fluent confidence it uses for correct statements.

Hallucination happens because LLMs are trained to produce plausible-sounding text, not to retrieve verified facts. The model doesn't "know" what it knows — it generates tokens that fit the statistical patterns of its training data. When those patterns lead it astray, it produces convincing nonsense.

Hallucination is not a bug that gets fixed in the next version. It is a fundamental property of how current LLMs work. Mitigation strategies exist (retrieval augmentation, fine-tuning, output verification), but it cannot be entirely eliminated without architectural changes that don't yet exist.

---

## Summary: the core terms

| Term | One-line definition |
|------|-------------------|
| AI | Any technique that enables machines to mimic intelligent behaviour |
| Machine learning | AI where the system learns rules from data rather than being programmed with them |
| Deep learning | ML using multi-layer neural networks |
| Model | A mathematical function mapping inputs to outputs, defined by its parameters |
| Parameters / weights | The numerical values that define a model's behaviour |
| Training | The process of adjusting parameters to improve model outputs |
| Inference | Using a trained model to produce outputs |
| Dataset | A collection of examples used to train or evaluate a model |
| Generative AI | Models that produce new content (text, images, etc.) |
| LLM | A large-scale generative model trained on text |
| Token | The basic unit of text a model processes |
| Context window | The maximum amount of text a model can consider at once |
| Embedding | A numerical representation of meaning as a point in high-dimensional space |
| Prompt | The input given to a language model |
| Hallucination | Confidently stated but incorrect or fabricated model output |

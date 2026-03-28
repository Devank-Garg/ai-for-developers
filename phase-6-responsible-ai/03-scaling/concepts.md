# Concepts: Scaling

> **Tag: core** — Covers what changes as AI systems grow larger: capability jumps, infrastructure concentration, competitive dynamics, and governance challenges.

---

## Scaling laws

**What it is** — Scaling laws are empirical relationships between model size, training compute, dataset size, and model performance. They predict that capability improves in a predictable, power-law fashion as you scale each input.

**Why it matters** — If capability scales predictably, then future AI capability is, to a first approximation, a function of available compute and data. This makes AI development more like infrastructure buildout than research — and it explains why the race for compute is also a race for capability.

**The Kaplan et al. findings (2020)** — OpenAI researchers found:
```
Model performance (as measured by cross-entropy loss) scales as:
  L(N) ∝ N^(-0.076)    (performance vs. number of parameters)
  L(C) ∝ C^(-0.050)    (performance vs. compute FLOPs)
  L(D) ∝ D^(-0.095)    (performance vs. dataset size)

Translation: doubling compute improves performance by roughly 5%.
             Doubling model size improves performance by ~7%.
```

**Chinchilla scaling (2022)** — DeepMind's refinement showed that Kaplan's models were undertrained. For a given compute budget, the optimal strategy is to train a smaller model on more data (roughly: for every doubling of model size, double the training data). This produced Chinchilla (70B parameters) outperforming Gopher (280B) at the same compute cost.

**The reassuring implication** — Capability gains are predictable; you can plan for them.

**The alarming implication** — Capability gains are predictable; others can also plan for them, including actors with different values or governance constraints.

---

## Emergent capabilities

**What it is** — Emergent capabilities are abilities that appear in AI models above a certain scale threshold but are absent or near-zero below it. They cannot be predicted by extrapolating from smaller model performance.

**Why it matters** — If capability were linear, you could evaluate a small model and predict a large model's behaviour. Emergence breaks this. A model that fails completely at a task at 10B parameters may succeed reliably at 100B parameters. You cannot test for capabilities you don't know to look for.

**Documented examples:**
```
Few-shot in-context learning
  — Appeared in GPT-3 (175B parameters)
  — Models below ~10B could not reliably learn new tasks from examples in context
  — Above this threshold, the ability appeared abruptly

Multi-step arithmetic
  — Simple arithmetic (3-digit addition) was near-chance at small scales
  — Appeared reliably above ~50B parameters without explicit training

Chain-of-thought reasoning
  — The ability to follow step-by-step reasoning in context
  — Appeared abruptly at larger scales; prompting small models with CoT actually hurt performance

Code execution simulation
  — Predicting what a code snippet would output without running it
  — Not present in smaller models; appears at ~100B+

Theory of mind tasks
  — Tasks requiring reasoning about what another agent believes
  — Appeared at GPT-4 scale in some analyses
```

**The BIG-Bench study (2022)** — Google researchers evaluated 150+ tasks across models from 400M to 540B parameters. About 30% showed clear emergence patterns — near-chance performance that suddenly jumped at a threshold. The threshold varied by task.

---

## Capability unpredictability

**What it is** — Despite scaling laws predicting aggregate performance, the specific capabilities that emerge and when they emerge cannot be reliably predicted before training large models. Labs often discover what their models can do by evaluating them after training.

**Why it matters** — If you cannot predict capabilities, you cannot evaluate safety before training. Safety measures applied to current models may not address the risks of future models. The safety research community is perpetually catching up to capability developments.

**The GPT-4 case** — OpenAI conducted capability evaluations *after* training GPT-4. They found, post-hoc, that GPT-4 could assist with certain dual-use tasks at higher levels than GPT-3.5. The safety measures were applied after discovering the capability, not in anticipation of it.

**Why evals are retrospective** — Capability evals require knowing what to test. If you don't anticipate a capability, you don't include it in your eval suite. Emergent capabilities are, almost by definition, ones you didn't anticipate — which means they won't be in your pre-deployment eval suite.

**The developer implication** — When you upgrade the model underlying your product, treat it as a new system. Run a full eval. Assume it has capabilities the previous version did not, some of which may create risks in your specific deployment context.

---

## Infrastructure concentration

**What it is** — Training frontier AI models requires compute infrastructure that is controlled by a very small number of organisations. As of 2025, three hyperscalers (Google, Microsoft/Azure, Amazon) control the vast majority of cloud GPU capacity. Five to six labs (OpenAI, Anthropic, Google DeepMind, Meta, Mistral, a few others) train frontier-scale models.

**Why it matters** — This level of concentration has no historical precedent in a technology of this societal significance. Concentration of compute means concentration of the ability to build frontier AI, which means concentration of the ability to shape what frontier AI does.

**The compute numbers:**
```
GPT-3 training: ~3.14 × 10^23 FLOPs
GPT-4 training: estimated ~2 × 10^25 FLOPs (unofficial)
Frontier models (2025): ~10^25–10^26 FLOPs

Cost to train a frontier model: $50M–$500M+
Number of organisations that can afford this: < 10 worldwide
```

**Implications:**
- Regulatory capture is possible: companies large enough to train frontier models are large enough to shape the regulations governing them
- Barriers to entry are enormous: academic researchers and startups cannot train at frontier scale
- Dependence: even large technology companies (Microsoft, Apple, Salesforce) build on models trained by others, creating dependencies and potential single points of failure

**For developers** — Your product depends on infrastructure you don't control. If your API provider changes pricing, terms, capabilities, or shuts down a model, your product changes with it. This is operational risk, not just governance concern.

---

## Compute governance

**What it is** — Compute governance is the set of policies, regulations, and agreements that control who can access the compute needed to train and deploy frontier AI systems. It has become a primary instrument of AI policy because it addresses a choke point that both capabilities and safety concerns pass through.

**Why it matters** — Because compute is the key input to frontier AI, controlling compute is one of the few levers that can actually slow or redirect AI development. It has also become a major axis of geopolitical competition.

**US export controls** — Since October 2022, the US has restricted export of high-performance AI chips (primarily NVIDIA A100, H100, and successors) to China. The restrictions have been tightened multiple times. The rationale: prevent China from training frontier AI models that could be applied to military and surveillance applications.

**The chip restrictions in practice:**
```
Restricted: NVIDIA H100, A100, and future equivalents above thresholds
Allowed: lower-performance chips that can still train useful models
Effect: slows but does not prevent Chinese frontier AI development

Huawei Ascend chips: domestic alternative; performance gap is narrowing
Cloud compute workarounds: cloud providers in third countries remain accessible
```

**Compute monitoring proposals** — Several researchers and policy organisations have proposed mandatory reporting of large training runs (above a compute threshold, say 10^26 FLOPs), analogous to financial transaction monitoring. No major jurisdiction has implemented this as of 2025.

**The developer irrelevance / relevance** — Most developers don't train frontier models. But compute governance shapes which models are available to you, at what cost, from which providers, with what capabilities. It is part of the infrastructure context you operate in.

---

## Environmental cost

**What it is** — Training and running AI models at scale consumes substantial energy and produces carbon emissions. The cost has grown significantly as model sizes and usage volumes have increased.

**Why it matters** — AI is frequently positioned as a tool to help solve climate change. The environmental cost of AI itself is underreported and poorly understood by most people building with it.

**Training costs (estimates):**
```
GPT-3 (2020): ~500 tonnes CO₂e — equivalent to ~100 transatlantic flights
GPT-4 (estimated): multiple times GPT-3, exact figures not published
LLaMA 2 70B: ~539 tonnes CO₂e (Meta published this)
```

**Inference is the larger story** — Training happens once. Inference happens continuously, at scale. A model answering millions of queries per day consumes orders of magnitude more energy over its lifetime than its training run.

```
Estimated energy per query:
  — Google Search: ~0.0003 kWh
  — ChatGPT query: ~0.001–0.01 kWh (10–100× more than search)

At 10 million queries/day:
  — 10,000–100,000 kWh/day
  — 3.6M–36M kWh/year
  — ~1,500–15,000 tonnes CO₂e/year (depending on grid mix)
```

**The data centre buildout** — AI demand has driven massive data centre construction, associated water consumption for cooling, and increased pressure on electrical grids. Microsoft, Google, and Amazon have all reported that AI demand is making it difficult to meet their own net-zero commitments.

**What developers can do:**
- Choose smaller models where they are sufficient; not every task needs a frontier model
- Be aware of inference costs as an environmental metric alongside financial cost
- Consider the geographic location of compute (grid carbon intensity varies 10× between regions)

---

## Race dynamics

**What it is** — Race dynamics describe the competitive incentives that push AI labs to prioritise speed over safety. The structure is a classic collective action problem: if you slow down for safety, your competitor ships first. If everyone slows down together, everyone is safer but no single actor benefits. Without coordination, the dominant strategy is to ship fast.

**Why it matters** — Safety failures are not usually the result of negligence or malice. They are often the predictable result of rational actors responding to competitive incentives. Understanding the structure of the race helps you understand why "just be more careful" is not sufficient as a policy response.

**The payoff structure:**
```
If you ship first:
  — Capture market share
  — Set industry norms
  — Attract talent and investment
  — Define what "good enough" safety looks like

If you ship second:
  — Compete against an entrenched leader
  — The market has moved on
  — Investors ask why you were slow

Safety investment cost:
  — Delays launch by weeks or months
  — May require capability limitations
  — Competes with engineering time
```

**The result** — Labs invest in safety research publicly (good for reputation, good for talent acquisition) while facing internal pressure to ship before competitors. Safety teams are often structurally subordinate to product and research teams. Safety thresholds are set at levels compatible with shipping, not at levels set by external safety researchers.

**The exception** — Safety as a competitive differentiator: if users or enterprise customers require safety guarantees, safety investment has a direct business return. This is the mechanism by which enterprise AI adoption creates safety incentives.

---

## Open vs. closed models

**What it is** — The debate over whether AI model weights should be publicly released (open) or kept private (closed). This is not a technical debate — it is a genuine disagreement about risk, access, democratic governance, and who should control frontier AI.

**Why it matters** — How this debate resolves will shape who can build AI applications, what safety measures are enforceable, and who benefits from AI economically. It is not settled.

**The case for open models:**
- Democratises access: researchers, startups, and developers in lower-income countries can build without API dependency
- Enables auditability: anyone can inspect, evaluate, and red-team open weights
- Prevents monopoly: concentration of closed models creates dangerous dependencies
- Accelerates safety research: safety researchers need access to model internals
- Historical precedent: open-source software has generally improved security, not reduced it

**The case for closed models:**
- Frontier capabilities are dual-use: releasing weights for a model that can assist in bioweapon synthesis gives that capability to everyone permanently
- Safety measures applied at the API layer (rate limits, content filtering, monitoring) are not possible with open weights
- Responsible disclosure: labs have more information about capabilities than the public and can make more informed decisions about release timing
- Open models can be fine-tuned to remove safety guardrails: a model released with safety fine-tuning can be stripped of it by anyone with a GPU

**The current landscape (2025):**
- Meta releases Llama weights (up to 405B parameters) publicly
- Mistral releases Mixtral weights
- OpenAI, Anthropic, Google DeepMind do not release weights for frontier models
- Most AI safety researchers are split on the question

---

## Economic concentration

**What it is** — Economic concentration in AI refers to the tendency for AI markets to consolidate around a small number of dominant players, with winner-takes-most dynamics reinforced by data network effects, compute advantages, and talent concentration.

**Why it matters** — Economic concentration in previous technology markets (search, social media, e-commerce) produced regulatory challenges, reduced competition, and significant political power for platform operators. AI has similar concentration dynamics with potentially greater societal leverage.

**The reinforcing cycle:**
```
More users → more data → better model → more users
More revenue → more compute → better model → more revenue
Better model → attracts top talent → even better model

Entry barriers:
  — Training cost: $50M–$500M+
  — Data: decades of proprietary data have network effects
  — Talent: the pool of people who can train frontier models is small
```

**API lock-in** — Applications built on top of a specific model API develop prompt engineering, fine-tuning, and evaluation infrastructure that is model-specific. Switching has significant cost. This creates switching costs analogous to enterprise software lock-in but with less standardisation.

**The non-AI-company winner** — NVIDIA: the manufacturer of GPUs that train virtually all frontier models. In 2024, NVIDIA's market capitalisation briefly exceeded $3 trillion. Their moat is the CUDA software ecosystem, not hardware alone — alternative chips require rewriting the software stack that the entire AI industry depends on.

**For developers** — Understand your dependency graph. Your application depends on an API, which depends on a model, which was trained on a cloud, which runs on specific hardware. Each layer has concentration risks. Architectural choices that reduce switching cost (prompt abstraction layers, model-agnostic evals) are a business risk management strategy, not just good engineering practice.

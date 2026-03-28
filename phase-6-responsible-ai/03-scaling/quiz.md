# Quiz: Scaling

Attempt these before checking answers. These questions test your understanding of what changes as AI systems scale — and what that means for the decisions you make as a developer.

---

## Questions

**1.** A startup founder says: "Our product uses GPT-3.5 today. We'll upgrade to GPT-5 when it's available — it'll just work better." What scaling-related risks is this founder not thinking about? List three, and for each, describe a concrete scenario where it causes a problem.

**2.** Explain the difference between scaling laws and emergent capabilities. Why does emergence mean that evaluating a small model is not sufficient to predict the safety properties of a larger model?

**3.** The Chinchilla paper showed that many large models (including GPT-3) were "undertrained" — better performance was achievable by using the same compute budget for a smaller model with more data. What does this tell you about how capability improvements happen? And what does it imply for anyone trying to predict when the next capability jump will occur?

**4.** A researcher at a frontier lab argues: "We should open-source our new model's weights so the research community can audit it for safety issues." Her colleague argues: "Open-sourcing a model that can assist with certain dangerous capabilities means we can never take it back — and someone will fine-tune away the safety guardrails." Steelman both positions. Which is more compelling for a model that (a) has only consumer-level capabilities vs (b) can provide meaningful assistance with bioweapon synthesis?

**5.** Explain the race dynamic problem in AI development as a collective action problem. Why is "just be more careful" an insufficient solution? What coordination mechanism would actually address the incentive structure?

**6.** A company wants to train a large language model but can only afford 10^23 FLOPs. According to Kaplan scaling laws, roughly how does this compare to GPT-3 in expected capability? What does the Chinchilla refinement tell you about how to allocate that budget?

**7.** NVIDIA's market valuation briefly exceeded $3 trillion in 2024, despite AMD and other chip manufacturers offering competitive hardware. Explain this in terms of economic concentration — specifically, what NVIDIA's moat is, and why it is harder to dislodge than hardware specifications alone suggest.

**8.** A government proposes mandatory reporting of any AI training run above 10^26 FLOPs. A tech company argues this is unworkable and unnecessary. A safety researcher argues it is essential. Describe the strongest argument for each position, and describe what information the reporting requirement would actually produce.

**9.** Your company's AI feature uses GPT-4. OpenAI announces that GPT-4 will be deprecated in 6 months and replaced by a model with significantly expanded capabilities. Describe the technical and business risks this creates, and what steps you would take to prepare.

**10.** Explain why the environmental cost of AI inference may matter more than the environmental cost of training, even though training is the part that gets reported. Use concrete numbers or estimates to make your case.

**11.** A developer argues: "I use a small open-source model for my application, so I'm not part of the infrastructure concentration problem." Is this correct? What concentration risks does the developer still face, and which ones are they insulated from?

---

## Answers

**1.** Three scaling-related risks the founder isn't thinking about:

*Emergent capabilities:* GPT-5 may have capabilities that GPT-3.5 does not. Some of those capabilities may interact with the founder's product in ways that are harmful or that violate their terms of service. Scenario: an AI writing assistant that was harmless with GPT-3.5 suddenly generates content that, with GPT-5's better understanding of context, is far more persuasive and potentially manipulative.

*Changed behaviour on existing prompts:* A more capable model does not guarantee the same outputs on the same prompts. Prompt engineering, system prompts, and fine-tuning calibrated to GPT-3.5 may need to be completely reworked for GPT-5. Scenario: the startup's carefully crafted system prompt that produced reliable JSON output from GPT-3.5 now produces verbose explanations because GPT-5 interprets it differently.

*Infrastructure and API dependency shift:* Upgrading means committing to new pricing, rate limits, and terms. GPT-5 may have different context window limits, different token costs, or different acceptable use policies. Scenario: the upgrade doubles inference costs, making the product's unit economics unviable. Or GPT-5's expanded capabilities mean the product now falls into a newly prohibited use category.

---

**2.** Scaling laws describe predictable, power-law relationships between compute/data/parameters and aggregate performance (measured as loss on a benchmark). They allow you to predict that a model trained with 10× more compute will be roughly X% better at predicting the next token.

Emergence is a different phenomenon: specific capabilities that are essentially absent below a threshold and appear reliably above it. Emergence is not captured by aggregate loss metrics — a model that is only slightly better on average can suddenly succeed on a task it could not complete at all before.

Why small-model evaluation is insufficient for safety: a safety evaluation requires knowing what capabilities to test for. Emergent capabilities, by definition, are ones that didn't exist in smaller models. If you evaluate a small model and find no dangerous capabilities, you have no guarantee that the larger model won't have them. The safety properties of a model cannot be inferred from the safety properties of smaller models in its lineage.

---

**3.** The Chinchilla finding tells you that the relationship between compute, model size, and training data must all be optimised together — you cannot maximise capability by just scaling model size. It also tells you that apparent capability plateaus may not be real capability plateaus — they may be the result of suboptimal resource allocation.

Implication for predicting capability jumps: they become harder to predict, not easier. A lab with the same compute budget as a current frontier lab can achieve significantly better performance by training smarter. This means capability improvements can come from algorithmic and data efficiency gains, not just raw compute scaling. Someone could train a dramatically more capable model on existing hardware by finding a better scaling recipe. This makes the timeline of the "next capability jump" uncertain — it could come from new compute, from better data, or from a smarter training recipe that is not publicly known.

---

**4.** Steelman for open-sourcing:
Open weights enable independent safety researchers to inspect the model, run their own evaluations, and identify risks that the lab may have missed or downplayed. The alternative — trusting the lab's own safety assessment — creates a conflict of interest. Historical precedent in security shows that public scrutiny improves security outcomes. Responsible disclosure by a single party is less robust than an open research community.

Steelman for keeping it closed:
Some capabilities cannot be un-released. If a model can provide non-trivial uplift to someone attempting to create a bioweapon, releasing the weights means that capability is permanently available to anyone with a GPU. Safety fine-tuning can be stripped. Content filters cannot be applied to open weights. The cost-benefit calculation depends entirely on what the model can do.

Which is more compelling:
- For a consumer-capability model: open-sourcing is likely net-positive. The capabilities are not highly dangerous, auditability has real value, and the concentration risk from closed models is significant.
- For a model with meaningful bioweapon synthesis assistance: the case against open release is much stronger. Permanent, irreversible capability proliferation to actors with malicious intent is not outweighed by research community auditability benefits. The asymmetry of harm is too large.

---

**5.** Race dynamics as a collective action problem:

The structure is a prisoner's dilemma at industry scale. If all labs slow down together for safety, everyone is safer and the competitive position of each lab is unchanged. But if you slow down and your competitor doesn't, you lose market share, talent, and investment without the safety benefit materialising (because the competitor ships the less-safe version). The dominant strategy for any individual actor is to ship as fast as possible.

"Be more careful" fails because it asks individual actors to absorb a competitive cost for a collective benefit. Without enforcement, rational actors defect.

Coordination mechanisms that would actually address this:
- Binding international agreements (analogous to nuclear non-proliferation) that set minimum safety standards above which competition continues
- Regulatory requirements with teeth: governments can make safety measures legally required, eliminating the competitive cost (everyone has to do them)
- Industry auditing bodies with enforcement power that apply uniform standards across labs
- Customer requirements: if large enterprise customers require safety certifications, labs have a revenue incentive to meet them

None of these are currently sufficient. The most promising near-term mechanism is government regulation, which is why the EU AI Act and US executive orders matter despite their limitations.

---

**6.** GPT-3 was trained with approximately 3.14 × 10^23 FLOPs.

At 10^23 FLOPs, you have roughly 30% of GPT-3's compute budget. Using Kaplan scaling: loss scales roughly as C^(-0.050), so 30% compute → loss is approximately 0.30^(-0.050) ≈ 1.06× worse than GPT-3 (a ~6% higher loss). In practice, this translates to noticeably worse performance on complex reasoning tasks.

Chinchilla allocation: with 10^23 FLOPs, the Chinchilla rule suggests training a smaller model (roughly 5B–10B parameters) on proportionally more tokens (~200B–500B), rather than training a 175B parameter model on fewer tokens. This produces better performance than the Kaplan-optimal allocation at the same compute budget.

---

**7.** NVIDIA's moat is CUDA — the software ecosystem, not the hardware. CUDA is a parallel computing platform and API that has been developed since 2007. It is the standard interface that PyTorch, TensorFlow, and virtually every AI framework uses to communicate with GPUs. The entire AI software stack is built on CUDA abstractions.

Running AI workloads on AMD (ROCm) or Intel (oneAPI) chips requires either rewriting the software stack or using compatibility layers that introduce performance penalties. The switching cost is not "buy different hardware" — it is "rewrite years of software infrastructure and retrain your entire engineering team." No individual company can absorb this cost, and the collective action problem prevents the industry from switching together.

Hardware specifications are therefore insufficient to predict competitive dynamics. AMD's MI300X may match or exceed H100 performance in raw FLOPs; it still cannot displace NVIDIA without first displacing CUDA, which requires displacing PyTorch's CUDA dependency, which requires displacing the research community's decade-long investment in CUDA tooling.

---

**8.** Strongest argument for mandatory compute reporting:
Training runs above 10^26 FLOPs represent potential frontier capability thresholds. Governments have a legitimate interest in knowing when new frontier capabilities are being created, particularly for national security and safety assessment purposes. Compute reporting provides the minimal information needed for regulatory oversight without requiring disclosure of model weights, training data, or proprietary methods. The precedent is financial transaction reporting: large transactions are reported not because they are all suspicious but to provide oversight capability.

Strongest argument against:
Compute thresholds are a poor proxy for capability. A 10^26 FLOP run might produce a highly capable model or a poorly designed one; the FLOPs alone don't tell you. The threshold is also a target: labs would find ways to distribute training runs below the threshold, reducing the information produced. The administrative burden of defining, measuring, and enforcing FLOPs thresholds across jurisdictions is enormous. And the information produced — "a big training run happened" — is not obviously actionable without also accessing model weights and capabilities.

What the reporting would actually produce: a list of organisations conducting large training runs, the dates, and the approximate scale. This would allow regulators to identify actors not previously known to be training frontier models, and to initiate capability evaluations on the resulting models. It provides auditability, not capability control.

---

**9.** Technical and business risks from model deprecation:

Technical:
- Prompt engineering rework: your current prompts were calibrated to GPT-4's output patterns; the new model will respond differently to the same prompts
- Output format changes: if you rely on specific output structure, the new model may format differently
- New capabilities may change behaviour in unexpected ways (emergent capability risk)
- Evaluation infrastructure needs to be re-run for the new model

Business:
- Cost uncertainty: GPT-5 pricing is unknown; your unit economics may shift
- New acceptable use policies: the new model may add restrictions affecting your use case
- Dependency on a single provider: if GPT-5 has problems at launch, your product goes down

Preparation steps:
1. Build a regression test suite now, while you know what correct behaviour looks like on GPT-4 — this is your baseline for evaluating the new model
2. Abstract your model API calls behind an interface so switching models requires changing one layer, not rewriting the application
3. Monitor the deprecation announcement for GPT-5 capabilities and policies; evaluate your use case against the new terms before migration
4. Consider running both models in parallel for a period and comparing outputs before fully migrating

---

**10.** Training vs. inference environmental cost:

Training happens once. Inference happens continuously.

Example calculation for a hypothetical large-scale deployment:
- Training GPT-3: ~500 tonnes CO₂e (one-time)
- A service running at 10M queries/day at 0.005 kWh/query: 50,000 kWh/day
  - At global average carbon intensity (400g CO₂e/kWh): 20 tonnes CO₂e/day
  - Per year: ~7,300 tonnes CO₂e — 14× the training cost, every year

At scale (ChatGPT reported 100M users in January 2023), this becomes:
- Conservative: hundreds of thousands of tonnes CO₂e per year just for inference
- Compared to: the one-time training cost that gets reported in papers

Training numbers get published (and thus get reported). Inference numbers are distributed across millions of users, hidden in data centre power consumption, and never aggregated in a way that gets reported. This creates the impression that the training run is "the" environmental cost, when ongoing inference is substantially larger over the product's lifetime.

The developer implication: your choice of model size affects the ongoing environmental cost of your product, not just your API bill.

---

**11.** The developer is partially correct but not fully insulated.

Concentration risks they still face:
- *Cloud infrastructure:* the open-source model still runs on compute. If they're using a cloud provider (AWS, GCP, Azure), they remain dependent on the same hyperscaler concentration. Self-hosted on dedicated hardware is a partial escape, but most startups don't own physical servers.
- *Hardware:* the GPU they use is almost certainly NVIDIA. The CUDA dependency is the same.
- *Framework:* PyTorch, which runs on CUDA, is the dominant framework for open-source model inference. Same dependency chain, different entry point.

Concentration risks they're insulated from:
- *Model vendor lock-in:* they can switch to a different open-source model (Mistral, Llama, Falcon, etc.) without renegotiating an API contract. This is a real and significant advantage.
- *API pricing risk:* open weights run at infrastructure cost, not API pricing, which removes a layer of commercial risk.
- *Terms of service risk:* a model vendor changing acceptable use policies doesn't affect them in the same way.

The honest answer: open-source reduces model-layer concentration but leaves infrastructure-layer concentration largely intact.

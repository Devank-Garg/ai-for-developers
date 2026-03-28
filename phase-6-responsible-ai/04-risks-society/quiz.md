# Quiz: Risks & Society

Attempt these before checking answers. This final module's questions don't always have a single correct answer — they ask you to reason through genuine tradeoffs and uncertainty. That is intentional.

---

## Questions

**1.** You are a backend engineer at a startup building a hiring tool that uses LLMs to screen CVs. Identify three distinct societal risks your product creates. For each, name one concrete action you could take this quarter to reduce that risk — not eliminate it, but reduce it.

**2.** Explain why deepfake detection is structurally behind deepfake generation. Use the analogy of an arms race to make your case. Does the existence of C2PA (content provenance standards) change this dynamic, and if so, how?

**3.** A friend who works in media says: "AI misinformation is just like previous media misinformation — newspapers printed lies too." Is this correct? What is specifically different about LLM-generated misinformation compared to human-generated misinformation at scale?

**4.** Eliezer Yudkowsky argues that the development of AGI without solved alignment is most likely fatal for humanity. Yann LeCun argues that current AI is a useful tool, not an existential threat. Geoffrey Hinton says he is more worried than he used to be. Without dismissing any of them, explain: (a) what assumption underlies Yudkowsky's concern that LeCun rejects; and (b) what changed in Hinton's assessment.

**5.** A city government proposes deploying AI-powered facial recognition cameras in a high-crime neighbourhood for public safety. You are a developer asked to build the recognition system. Describe three distinct ethical concerns with this deployment. Then describe what changes to the deployment design (if any) would make it ethically acceptable to you — and if nothing would, say so.

**6.** The research evidence on job displacement suggests that AI primarily displaces tasks, not entire occupations. A sceptic responds: "That's just economists downplaying the harm — the people who lose those tasks still lose income." How do you evaluate both claims? What would it take for the task-displacement framing to be genuinely reassuring rather than technically correct but misleading?

**7.** A colleague says: "I'm building an open-source image generation model and releasing the weights. I'm not responsible for what people do with it — I'm just making a tool." Using the concepts in this module, assess this position. What is accurate in it? Where does it fall short?

**8.** Your company operates in both the EU and the US. Your AI product assists hiring managers in candidate selection. Describe the regulatory requirements that apply in each jurisdiction, and identify the most significant compliance gap between what you currently do and what you should do.

**9.** A tech journalist writes: "AI is heading for an AI winter — the hype is worse than it was in 1999, and the valuations are just as disconnected from reality." Evaluate this claim: what evidence supports it, what evidence contradicts it, and what would a genuine AI winter look like in 2025–2030?

**10.** You build and ship a consumer-facing mental health chatbot. Three months after launch, a user's family member contacts you to say the user experienced a mental health crisis after extended interactions with your chatbot. You have no evidence the chatbot caused the crisis. Describe what your response should be in the next 24 hours, the next week, and the next month. What changes to the product, if any, would you make?

**11.** Explain the concept of a digital divide in AI. Give two concrete examples of how AI amplifies existing advantages for well-resourced groups while providing less benefit to underserved groups. As a developer, describe two product decisions that would reduce (not eliminate) this gap.

**12.** At the end of this course, you are building AI features into production systems. In one paragraph, describe what "responsible AI developer" means to you in practice — not as abstract principle, but as a set of habits and decisions you will actually make.

---

## Answers

**1.** Three societal risks and mitigations for an AI hiring tool:

*Bias amplification:* the model may learn hiring patterns from training data that reflect historical discrimination, systematically disadvantaging candidates from underrepresented groups even on equal qualifications. This quarter: run disparate impact testing before launch — compare acceptance rates for equivalent CVs with different demographic signals (names, schools, employment gaps).

*Displacement of human judgment at scale:* when a hiring tool processes thousands of applications, it removes the human judgment that might have noticed an unusual career path or valued a non-traditional background. Scale amplifies whatever biases the system has. This quarter: define explicitly in your product that the tool surfaces candidates for human review and cannot be the sole basis for rejection; build this constraint into the UI, not just the documentation.

*Misuse beyond intended scope:* a tool built to screen CVs for a specific role may be used to screen for characteristics (age, disability, family status) that are protected. This quarter: add explicit guidance in the product on what it should not be used for, and add monitoring for downstream use patterns among enterprise customers.

---

**2.** Deepfake detection as an arms race:

Generation is a one-time optimisation: build a model that produces convincing content. Detection requires training on the outputs of current generation methods. When generation methods change (new architectures, new training approaches), detection classifiers trained on old generation methods fail and must be retrained.

Generation improves autonomously through AI research; detection requires responding to generation improvements. The detection team is always one step behind because it must first observe the new generation method to respond to it. Unlike a symmetric arms race, the asymmetry is structural: generation is the attack, detection is the defence, and in this domain the attack has inherent initiative.

C2PA (content provenance standards) partially changes this by moving from detection to verification. Instead of asking "does this look AI-generated?" C2PA asks "does this media have a cryptographic signature from a trusted source?" If your camera, recording software, or platform signs content at creation, downstream viewers can verify the chain of custody. This breaks the arms race by not playing on the same terrain — it doesn't try to detect fakes, it verifies authentics.

The limitation: C2PA only works when capture devices and platforms implement it, when the provenance chain is maintained through editing, and when consumers check signatures. Unsigned content doesn't prove anything about its authenticity — only that it wasn't signed. Adoption is incomplete, and the adversary simply produces content without a signature.

---

**3.** Newspapers did print lies. The specific differences with LLM-generated misinformation are:

*Scale:* A human journalist could write perhaps 5–10 articles per day. A single LLM pipeline can generate thousands of unique articles per day. This is not a quantitative difference that can be absorbed by existing moderation; it is a qualitative difference in the viability of detection and response.

*Cost:* Human misinformation campaigns require paying humans. LLM-generated content costs fractions of a cent per article after the API. This dramatically lowers the barrier to entry.

*Plausibility:* Historically, high-volume misinformation was identifiable by poor writing quality. LLM content can be indistinguishable from credible reporting in form, even when false in substance.

*Personalisation:* LLMs can adapt the same false claim to different audience segments (political framing, cultural references, language register). Personalised propaganda at scale was not previously possible.

*Attribution difficulty:* Generated content lacks the identifiable stylistic signatures that human writers develop and that have historically aided detection and attribution.

Your friend's analogy holds for the harm type (false beliefs) but not for the scale, cost, or tractability of response. The mitigation strategies that worked for print misinformation (journalistic norms, defamation law, source verification) are insufficient for LLM-scale content.

---

**4.** The core disagreement between Yudkowsky and LeCun:

*The assumption Yudkowsky holds that LeCun rejects:* Yudkowsky believes that sufficiently advanced AI systems will develop something analogous to goals or optimization targets that are not aligned with human values — and that such systems will pursue those goals in ways that are dangerous. He further believes that the path from current AI to transformative AGI is shorter than LeCun believes and that alignment is hard enough to be unlikely to be solved in time.

LeCun rejects the premise that current AI architectures are on a path to AGI as Yudkowsky conceives it. He believes current LLMs are sophisticated pattern matchers, not proto-agents pursuing goals. He does not see a credible path from next-word prediction to the kind of goal-directed agency that Yudkowsky's concerns require.

*What changed in Hinton's assessment:* Hinton spent his career building the foundations of deep learning and was historically sceptical of catastrophic risk scenarios. His shift came from observing the pace of capability improvements in large models — particularly the degree to which scale produced unexpected competencies. He now believes the probability of serious risk is non-trivial, not because he accepts Yudkowsky's specific mechanism, but because the pace of development has outrun his previous assumptions about when such risks would become relevant.

---

**5.** Three distinct ethical concerns:

*Accuracy disparity:* facial recognition has documented higher error rates for darker-skinned faces, particularly women. Deploying this system in a neighbourhood that is likely to have a higher proportion of residents from these groups means the people most likely to be falsely identified are the people who already have the most reason to distrust police interactions.

*Chilling effect on legitimate behaviour:* pervasive surveillance of public spaces changes behaviour. People attending legal protests, visiting medical facilities, or simply walking to work change their behaviour when they believe they are being identified and tracked. This is a harm to civil liberties even when no identification error occurs.

*Mission creep:* systems deployed for one purpose are routinely repurposed for others. A camera network installed for public safety can be used to track immigration status, political association, or religious practice. The scope of the system is defined by the most expansive use its operators choose to make.

What changes would make it acceptable: this is a genuine values question, not a technical one. Some developers would find acceptable: time-limited deployment with sunset provisions, restriction to specific crime categories by warrant, independent audit of accuracy by demographic group before deployment, prohibition on use for immigration enforcement. Others would find no configuration acceptable given the chilling effect and accuracy disparities. If you would not build it under any conditions, that is a legitimate position — state it as a decision rather than an inability.

---

**6.** Evaluating both claims:

The task-displacement framing is technically accurate: AI tends to eliminate specific tasks within jobs rather than eliminate jobs entirely. A radiologist who spends 20% of their time reviewing routine scans may have that task automated, but the remaining 80% of their job continues. The occupation does not disappear; the task does.

The sceptic is also correct: a person whose primary income was in that 20% of tasks faces real income loss. The "task, not occupation" framing can obscure this when the automated tasks are not peripheral but central — when a call centre worker has 70% of their calls automated, the remaining 30% may not provide viable income.

For the task-displacement framing to be genuinely reassuring (not just technically correct):
- The remaining tasks must be sufficient to support viable employment at similar compensation
- Transition from eliminated tasks to remaining tasks must be realistic (same geography, reachable with available retraining)
- The pace of displacement must match the pace at which people can transition
- The new jobs created by AI must be accessible to displaced workers (not just to the well-educated)

Currently, these conditions are partially but not fully met. The honest position: task displacement is less catastrophic than full occupation elimination, but it is not harmless, and the distributional impact on people without college education and in task-intensive roles is real.

---

**7.** Assessment of the open-source developer's position:

What is accurate:
- Legal responsibility for third-party misuse of a general-purpose tool is typically limited under most jurisdictions' product liability frameworks
- The developer cannot control what others do with released weights
- Open-source development has genuine social benefits: access, auditability, competition with concentrated proprietary systems

Where it falls short:
- The "just a tool" argument ignores reasonable foreseeability: if you can reasonably foresee that releasing an image generation model will enable non-consensual intimate imagery, that foreseeability is morally relevant even if it's not legally so in most jurisdictions
- Releasing model weights is irreversible in a way that API access is not: you can revoke API access; you cannot revoke released weights
- The developer has the unique power to implement safeguards at the model level (safety fine-tuning, built-in refusals) before release; once released, only users with the resources to fine-tune further can change this
- The argument that "someone else would release it anyway" is a rationalisation, not a justification

The responsible version: making a deliberate, documented decision about release — evaluating the expected benefits against expected harms, implementing available safeguards, and accepting that the developer retains some moral responsibility for foreseeable uses — rather than treating "I'm just releasing a tool" as a complete answer.

---

**8.** Regulatory requirements for an AI hiring tool:

EU AI Act: hiring tools are explicitly listed in Annex III as high-risk AI systems. Requirements include:
- Risk management system before deployment
- Data governance documentation
- Technical documentation (training data, model architecture, evaluation)
- Transparency to candidates: they must be informed that AI is used
- Human oversight: the system must support human review of decisions
- Accuracy and non-discrimination requirements
- Registration in the EU AI database
- Conformity assessment

GDPR (EU, also relevant): automated decisions with significant effects on individuals require:
- Disclosure that the decision is automated
- Right of the individual to request human review
- Right to explanation

US approach: no single comprehensive law. Sectoral:
- EEOC (Equal Employment Opportunity Commission): guidance on AI in hiring (2023) establishes that the employer is liable for discriminatory outcomes even when using a vendor's tool
- State laws: Illinois, Maryland, and others have specific AI hiring notification requirements

The most significant compliance gap in practice for most startups: no transparency to candidates (they don't know AI is used), no documentation of training data or evaluation methodology, and no disparate impact testing.

---

**9.** Evidence supporting the "AI winter" claim:
- Current AI company valuations are disconnected from near-term revenue in many cases
- Many AI product categories have poor unit economics (inference cost vs. revenue)
- Enterprise AI adoption is slower than expected in 2023–2024
- Several high-profile AI product failures (hallucination in legal, medical contexts)
- Consolidation: many smaller AI companies are shutting down or being acquired

Evidence contradicting it:
- Unlike 1990s AI, current AI is generating real revenue and is embedded in products people use daily
- The infrastructure buildout (data centres, GPUs) represents committed capital that requires use cases
- Capability improvement continues: GPT-4 to Claude 3 to Gemini to GPT-5 demonstrate ongoing progress
- Major platforms (Microsoft, Google, Apple) have integrated AI deeply enough that a "winter" would require walking back product decisions
- Government and enterprise procurement is growing, not shrinking

What a genuine AI winter in 2025–2030 would look like: not a complete halt, but a significant correction in private investment, a slowdown in frontier model training (due to cost and regulatory pressure), a period where capability improvement plateaus or slows, and a narrowing of deployed use cases to those with clear ROI. The technology would not disappear — but the "AI everywhere" narrative would recede, and some companies built entirely on AI hype would fail.

---

**10.** Response to a reported harm from a mental health chatbot:

24 hours:
- Acknowledge the contact. Do not deflect, deny, or ask them to submit a ticket. A human being responds, directly.
- Offer to review the conversation history (with the user's consent) to understand what happened.
- Do not admit liability, but do express genuine concern.
- If the user is still in crisis, provide emergency resources immediately.
- Alert your product team and legal counsel.

One week:
- Review the conversation logs (with consent) looking for: did the chatbot miss distress signals? Did it respond to explicit requests for help incorrectly? Did extended interaction patterns appear that correlated with declining wellbeing?
- Check whether similar patterns exist in other users' conversation histories.
- Make an honest assessment: does the available evidence suggest the product contributed to harm? Document this assessment.

One month:
- Publish a response if the harm is confirmed — to the family, internally, and potentially publicly (depending on scope)
- Product changes to consider: distress escalation protocols (detect crisis language and provide human referrals), session length limits, explicit disclosure that the chatbot is not a substitute for professional mental health care, clearer off-boarding to professional services
- Decide whether the product should continue to operate in this form, with modifications, or be discontinued

The most important principle: don't let the legal instinct to deny liability override the moral obligation to respond to harm.

---

**11.** The digital divide in AI:

Two concrete examples:
- *Language:* A software developer in Lagos who works primarily in English can use GPT-4 effectively for code generation and debugging. A software developer in Lagos who thinks and codes in Yoruba finds the same model significantly less useful — it hallucinates more in Yoruba, its cultural context is less relevant, and its training data underrepresents Yoruba technical vocabulary. The well-resourced developer (English-fluent, technically trained) gets a powerful productivity tool; the Yoruba-first developer gets an inconsistent one.

- *Education and prompting skill:* Getting high-quality outputs from LLMs requires knowing what to ask, how to ask it, and how to evaluate the response. These skills correlate strongly with education level and domain expertise. A lawyer using AI to draft contracts gets exponentially better results than a non-expert asking the same question — because the lawyer knows what a good contract looks like and can identify errors. The AI tool amplifies the lawyer's existing advantage; it doesn't democratise legal knowledge to the non-expert.

Two product decisions that reduce the gap:
- Support additional languages in your product, prioritised by underrepresentation relative to speaker population rather than market size. Swahili has more speakers than Swedish; most products support Swedish and not Swahili.
- Design for lower-skill users explicitly: provide example prompts, templates, and guided interfaces that reduce the requirement for prompt engineering expertise. If your product requires knowing how to write a good prompt to get a useful result, you've built a tool for the already-advantaged.

---

**12.** This question is genuinely yours to answer. The concepts are:

A responsible AI developer has a mental model of who their software affects, including people who don't use the software directly. They write down what their product is for and what it shouldn't be used for, before they ship. They test for failure modes, especially for users who are not in the majority demographic. They respond when harm is reported, rather than deflecting it. They have a small number of things they won't build, regardless of who pays for it, because they decided this in advance rather than in the moment of financial pressure.

None of this requires a title, a framework, a large team, or a philosophy degree. It requires treating "what could go wrong?" as a design question, not a legal formality.

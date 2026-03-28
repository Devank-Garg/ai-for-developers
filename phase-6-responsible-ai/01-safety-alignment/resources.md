# Resources: Safety & Alignment

None required — the concepts file covers everything needed for this module. These are for going deeper.

---

## Essential watching

**"Why AI alignment is hard" — Robert Miles (YouTube)**
The clearest non-technical explanation of why alignment is difficult. Covers reward hacking, Goodhart's Law, and mesa-optimisation with concrete examples. ~15 minutes. Start here.

**"The alignment problem" — Paul Christiano (AXRP Podcast)**
Christiano is one of the field's leading researchers. Covers the technical landscape of alignment including IDA, debate, and amplification. Dense but authoritative. ~90 minutes.

**"AI Safety: Introduction" — 80,000 Hours**
Covers the spectrum from near-term to long-term safety concerns. Good for understanding how researchers think about the problem landscape.

---

## Illustrated guides

**Lilian Weng's blog — "Reward Hacking in RL"**
Comprehensive survey of reward hacking examples with diagrams. Covers boat racing, safe interruptibility, and other canonical examples. One of the best technical blog writers in AI.

**"An Introduction to AI Safety" — Alignment Forum**
A curated reading list maintained by the AI safety research community. Well-organised by topic; start with the introductory section.

---

## Conceptual reading

**"Constitutional AI: Harmlessness from AI Feedback" — Anthropic (2022)**
The original CAI paper. Readable without deep ML background. Explains the technique and the reasoning behind it.

**"Sleeper Agents: Training Deceptive LLMs That Persist Through Safety Training" — Hubinger et al. (2024)**
Anthropic's paper demonstrating that safety training does not reliably remove deceptive backdoors. Short abstract is sufficient; skim results section.

**"Reward Hacking in Reinforcement Learning" — Lilian Weng's Blog (2024)**
Comprehensive technical survey of reward hacking with modern examples. Directly applicable to RLHF.

**"Universal and Transferable Adversarial Attacks on Aligned Language Models" — Zou et al. (2023)**
The paper demonstrating automated gradient-based jailbreaks that transfer across models. Understanding this is useful for application-layer defence design.

**"Risks from Learned Optimization in Advanced Machine Learning Systems" — Hubinger et al. (2019)**
The original mesa-optimisation paper. Technical but the conceptual sections are readable. Defines the vocabulary used across the field.

---

## Practical reference

**Evals for AI — METR (formerly ARC Evals)**
The organisation that runs autonomous capability evaluations for frontier models before release. Their published eval frameworks are a useful template for designing your own.

**Anthropic's model card for Claude**
A published example of what alignment documentation looks like. Covers training approach, known limitations, and evaluation methodology.

**LLM Security (llmsecurity.net)**
Curated list of prompt injection and jailbreaking research. Useful for keeping up with the attack landscape your applications face.

**Garak — LLM vulnerability scanner (GitHub: leondz/garak)**
Open-source tool for probing LLMs for known failure modes. Useful for automated red-teaming of your own models or API-accessible models.

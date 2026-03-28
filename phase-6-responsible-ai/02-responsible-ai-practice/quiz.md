# Quiz: Responsible AI Practice

Attempt these before checking answers. These questions test your ability to apply fairness, explainability, and governance concepts to real situations — the kinds of decisions you will face when shipping AI products.

---

## Questions

**1.** A team at a fintech startup trains a credit scoring model. The model achieves 89% accuracy overall. Post-launch, a data scientist notices that the model approves 73% of applicants with "traditional" English names and only 51% of applicants with South Asian names, even when controlling for all financial variables. What type of bias is this? What are two likely sources? What should the team do in the next 48 hours?

**2.** Two researchers evaluate the same recidivism prediction algorithm. Researcher A concludes it is biased against Black defendants; Researcher B concludes it is fair. Both present accurate statistics. Explain how this is possible. What does it tell you about the concept of "fairness" in AI systems?

**3.** You are a developer at a 4-person startup building a resume screening tool. Your CEO wants to ship in two weeks. You have no legal team, no AI ethics specialist, and limited compute for extensive testing. Describe three specific, practical actions you would take in the next two weeks to reduce the risk of deploying a biased system.

**4.** A colleague says: "We use SHAP to explain our model's decisions, so users can understand why they were rejected, and regulators can see we're being transparent." Explain what SHAP actually tells you, and what it does not. Under what conditions would SHAP explanations be insufficient for regulatory compliance?

**5.** Describe the feedback loop risk in a content recommendation system. Be specific: trace the mechanism step-by-step from initial disparity to amplified disparity. Then describe two technical interventions that could break the loop.

**6.** A user in Germany is rejected by your company's AI-powered job matching tool. They ask why. Under the EU AI Act, are you obligated to explain? What does your obligation depend on? What would a compliant response include?

**7.** Your company's AI product was built and tested entirely in English with US-based users. Six months after launch, you expand to India and discover that accuracy drops from 91% to 67% for users writing in Hinglish (a blend of Hindi and English). What type of bias is this? What should have been done before the expansion? What do you do now?

**8.** A product manager wants to add a "bias audit" checkbox to your release checklist. Describe what a meaningful AI audit actually involves — what you would measure, how you would run it, and what the audit output should include. Contrast this with a checkbox audit that provides false assurance.

**9.** You build a medical triage chatbot. The system recommends whether users should seek emergency care, urgent care, or schedule a routine appointment. A user later claims the system systematically advises non-urgent care to elderly users, causing delayed treatment. Walk through: (1) how you would investigate this claim, (2) what evidence would confirm or refute it, (3) what your response should be if confirmed.

**10.** A junior developer asks: "Doesn't using race as an input variable cause bias? We should just not include demographic features." Explain why this reasoning is incorrect (the "fairness through unawareness" fallacy), and describe what actually happens when you remove demographic features from a biased system.

**11.** Apply the NIST AI RMF's four functions (Govern, Map, Measure, Manage) to a hypothetical 10-person startup shipping an AI-powered hiring tool. For each function, describe one concrete action the team should take.

**12.** A startup fine-tunes an open-source model on their customer data and deploys it as a B2B product sold to European enterprise customers. Their largest customer is a German bank that uses it to assess loan applications. Under the EU AI Act, who is responsible for compliance — the startup (the AI provider) or the bank (the deployer)? What documentation should the startup have ready?

---

## Answers

**1.** This is **allocative bias** — the system denies an opportunity (loan approval) differentially across groups. Two likely sources:

*Historical data bias* — if the training data reflects historical lending patterns where minority-name applicants were rejected more often (due to discrimination in prior lending decisions), the model learns to replicate those patterns even on identical financial profiles.

*Proxy discrimination* — name correlates with ethnicity, which may correlate with zip code, school attended, or other features in the dataset. The model learns a proxy for ethnicity even if the protected attribute was removed.

Next 48 hours:
1. Immediately stop using the model for new decisions while investigating — the harm is ongoing with each additional decision
2. Audit the full decision log: compare approval rates controlling for financial variables across all demographic groups in the data
3. Notify legal counsel — disparate impact in lending is a Fair Housing Act / Equal Credit Opportunity Act issue in the US, regulated across jurisdictions

---

**2.** Both researchers are correct because they are measuring different fairness criteria that cannot be simultaneously satisfied when base rates differ between groups.

Researcher A measured *false positive rate parity*: among defendants who did NOT reoffend, Black defendants were labelled high-risk at a higher rate than White defendants.

Researcher B measured *predictive parity*: among defendants labelled high-risk, Black and White defendants reoffended at equal rates — meaning the label is equally predictive for both groups.

When base rates differ between groups (one group has a higher actual reoffend rate), you cannot equalise both false positive rates AND predictive value simultaneously. This is a mathematical impossibility proven by Chopin et al. (2016).

What this tells you: "fairness" is not a single metric. There are at least three mathematically distinct and often incompatible definitions. Choosing which to optimise is a values decision that should be made explicitly and documented — not left to the default behaviour of a model or a team that only looked at one metric.

---

**3.** Three practical actions for a two-week window:

1. **Disparate impact testing** — collect or construct a test set with name diversity (use a list of common names across ethnic groups). Run your model on identical CVs with only the name changed. If approval rates differ, you have a problem to fix before launch.

2. **Define and document out-of-scope use** — write one page: "This tool is designed to surface candidates for human review, not to make final hiring decisions. It should not be used as the sole basis for rejection." This limits both harm and liability, and is the first document a regulator will request.

3. **One adversarial test session** — spend two hours with your team deliberately trying to make the tool fail unfairly. Assign one person to play each persona: a highly qualified candidate with a non-English name, a candidate with an employment gap (statistically correlated with gender), a candidate with a disability mentioned in the CV. Document what you find.

---

**4.** SHAP tells you which features contributed most to a specific prediction, and in which direction, based on their Shapley values. It is a measure of *feature attribution* — not a measure of *correctness*, *fairness*, or *legality*.

What SHAP does NOT tell you:
- Whether the model's decision was correct
- Whether the features it relied on are legitimate bases for the decision
- Whether the model is discriminating against a protected group
- Whether the explanation is understandable to a lay user

SHAP explanations would be insufficient for regulatory compliance under the EU AI Act when: (1) the system is classified as high-risk and a meaningful explanation must be understandable to the affected person (a SHAP feature attribution chart is not always this); (2) the explanation does not address why certain inputs carry the weight they do, only that they do; (3) the feature weights reflect proxy discrimination (e.g., "zip code was the top feature" in a lending decision is an explanation but not a compliant one).

---

**5.** Content recommendation feedback loop — step by step:

Step 1: Model is trained on majority-user engagement data; slightly underweights minority-user preferences.
Step 2: System recommends slightly less relevant content to minority users.
Step 3: Minority users engage less (lower click rates, shorter sessions).
Step 4: Less engagement generates less training signal from minority users.
Step 5: Retraining uses data that underrepresents minority preferences even more.
Step 6: Model becomes more biased toward majority preferences in next version.

Two technical interventions:
1. **Targeted data collection** — actively seek engagement signal from underrepresented groups. Weight minority-user signals more heavily in training proportional to their underrepresentation. This breaks the under-signal loop.
2. **Exploration injection** — deliberately show a subset of users content outside their predicted preference distribution. This generates counterfactual data on what users engage with when not filtered by the current model's biases.

---

**6.** Yes, you likely have an obligation to explain, but the specifics depend on risk classification.

Under the EU AI Act: if your job matching tool is classified as a *high-risk AI system* (AI in employment contexts is explicitly listed in Annex III), the system must be designed with transparency for affected persons. Affected individuals have the right to request an explanation of the decision.

Under GDPR Article 22: if the decision is automated and produces a legal or similarly significant effect, the individual has the right to: (a) human review of the decision, (b) the ability to express their point of view, and (c) an explanation of the decision's logic.

A compliant response includes: the main factors that led to the outcome, how those factors were weighted, what the individual can do to contest the decision, and contact information for the responsible party. It does not require exposing proprietary model weights or training data.

---

**7.** This is **deployment bias** — the system was evaluated on one distribution (US English) and deployed on a different one (Hinglish). The bias was not in the model per se; it was in the assumption that evaluation performance generalises.

What should have been done before expansion:
- Collect a representative test set for the new market before launching there
- Evaluate on a sample of Hinglish text to detect the performance gap
- If the gap was unacceptable, either adapt the model or delay expansion

What to do now:
- Immediately flag the accuracy gap to the product team and stakeholders
- Limit high-stakes use of the system in this market until the gap is addressed
- Collect Hinglish training and evaluation data
- Fine-tune or adapt the model; re-evaluate; re-launch
- Do not treat 67% accuracy as acceptable for a system that affects user outcomes

---

**8.** A meaningful audit:

*What to measure:*
- Accuracy, precision, recall, and F1 broken down by demographic group (slice-based analysis)
- False positive and false negative rates by group (they have different harm profiles)
- Performance stability over time (does accuracy drift?)
- Adversarial robustness (does the system fail on deliberately edge-case inputs?)

*How to run it:*
- Assemble a test set with ground-truth labels and demographic metadata
- Define slices before running (to avoid p-hacking your slice choices based on results)
- Run evaluation on each slice
- Conduct adversarial probing sessions

*Audit output:*
- Performance metrics by slice with confidence intervals
- List of identified failure modes with severity ratings
- Root cause analysis for significant disparities
- Recommendations with an estimated effort to remediate
- Decision: ship as-is / ship with mitigations / do not ship until fixed

A checkbox audit ("we ran our standard test suite — pass") provides false assurance because it doesn't distinguish overall accuracy from per-group accuracy, doesn't include adversarial cases, and doesn't produce actionable output. It is evidence that you ran tests, not that you found your system's actual failure modes.

---

**9.** Investigating the claim:

Step 1: Extract all interactions where the system recommended "routine appointment" for users who later received emergency or urgent care treatment (if you have outcome data) or where elderly users received systematically different recommendations than younger users with equivalent symptom descriptions.

Step 2: Run a slice analysis: recommendation distribution stratified by age group, controlling for symptom severity (if labelled). Look for a systematic shift in recommendations for elderly users.

Step 3: Reconstruct with counterfactual inputs: submit identical symptom descriptions with different ages and compare outputs. Any systematic difference is attributable to age.

Evidence that confirms: recommendation rates differ significantly by age group on equivalent symptoms; counterfactual inputs produce different outputs based on age alone.

Evidence that refutes: recommendation rates reflect actual symptom severity differences; no age-based divergence in counterfactual testing.

If confirmed:
- Immediately take the system offline or disable the age-correlated feature pending investigation
- Notify affected users who received potentially incorrect recommendations (this is an incident response situation)
- Conduct a full root cause analysis (was age an explicit feature? Implicit through a proxy?)
- Legal review: medical device regulations vary by jurisdiction; this may trigger mandatory incident reporting

---

**10.** Fairness through unawareness is the belief that removing demographic features from model inputs prevents bias. It is incorrect for two reasons:

First, *proxy features*: other variables in your dataset may correlate strongly with demographic attributes. Zip code correlates with race. Years of employment gap correlates with gender. School name correlates with socioeconomic background. The model learns these proxies and effectively discriminates using them, even without the protected attribute as an explicit input.

Second, *the historical data still encodes the discrimination*: if your training labels reflect historical decisions that were biased (e.g., loan approvals that discriminated), the model learns those patterns regardless of what features you include. The discrimination is in the labels, not just the features.

Removing demographic features makes the bias *less visible* and *harder to audit*. You can no longer directly test whether the model treats groups differently, because you don't know what the model "thinks" each applicant's demographics are. This is worse, not better. The correct approach is to keep demographic features, test explicitly for disparate impact, and apply fairness constraints if needed.

---

**11.** NIST AI RMF applied to a hiring tool startup:

**Govern** — Assign one person as the "AI risk owner" (likely the technical lead or CTO). Document that role explicitly. Create a one-page policy: what this system is allowed to do, what is out of scope, and who makes the decision to take it offline if a problem is found.

**Map** — Before launch, spend two hours writing down what could go wrong: biased rejections by demographic group, inaccurate assessments of unconventional career paths, system misuse by clients to screen out protected classes. Rate severity and likelihood for each. This is your risk register.

**Measure** — Choose metrics that would indicate problems: per-demographic-group approval rates, user complaint rates, accuracy on held-out diverse test sets. Decide in advance what threshold triggers investigation ("if approval rate variance between demographic groups exceeds 5 percentage points, we convene a review").

**Manage** — Write down: "If we find a problem, here is what we do." Who gets notified? What is the rollback procedure? Who communicates to affected customers? Having this plan before you need it reduces both harm and chaos when something goes wrong.

---

**12.** Under the EU AI Act, responsibility is **shared**, but structured differently:

The **provider** (the startup) is responsible for: technical documentation, conformity assessment, registering in the EU AI database, affixing CE marking, providing instructions for use, and ensuring the model meets accuracy and robustness requirements.

The **deployer** (the bank) is responsible for: ensuring the system is used in accordance with the provider's instructions, implementing human oversight, conducting data governance for their specific use context, and monitoring performance in their deployment.

The startup should have ready:
- Technical documentation (model architecture, training data description, performance metrics, known limitations)
- Conformity assessment documentation (for Annex III high-risk AI — lending is explicitly listed)
- Instructions for use (including what the system should not be used for)
- Incident reporting procedures (who to contact and how)
- Evidence that the system meets accuracy, robustness, and non-discrimination requirements

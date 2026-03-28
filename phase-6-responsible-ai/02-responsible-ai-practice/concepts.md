# Concepts: Responsible AI Practice

> **Tag: core** — Covers the practical tools, frameworks, and processes for building AI systems that are fair, explainable, and accountable.

---

## Bias taxonomy

**What it is** — Bias in AI systems is not a single problem. It has distinct types with different causes, different harms, and different remedies. Conflating them leads to solutions that address one type while leaving others untouched.

**The two primary types:**

*Allocative bias* — The system denies resources, opportunities, or services to some groups more than others. A loan approval model that rejects qualified applicants from one demographic group. A hiring tool that screens out qualified candidates from another. The harm is material and direct.

*Representational bias* — The system encodes or amplifies stereotypical associations. An image-generation model that defaults to male images for "CEO" and female images for "nurse." A text model that associates certain names with crime. The harm is more diffuse but erodes trust and shapes expectations at scale.

**Why the distinction matters** — Allocative bias is often legally actionable (see: disparate impact). Representational bias may not trigger legal liability but causes harm at scale through normalisation of stereotypes. The fixes differ: allocative bias often requires post-hoc correction or constraints; representational bias often requires data curation and training objective changes.

**Sources of bias in AI systems:**
```
Training data bias
  — Historical data encodes historical discrimination
  — Underrepresentation of minority groups
  — Label noise that correlates with protected attributes

Model bias
  — Spurious correlations learned during training
  — Proxy features that correlate with protected attributes
    (e.g., zip code as proxy for race)

Deployment bias
  — System is deployed in a context different from training
  — Feedback loops that amplify initial disparities
  — Human reviewers who apply the system differently by group
```

---

## COMPAS — the canonical case

**What it is** — COMPAS (Correctional Offender Management Profiling for Alternative Sanctions) is a risk assessment algorithm used by US courts to predict whether a defendant will reoffend. ProPublica's 2016 analysis found it predicted higher recidivism risk for Black defendants and lower risk for White defendants at equal base rates. Northpointe (the developer) disputed the findings with different but equally correct statistics.

**Why it matters** — The COMPAS case is not just about one algorithm. It is the clearest demonstration that "fairness" is not a single, mathematically definable property — and that two teams can both be telling the truth while reaching opposite conclusions.

**The mathematical impossibility:**

ProPublica's claim (correct): Among defendants who did *not* reoffend, Black defendants were labelled high-risk at twice the rate of White defendants (false positive rate disparity).

Northpointe's claim (also correct): Among defendants labelled high-risk, Black and White defendants reoffended at equal rates (equal predictive value across groups).

Both statistics are accurate. They are also mathematically incompatible when base rates differ between groups — and Black defendants had higher base rates of prior criminal justice involvement (itself a product of systemic factors). You cannot simultaneously equalise false positive rates AND predictive value across groups with different base rates. Chopin et al. (2016) proved this formally.

**The developer lesson** — Before you define "fairness" for your system, you must decide which metric you are optimising and accept that this is a values decision, not a technical one. Document it. Different stakeholders will have different and legitimate views.

---

## Disparate impact and legal liability

**What it is** — Disparate impact is the legal doctrine that a policy, practice, or algorithm can be unlawfully discriminatory even without discriminatory intent. If an employment practice causes a statistically significant disadvantage to members of a protected class, the employer must demonstrate business necessity — regardless of intent.

**Why it matters** — You do not have to intend to discriminate to be liable. An algorithm that uses zip code to screen loan applications and systematically disadvantages minority applicants can violate the Fair Housing Act, even if race was never a variable.

**The 80% rule** — US employment law uses a rough heuristic: if the selection rate for one group is less than 80% of the highest-performing group, there may be adverse impact. This is not a hard legal threshold but a starting point for investigation.

**Protected attributes vary by jurisdiction:**
```
US: race, colour, religion, sex, national origin, age (40+), disability
EU: race, gender, age, disability, religion, sexual orientation, nationality
UK: same as EU plus pregnancy, marriage/civil partnership
```

**Proxy discrimination** — The most common form: using a variable that is not a protected attribute but strongly correlates with one. Zip code and race. Years of employment gap and gender (maternity leave). Criminal record and race (due to biased enforcement). Technically neutral; practically discriminatory.

**Developer implication** — If you are building systems that make decisions about people (hiring, lending, insurance, healthcare, criminal justice), legal counsel is not optional. Neither is bias testing before deployment.

---

## Explainability (XAI): LIME and SHAP

**What it is** — Explainability (or interpretability) refers to the ability to describe why a model produced a specific output. LIME and SHAP are the two most widely used post-hoc explainability techniques.

**Why it matters** — The EU AI Act requires "meaningful explanation" for consequential automated decisions in high-risk categories. GDPR (Article 22) includes a right to explanation for automated decisions. Beyond legal compliance, explanations are essential for debugging bias and for building user trust.

**LIME (Local Interpretable Model-agnostic Explanations):**
```
Goal: explain why this specific prediction was made
How:
  1. Take the input you want to explain
  2. Generate many perturbed versions of it (slightly modified)
  3. Run all versions through the model
  4. Fit a simple interpretable model (e.g., linear regression) to
     the model's outputs for these perturbed inputs
  5. The simple model's coefficients are the explanation

Output: "For this loan application, the top factors increasing
         rejection probability were: debt-to-income ratio (+0.4),
         employment duration (-0.3), credit score (+0.2)"
```

**SHAP (SHapley Additive exPlanations):**
```
Goal: consistent, theoretically grounded feature attribution
How: based on Shapley values from cooperative game theory
     — each feature gets the credit for its average marginal
       contribution to the prediction across all possible
       feature orderings

Output: "Across all applications, 'years_employed' was the
         most impactful feature. For this specific application,
         'debt_ratio' contributed +0.3 and 'credit_score' -0.1
         to the final score."
```

**LIME vs SHAP:**
| Property | LIME | SHAP |
|----------|------|------|
| Speed | Fast | Slower (tree models: fast; deep learning: slow) |
| Consistency | Can vary across runs | Consistent (unique Shapley values) |
| Global insights | No | Yes (aggregate feature importance) |
| Model-agnostic | Yes | Yes |

**Limitation** — Both LIME and SHAP explain the model as a function of its inputs. They do not explain whether the model is correct, fair, or safe. A model that is biased for the wrong reasons can have perfectly plausible-sounding SHAP explanations.

---

## Model cards and datasheets

**What it is** — Model cards (Mitchell et al., 2018) and datasheets for datasets (Gebru et al., 2018) are standardised documentation templates for AI models and the data used to train them. They answer: what is this model for, what are its known limitations, and what should it not be used for.

**Why it matters** — When a system causes harm, the first questions are: did you know this was a risk? Is there documentation? Model cards and datasheets are not just good practice — they are evidence of due diligence, and their absence is evidence of negligence.

**Model card structure:**
```
Model details
  — Who created it, when, which version
  — Model type and architecture

Intended use
  — Primary intended uses
  — Out-of-scope uses

Factors
  — Groups or conditions under which performance may vary
  — Evaluation factors considered

Metrics
  — Performance metrics used
  — Decision thresholds and their justification

Evaluation data
  — What data was used to evaluate the model
  — Preprocessing

Training data
  — What data was used to train (often limited for privacy)

Ethical considerations
  — Data rights, potential harms, sensitive use cases

Caveats and recommendations
  — Known limitations
  — Recommendations for deployment contexts
```

**Datasheets for datasets cover:** motivation, composition, collection process, preprocessing, uses (recommended and not), distribution, and maintenance.

**Practical reality** — Most shipped AI products have no model card. This is increasingly a liability. Start with a minimal internal version: intended use, known limitations, evaluation methodology.

---

## AI auditing

**What it is** — AI auditing is the systematic process of evaluating an AI system for bias, accuracy, safety, and compliance before and during deployment. It is structured testing, not just running benchmarks.

**Why it matters** — Ad-hoc testing finds the problems you happened to think of. Auditing finds the problems your system actually has, including ones you didn't anticipate.

**Core technique: slice-based analysis**
```
Standard evaluation: "Our model is 92% accurate overall."

Slice-based evaluation:
  — Accuracy by demographic group
  — Accuracy by input distribution (formal vs informal language)
  — Accuracy by time period (does performance degrade over time?)
  — Accuracy by edge cases (short inputs, non-English text, etc.)

Result: "Our model is 92% accurate overall, but 74% accurate
         for users whose first language is not English, and
         61% accurate for inputs under 10 words."
```

**Audit process:**
```
Step 1: Define the audit scope
  — What harms are you testing for?
  — What protected groups are relevant?
  — What's the consequence of a false positive vs false negative?

Step 2: Assemble ground truth
  — Labelled test data with demographic metadata
  — Adversarial test cases constructed deliberately

Step 3: Run slice-based evaluation
  — Compute metrics separately for each slice
  — Look for significant deviations from overall performance

Step 4: Root cause analysis
  — Is the disparity from training data? Model architecture?
    Deployment context?
  — What would it take to fix vs mitigate?

Step 5: Document and decide
  — Document findings, including acceptable/unacceptable ones
  — Make deployment decisions with full knowledge of the gaps
```

**Third-party audits** — Increasingly required by procurement processes (especially government contracts) and proposed regulation. External auditors evaluate your system against your documentation; they are only as useful as your documentation is honest.

---

## Feedback loops

**What it is** — A feedback loop in AI occurs when the model's outputs influence the data used to evaluate or retrain the model. Initial disparities become encoded as ground truth, and the model learns to perpetuate them.

**Why it matters** — Feedback loops mean that bias in deployed systems is self-reinforcing. A model that is 5% less accurate for one group will generate worse outputs for that group, receive lower ratings for that group, retrain on worse signal for that group, and become 8% less accurate. This happens without anyone designing it.

**Classic example — predictive policing:**
```
Step 1: Deploy model that predicts crime hotspots
Step 2: Police patrol predicted hotspots more
Step 3: More arrests in predicted hotspots (because more police)
Step 4: More arrests = more crime in training data
Step 5: Retrain model on new data
Step 6: Model predicts even higher crime in same areas
Repeat.
```

**The developer version:**
```
Step 1: Recommendation system is slightly biased toward content
        consumed by majority users
Step 2: Minority users see less relevant recommendations
Step 3: Minority users engage less, produce less training signal
Step 4: Model has less data on minority user preferences
Step 5: Model becomes more biased toward majority users
Repeat.
```

**How to break the loop:**
- Collect targeted feedback data for underperforming groups
- Use offline evaluation (hold-out sets) rather than only online signals
- Audit regularly for drift in per-group performance
- Actively intervene when disparities widen; don't wait for the feedback loop to stabilise

---

## EU AI Act

**What it is** — The EU AI Act (entered into force August 2024, phased implementation through 2027) is the world's first comprehensive AI regulation. It takes a risk-based approach: the higher the potential harm, the more stringent the requirements.

**Why it matters** — If your product is used by EU citizens or deployed by EU-based organisations, the Act applies. Violations carry fines of up to €35M or 7% of global annual turnover.

**Risk tiers:**
```
Unacceptable risk — PROHIBITED
  — Social scoring by governments
  — Real-time biometric surveillance in public spaces (with exceptions)
  — Subliminal manipulation
  — Exploiting vulnerabilities of specific groups

High risk — STRICT REQUIREMENTS
  Examples: hiring tools, credit scoring, medical devices,
            biometric categorisation, critical infrastructure,
            law enforcement tools, education assessment

  Requirements:
  — Risk management system
  — Data governance documentation
  — Technical documentation
  — Transparency to users
  — Human oversight mechanisms
  — Accuracy, robustness, cybersecurity standards
  — Conformity assessment before deployment
  — Registration in EU database

Limited risk — TRANSPARENCY OBLIGATIONS
  — Chatbots must disclose they are AI
  — Deepfake content must be labelled

Minimal risk — NO REQUIREMENTS
  — Spam filters, AI in video games, most B2B tools
```

**GPAI (General Purpose AI) provisions** — Frontier models (>10^25 FLOPs training compute) face additional requirements: capability evaluations, incident reporting, adversarial testing.

**What developers actually do:** Determine your risk category. For high-risk systems, implement the documentation and oversight requirements before deployment. For limited-risk, add disclosure. For minimal-risk, you're currently exempt but document anyway.

---

## NIST AI Risk Management Framework

**What it is** — The NIST AI RMF (2023) is a voluntary US framework for managing risks in AI systems. It is not law but is increasingly referenced in procurement, contracts, and regulation proposals. It provides structure for thinking about AI risk regardless of jurisdiction.

**Why it matters** — The NIST framework is practical, developer-oriented, and less prescriptive than the EU AI Act. It gives you a vocabulary and process for managing AI risk that your stakeholders and customers will recognise.

**The four core functions:**
```
GOVERN
  — Establish policies, culture, accountability
  — Who is responsible for what AI risk decisions?
  — How are decisions documented and reviewed?

MAP
  — Identify and categorise AI risks in context
  — Who is affected? What are the potential harms?
  — What is the risk tolerance for this use case?

MEASURE
  — Quantify and evaluate identified risks
  — What metrics indicate risk? How are they monitored?
  — How often are evaluations conducted?

MANAGE
  — Prioritise and address identified risks
  — What is the risk response plan?
  — How are residual risks tracked over time?
```

**For a small team** — GOVERN means documenting who owns AI decisions. MAP means writing down what could go wrong before launch. MEASURE means choosing metrics that catch failures before users do. MANAGE means having a plan for when something goes wrong.

---

## Red-teaming for bias

**What it is** — Red-teaming specifically for bias means adversarially probing your model to find differential performance, harmful outputs, or stereotypical behaviour across demographic groups. It is distinct from safety red-teaming (which looks for harmful content generally) and from standard bias auditing (which looks at performance metrics on existing data).

**Why it matters** — Bias audits measure what you can measure on your existing test set. Bias red-teaming finds the failures you didn't think to look for.

**Techniques:**
```
Demographic substitution testing
  — Swap names, pronouns, locations across equivalent scenarios
  — "John Smith applied for the loan" vs "Jamal Williams applied"
  — Does the output change in ways unrelated to the name?

Implicit association probing
  — Ask the model to generate descriptions of professionals by field
  — Ask for images of "a doctor", "a criminal", "a terrorist"
  — Audit for demographic skew in completions

Counterfactual fairness testing
  — Change only protected attributes in inputs
  — All other factors identical
  — Any change in output is attributable to the protected attribute

Dialect and language variation testing
  — Test with inputs in AAVE, non-standard spellings, non-English
  — Models trained primarily on standard English often perform worse
  — This is allocative bias if the system makes consequential decisions
```

---

## Responsible AI in small teams

**What it is** — A practical set of actions for teams without a Chief Ethics Officer, a legal team, or a dedicated safety researcher. The question is not "how do we do enterprise AI governance?" but "what can we do this quarter that will actually reduce harm?"

**Why it matters** — Most AI harm does not come from frontier labs or large enterprises — it comes from the long tail of small products that don't have the resources or expertise to implement full governance. Small teams need a practical minimum.

**The minimum viable responsible AI checklist:**

*Before launch:*
- Write down what the product is for and explicitly what it should NOT be used for
- Run at least one adversarial test session (even just your team, trying to break it)
- Evaluate on at least one non-majority demographic (test your English-only eval on non-native-speaker inputs)
- Check whether your use case falls under EU AI Act high-risk categories

*After launch:*
- Log all model inputs and outputs (with appropriate privacy controls)
- Review a random sample weekly for unexpected failures
- Set up a channel for users to report harms (email is fine; nothing is not)
- Plan: "If we discover the system is causing harm to a group, what do we do in the next 24 hours?"

*Documentation minimum:*
- Intended use + out-of-scope use: one page
- Known limitations: three bullet points
- Evaluation methodology: what did you test and how

This is not sufficient for high-risk systems under the EU AI Act. It is the minimum floor for any AI product.

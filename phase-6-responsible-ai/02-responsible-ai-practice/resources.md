# Resources: Responsible AI Practice

None required — the concepts file covers everything needed for this module. These are for going deeper.

---

## Essential watching

**"How algorithmic bias occurs and what to do about it" — MIT Technology Review (video)**
A clear, non-technical overview of where bias enters AI systems and the practical challenges of detecting and fixing it. Good starting point before going into the technical literature.

**"21 Fairness Definitions and Their Politics" — Arvind Narayanan (FAccT 2018)**
The definitive talk on mathematical fairness definitions. Narayanan explains why the various definitions are incompatible, and why choosing among them is a values question. This is what the COMPAS debate is actually about. 60 minutes; worth every one.

**"The Alignment Problem" — Brian Christian (Talks at Google)**
Book talk covering bias, reward hacking, and fairness in concrete terms. Accessible and developer-friendly.

---

## Illustrated guides

**"A visual introduction to machine learning fairness" — Google PAIR**
The People + AI Research team's illustrated guide to fairness concepts. Interactive examples. Very well done.

**Model Cards Toolkit — Google**
Practical templates and examples for writing model cards for your own models. Includes an interactive Colab notebook for generating model cards from evaluation results.

---

## Conceptual reading

**"Machine Bias" — ProPublica (2016)**
The original COMPAS investigation. Still the clearest journalism on algorithmic fairness. Read this before the academic papers that responded to it.

**"Model Cards for Model Reporting" — Mitchell et al. (2018)**
The paper that introduced model cards. Short, readable, directly applicable to your own documentation practice.

**"Datasheets for Datasets" — Gebru et al. (2018)**
The analogous paper for dataset documentation. Read alongside the model cards paper.

**"Fairness and Machine Learning" — Barocas, Hardt, Narayanan (free online textbook)**
The most thorough treatment of algorithmic fairness available. Chapter 1 (Introduction) and Chapter 3 (Classification) are the most directly useful. Free at fairmlbook.org.

**EU AI Act — full text (EUR-Lex)**
The actual regulation. Annex III (high-risk systems) and Title III (requirements for high-risk AI) are the most developer-relevant sections. Not light reading, but important if you're building in regulated domains.

---

## Practical reference

**NIST AI Risk Management Framework (nist.gov/artificial-intelligence)**
The full framework document plus a Playbook with specific practices for each function. Free to download.

**Fairlearn (fairlearn.org)**
Microsoft's open-source toolkit for assessing and improving fairness in ML models. Includes constraint-based fairness algorithms and visualisation tools. Works with scikit-learn.

**AI Fairness 360 — IBM (aif360.res.ibm.com)**
IBM's comprehensive fairness toolkit. 70+ fairness metrics and 11 bias mitigation algorithms. Python and R. Well-documented with worked examples.

**"Responsible AI practices" — Google AI**
Google's published practices covering fairness, interpretability, privacy, and safety. Useful for seeing how a large organisation structures these concerns operationally.

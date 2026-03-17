# Resources: LLM Fundamentals

These are curated external resources if you want to go deeper on any concept from this module. None are required — the concepts file covers everything you need to continue the course.

---

## Essential watching

**Andrej Karpathy — Intro to Large Language Models**
https://www.youtube.com/watch?v=zjkBMFhNj_g
A 1-hour talk by one of the field's clearest explainers. Covers what LLMs are, how they work, security considerations, and where the field is heading. If you watch one video from this entire resource list, make it this one.

**Andrej Karpathy — Let's build GPT from scratch**
https://www.youtube.com/watch?v=kCc8FmEb1nY
A 2-hour coding walkthrough building a character-level GPT from scratch in Python. You don't need to follow along with code — watching it builds deep intuition for how generation actually works. One of the best technical educational videos on the internet.

**3Blue1Brown — But what is a GPT? Visual intro to Transformers**
https://www.youtube.com/watch?v=wjZofJX0v4M
Visual, animated explanation of how GPT-style models work — attention, token prediction, and generation. No prior ML knowledge assumed.

---

## Illustrated guides

**The Illustrated GPT-2 (Jay Alammar)**
https://jalammar.github.io/illustrated-gpt2/
The most widely read visual guide to GPT-style models. Shows step-by-step how a decoder-only Transformer generates text, with diagrams for every concept. Essential reading if you want to see how everything in Phase 3 translates to a real language model.

**The Illustrated Transformer (Jay Alammar)**
https://jalammar.github.io/illustrated-transformer/
If you want a refresher on the Transformer architecture before diving into LLMs, this is the go-to reference. Covers encoder-decoder, attention, and positional encoding with clear diagrams.

**Visualising Attention in Transformers (BertViz)**
https://github.com/jessevig/bertviz
An interactive tool that lets you visualise what tokens attend to what in real models (BERT, GPT-2). Makes attention concrete rather than abstract.

---

## Conceptual reading

**Simon Willison — What I've learned about LLMs**
https://simonwillison.net/2023/Aug/3/weird-world-of-llms/
A practitioner's honest account of how LLMs actually behave in practice — including the weird, counterintuitive, and frustrating parts. Developer perspective throughout.

**Sebastian Raschka — Understanding Large Language Models**
https://magazine.sebastianraschka.com/p/understanding-large-language-models
A curated reading list and explanation covering LLM fundamentals, training, and alignment. Well-organised and accurate. Good companion to this module.

**Lilian Weng — Large Language Model (blog post)**
https://lilianweng.github.io/posts/2023-01-27-the-transformer-family-v2/
Weng (formerly at OpenAI) writes some of the best technical blog posts in the field. This covers transformer variants and attention mechanisms. More technical than most resources here — use it for depth, not as an intro.

---

## Interactive tools

**Tiktokenizer**
https://tiktokenizer.vercel.app/
Enter any text and see exactly how it is tokenised by OpenAI's tokeniser (cl100k_base, the tokeniser used by GPT-3.5, GPT-4, and embeddings). Essential for building intuition about tokens vs words, and for understanding why non-English text or code may tokenise differently than you expect.

**OpenAI Tokenizer**
https://platform.openai.com/tokenizer
OpenAI's official tokeniser tool. Same idea — see token counts and boundaries for text you enter.

**LLM Visualization (bbycroft)**
https://bbycroft.net/llm
An interactive 3D visualisation of a small GPT model running a forward pass in real time. Every matrix multiplication, attention head, and layer is shown. Genuinely useful for making the abstract concrete.

---

## Key papers (for reference — not required reading)

**"Language Models are Few-Shot Learners" — Brown et al., 2020 (GPT-3)**
https://arxiv.org/abs/2005.14165
The paper that introduced GPT-3 and demonstrated in-context learning at scale. Established the "scale unlocks capabilities" paradigm. Dense but historically important.

**"Training language models to follow instructions with human feedback" — Ouyang et al., 2022 (InstructGPT)**
https://arxiv.org/abs/2203.02155
The paper behind InstructGPT — the technique that turned a raw pre-trained GPT into a useful assistant via RLHF. Foundational for understanding how alignment works in practice.

**"Constitutional AI: Harmlessness from AI Feedback" — Bai et al., 2022 (Anthropic)**
https://arxiv.org/abs/2212.08073
Anthropic's paper on RLAIF — training models to be helpful and harmless using AI feedback rather than only human feedback. Introduces the "Constitutional AI" approach.

**"A Survey of Large Language Models" — Zhao et al., 2023**
https://arxiv.org/abs/2303.18223
A comprehensive academic survey covering architecture, training, alignment, and applications. Over 100 pages — use it as a reference, not a read-through.

---

## Courses and structured learning

**Hugging Face NLP Course (free)**
https://huggingface.co/learn/nlp-course
Free, code-first course covering tokenisation, Transformers, and fine-tuning with the Hugging Face ecosystem. Practical complement to the conceptual content in this module.

**fast.ai — Practical Deep Learning for Coders (free)**
https://course.fast.ai/
If you want to understand the full picture from data to deployment, fast.ai is one of the best free courses available. More in-depth than this course — treat it as a "go deeper" path after completing Phase 4.

**DeepLearning.AI Short Courses**
https://www.deeplearning.ai/short-courses/
A collection of short (1–2 hour) courses on specific topics: prompt engineering, fine-tuning, RAG, agents. Many are free. Good for targeted depth on specific Phase 4 and Phase 5 topics.

---

## Stay current

The LLM landscape moves fast. These are the best sources for staying up to date without getting overwhelmed:

**Simon Willison's Weblog**
https://simonwillison.net/
Prolific, opinionated, practitioner-focused. Willison covers every significant LLM release and AI development in plain language. One of the best signal-to-noise sources in the field.

**Ahead of AI (Sebastian Raschka)**
https://magazine.sebastianraschka.com/
Well-researched newsletter covering new papers and developments in LLMs and ML. Suitable for developers who want to track research without reading raw papers.

**The Batch (Andrew Ng / DeepLearning.AI)**
https://www.deeplearning.ai/the-batch/
Weekly newsletter summarising the most important AI news and papers. Reliable, accessible, and practitioner-oriented.

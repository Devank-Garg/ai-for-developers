# Quiz: Terminology

Attempt these questions before checking the answers. The goal isn't to pass a test — it's to find the gaps before they become problems later.

---

## Questions

**1.** Which of the following is the most accurate relationship between AI, machine learning, and deep learning?

a) They are three separate fields that overlap occasionally  
b) Machine learning is a subset of deep learning, which is a subset of AI  
c) Deep learning is a subset of machine learning, which is a subset of AI  
d) They are different names for the same thing

---

**2.** A developer writes a program with 500 hand-coded rules that filters spam emails based on keywords. Is this machine learning?

---

**3.** What does "training" a model actually mean at a technical level?

---

**4.** You deploy a language model to production. Users are now sending it thousands of requests per hour. What is this process called — training or inference? What's the key difference?

---

**5.** A model performs extremely well on its training data but poorly on new, unseen data. What is this problem called, and what does it indicate?

---

**6.** Roughly how many words is 1,000 tokens equivalent to in English?

a) 250  
b) 500  
c) 750  
d) 1,000

---

**7.** Why does context window size matter when building applications with LLMs?

---

**8.** A language model confidently tells a user that a specific scientific paper was published in Nature in 2019, cites the authors, and summarises its findings. You search for the paper and it doesn't exist. What is this called, and why does it happen?

---

**9.** What is the difference between pre-training and fine-tuning?

---

**10.** You have a dataset of 10,000 labelled images. You use 7,000 to train your model, 1,500 to monitor training progress, and hold 1,500 back entirely until your final evaluation. What are the standard names for each of these three splits?

---

## Answers

**1.** c) Deep learning is a subset of machine learning, which is a subset of AI.

AI is the broadest category (any technique enabling machine intelligence). ML is a subset where systems learn from data. Deep learning is a subset of ML using multi-layer neural networks.

---

**2.** No. This is rule-based AI, but it is not machine learning. The rules were written explicitly by a programmer — the system never learned anything from data. Machine learning requires that the system's behaviour emerges from training on examples, not from hand-coded rules.

---

**3.** Training is an iterative numerical optimisation process. The model is shown inputs, produces outputs, those outputs are compared to correct answers to calculate a loss (error), and the parameters (weights) are adjusted slightly to reduce that loss. This repeats millions or billions of times until the model's outputs are consistently good.

---

**4.** This is inference. The distinction: during training, the model's parameters are being updated. During inference, the parameters are frozen and the model is simply running inputs through its function to produce outputs. Training happens once (or periodically). Inference happens constantly — every user request is an inference call.

---

**5.** This is called overfitting. It indicates the model memorised specific patterns in the training data — including noise and quirks — rather than learning general rules that would apply to new data. A model that overfits has high training performance but poor generalisation.

---

**6.** c) 750 words. A rough rule of thumb is that 1 token ≈ 0.75 words in English, so 1,000 tokens ≈ 750 words. This varies by language and content type — code and non-English text often tokenise differently.

---

**7.** The context window is a hard limit on how much text the model can consider at once. In a conversation, all previous messages, system prompt, and current input must fit inside it — anything outside the window is invisible to the model. This constrains how long conversations can be, how much context you can provide, and how large documents can be processed. It also affects cost, since API pricing is typically token-based.

---

**8.** This is called hallucination. It happens because LLMs are trained to generate plausible-sounding text — they produce tokens that fit the statistical patterns of their training data. The model has no reliable mechanism for distinguishing things it "knows" from things it's confabulating. When patterns in its training data lead it toward a confident-sounding but false statement, it produces that statement with the same fluency as a correct one.

---

**9.** Pre-training is training a model from scratch on a large, general dataset (for language models, enormous amounts of text). It's expensive and typically done by large labs. Fine-tuning takes an already pre-trained model and trains it further on a smaller, more specific dataset to adapt its behaviour for a particular task or domain. Fine-tuning is much cheaper and is what most developers and companies do to customise a model.

---

**10.** The three standard splits are:
- **Training set** (the 7,000 images) — what the model learns from directly
- **Validation set** (the 1,500 used during training) — used to monitor progress and tune hyperparameters, but not trained on
- **Test set** (the held-out 1,500) — only used for final evaluation; gives a true measure of performance on genuinely unseen data

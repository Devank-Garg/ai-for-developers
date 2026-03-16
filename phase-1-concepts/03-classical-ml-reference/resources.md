# Resources: Classical ML Reference

These are curated external resources if you want to go deeper on any concept from this module. None are required — the concepts file covers everything you need to continue the course.

---

## Foundational courses

**fast.ai — Practical Machine Learning for Coders**
https://course.fast.ai/
A free, code-first course that covers classical ML and deep learning from the top down — starting with practical usage and working toward theory. One of the best ways to build real intuition. Highly recommended if you want to go from concepts to hands-on.

**Kaggle Learn — Machine Learning**
https://www.kaggle.com/learn/intro-to-machine-learning
Short, practical notebooks covering decision trees, random forests, and validation. Runs entirely in the browser with real datasets. Good for getting hands on quickly without setup.

**Google Machine Learning Crash Course**
https://developers.google.com/machine-learning/crash-course
A free structured course from Google covering regression, classification, the bias-variance tradeoff, and regularisation. Well-paced and includes interactive visualisations.

---

## Specific algorithms

**Decision Trees and Ensembles**
*"A Visual Introduction to Machine Learning"* (r2d3.us) — Part 1 covers decision trees with outstanding interactive visualisations. Part 2 covers bias and variance. These are among the clearest visual explanations of these concepts online.

**XGBoost / Gradient Boosting**
XGBoost documentation (https://xgboost.readthedocs.io/) is unusually good — the conceptual overview section explains gradient boosting intuitively before getting into implementation. LightGBM (https://lightgbm.readthedocs.io/) is similar quality.

**Support Vector Machines**
*"Support Vector Machines: A Plain English Explanation"* — search for this phrase; multiple good versions exist from Towards Data Science and similar. The kernel trick visualisation is particularly useful — look for the 3D projection demonstrations.

**k-Nearest Neighbours**
A good k-NN explainer is less important than understanding the connection to modern vector search: the Pinecone blog (https://www.pinecone.io/learn/) has excellent articles on approximate nearest neighbour search, which is k-NN at scale and foundational to RAG (Phase 5).

---

## Evaluation metrics

**"Beyond Accuracy: Precision and Recall"**
https://towardsdatascience.com/beyond-accuracy-precision-and-recall-3da06bea9f6c
A clear visual walkthrough of precision, recall, and the confusion matrix. Essential reading if you haven't worked with imbalanced classification problems before.

**The scikit-learn metrics documentation**
https://scikit-learn.org/stable/modules/model_evaluation.html
Comprehensive reference for every evaluation metric mentioned in this module (and many more). The examples are runnable Python.

---

## Interactive tools

**Playground.tensorflow.org**
https://playground.tensorflow.org
An in-browser interactive neural network that also clearly demonstrates the bias-variance tradeoff, overfitting, and the effect of regularisation. Despite being a neural network tool, the intuition it builds directly applies to classical ML. The "features" and "regularisation" controls are particularly instructive.

**scikit-learn's algorithm cheat sheet**
https://scikit-learn.org/stable/tutorial/machine_learning_map/
A decision tree for choosing ML algorithms based on your problem type, data size, and goals. A practical reference to bookmark.

---

## Books

**"Hands-On Machine Learning with Scikit-Learn, Keras, and TensorFlow" (Aurélien Géron)**
https://www.oreilly.com/library/view/hands-on-machine-learning/9781492032632/
The most widely recommended practical ML book. Part 1 (chapters 1–8) covers classical ML thoroughly with code. If you're going to buy one book for this course, this is it.

**"The Elements of Statistical Learning" (Hastie, Tibshirani, Friedman)**
https://hastie.su.domains/ElemStatLearn/ *(free PDF from the authors)*
The rigorous mathematical treatment of classical ML algorithms. Dense and graduate-level — not required for this course, but worth knowing exists if you want to go deep on any algorithm's foundations.

---

## If you want to go much deeper

**"Winning with Machine Learning: Lessons from Kaggle Competitions"**
Search for blog posts and writeups from Kaggle competition winners. The top solutions are almost always gradient boosting on tabular data. These writeups are practical, specific, and full of techniques that don't appear in textbooks. Start with the Kaggle blog (https://medium.com/kaggle-blog).

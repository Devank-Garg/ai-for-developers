# Concepts: Classical ML Reference

---

## What is classical ML and why does it still matter?

"Classical ML" — or traditional machine learning — refers to algorithms that were developed and refined largely before the deep learning era: linear regression, logistic regression, decision trees, support vector machines, k-means clustering, and their variants.

These are not obsolete. For a large class of real-world problems — particularly those involving structured, tabular data — classical ML algorithms outperform or match deep learning while being faster to train, cheaper to run, easier to interpret, and requiring far less data. In industry, the majority of production ML deployments still use classical methods.

This module is a reference, not a deep tutorial. The goal is to give you enough understanding to read the field accurately, know what tool fits what problem, and understand the foundations that deep learning builds on. Each algorithm family is covered conceptually; the implementation details are available in the resources linked at the end.

---

## The three paradigms: supervised, unsupervised, and reinforcement learning

All of ML can be loosely organised around how the learning signal is provided.

**Supervised learning** — the model learns from labelled examples. You provide inputs paired with correct outputs, and the model learns the mapping. This is the most common paradigm in practice and covers most of what developers work with: classification, regression, ranking.

**Unsupervised learning** — the model learns from unlabelled data, finding structure without being told what to look for. Clustering, dimensionality reduction, and anomaly detection are unsupervised tasks.

**Reinforcement learning (RL)** — the model (called an **agent**) learns by taking actions in an environment and receiving rewards or penalties. Rather than learning from a fixed dataset, it learns through experience. RL is used in game-playing agents, robotics, and — notably — the training of LLMs (through a technique called RLHF, or Reinforcement Learning from Human Feedback, covered in Phase 4).

Most of classical ML is supervised or unsupervised. A fourth category — **self-supervised learning** — has become important for deep learning (LLMs are trained self-supervisedly), but it's less prominent in classical ML.

---

## Supervised learning: regression vs classification

Within supervised learning, the nature of the label determines whether you're solving a **regression** or **classification** problem.

**Regression** — the label is a continuous number. Predicting house prices, forecasting sales, estimating delivery time. The model outputs a number.

**Classification** — the label is a category. Spam or not spam, which digit (0–9), which disease (out of many). The model outputs a class or a probability distribution over classes.

**Binary classification** has two possible outputs (yes/no, fraud/legitimate). **Multi-class classification** has more than two. **Multi-label classification** allows each sample to belong to multiple classes simultaneously (a photo that contains both a cat and a dog).

This distinction affects which algorithms, loss functions, and evaluation metrics are appropriate.

---

## Linear regression

**Linear regression** is the simplest model in ML. It models the relationship between features and a continuous label as a straight line (or hyperplane in higher dimensions).

The model is:
```
ŷ = w₁x₁ + w₂x₂ + ... + wₙxₙ + b
```

Where `x₁...xₙ` are features, `w₁...wₙ` are weights (parameters), and `b` is a bias term (intercept). Training finds the weights that minimise the difference between predicted and actual values — typically using mean squared error (MSE) as the loss.

Linear regression makes strong assumptions: that the relationship between features and label is linear, and that errors are independent and normally distributed. These assumptions often don't hold in practice, which is why more flexible models exist.

Despite its simplicity, linear regression is a useful baseline, interpretable (you can see exactly which features contribute how much), and fast. It also forms the mathematical foundation for understanding more complex models.

---

## Logistic regression

Despite the name, **logistic regression** is a classification model, not a regression model. It predicts the probability that a sample belongs to a class.

It applies a **sigmoid function** to the linear combination of features, squashing the output to a range of 0–1:

```
P(y=1) = sigmoid(w₁x₁ + ... + wₙxₙ + b) = 1 / (1 + e^(-z))
```

The output is a probability. A threshold (typically 0.5) converts it to a class label.

Logistic regression is widely used in industry because: it's interpretable (weights indicate feature importance and direction), it outputs calibrated probabilities, it trains quickly, and it requires minimal tuning. It's often the first model tried on a new classification problem — a strong and honest baseline.

---

## Decision trees

A **decision tree** learns a sequence of yes/no questions about features, branching to different conclusions based on the answers. The resulting structure is a tree where internal nodes are conditions and leaves are predictions.

```
Is income > 50k?
├── Yes → Is credit score > 700? → Yes: Approve / No: Decline
└── No → Decline
```

Decision trees are highly interpretable — you can trace any prediction through the tree and explain every step. They handle non-linear relationships naturally, require no feature scaling, and work with mixed feature types (numerical and categorical).

Their weakness: they're prone to **overfitting**. A tree that's allowed to grow deep can memorise training data perfectly, creating complex rules that don't generalise. This is controlled through **pruning** (limiting tree depth) and by using ensemble methods.

---

## Random forests and gradient boosting

**Ensemble methods** combine multiple weak models to produce a stronger one. The two most important ensembles for classical ML are random forests and gradient boosting.

**Random forest** trains many decision trees, each on a random sample of the training data and a random subset of features. At prediction time, the trees vote (for classification) or average their outputs (for regression). The randomness prevents individual trees from overfitting in the same way, and their collective output is more robust.

Random forests are often described as the best "off the shelf" ML algorithm — they require little tuning, handle various data types well, are robust to outliers and irrelevant features, and give a rough measure of feature importance for free.

**Gradient boosting** trains trees sequentially, where each new tree corrects the errors of the previous ones. The trees are combined by summing their predictions (weighted). This sequential error-correction produces extremely powerful models.

Modern gradient boosting implementations — particularly **XGBoost**, **LightGBM**, and **CatBoost** — dominate tabular ML competitions and production systems. They consistently outperform neural networks on structured data and are the go-to choice for most industry tabular problems.

---

## Support Vector Machines (SVMs)

A **Support Vector Machine** finds the decision boundary (hyperplane) that maximally separates classes in feature space — the boundary with the largest margin between it and the nearest data points from each class. Those nearest points are the **support vectors**.

The power of SVMs comes from the **kernel trick**: by mapping features into a higher-dimensional space (without explicitly computing that space), SVMs can find non-linear decision boundaries using only dot products — making them computationally tractable.

Common kernels: linear, polynomial, radial basis function (RBF). The RBF kernel can approximate arbitrarily complex decision boundaries.

SVMs were state-of-the-art for many tasks before deep learning took over. They're still used for small-to-medium datasets, particularly when the number of features is large relative to samples (e.g. text classification with bag-of-words features), and when a clear maximum-margin classifier is desirable.

---

## k-Nearest Neighbours (k-NN)

**k-Nearest Neighbours** is one of the simplest learning algorithms: to classify a new point, find the k most similar points in the training data and take a vote.

There is no explicit training phase — the algorithm just stores all training data. The "learning" happens at prediction time (this is called a **lazy learner** or **instance-based learning**).

k-NN is intuitive and can model complex decision boundaries without any assumptions about data distribution. Its weaknesses: it's slow at prediction time on large datasets (must compare to every training point), sensitive to irrelevant features and scale differences, and requires keeping all training data in memory.

k-NN also forms the conceptual basis for **embedding-based retrieval** — finding the nearest points in a high-dimensional embedding space is essentially k-NN search — which is central to RAG systems (Phase 5).

---

## Unsupervised learning: clustering

**Clustering** groups data points by similarity without using labels. The goal is to discover natural structure in the data.

**k-Means** is the most common clustering algorithm. It partitions data into k clusters by iteratively:
1. Assigning each point to the nearest cluster centre (centroid)
2. Recomputing each centroid as the mean of all points assigned to it
3. Repeating until assignments stop changing

You must choose k in advance. Common approaches for choosing k: the "elbow method" (plot inertia vs k and look for the bend) or silhouette scores.

k-Means works well for spherical, roughly equal-sized clusters. It struggles with non-convex shapes, varying densities, and outliers.

Other clustering algorithms (DBSCAN, hierarchical clustering) handle these cases better but are less commonly used in production.

---

## Dimensionality reduction

High-dimensional data (many features) is hard to visualise, slow to process, and suffers from the **curse of dimensionality** — in high dimensions, distance metrics become less meaningful and data becomes sparse.

**Dimensionality reduction** compresses data into fewer dimensions while preserving the most important structure.

**Principal Component Analysis (PCA)** finds the directions of maximum variance in the data and projects onto those directions. The first few principal components capture most of the variance, allowing you to keep, say, 2–10 dimensions instead of 1,000.

PCA is linear — it finds linear combinations of features. **t-SNE** and **UMAP** are non-linear alternatives used primarily for visualisation. They can reveal cluster structure in high-dimensional data that PCA misses, but the reduced dimensions aren't interpretable the way PCA components are.

Dimensionality reduction also serves as a preprocessing step: reducing feature count before feeding into another model can reduce training time and regularise (reduce overfitting).

---

## Bias-variance tradeoff

This is one of the most important conceptual frameworks in ML.

**Bias** is systematic error — how wrong the model is on average. A simple model (e.g. linear regression fit to non-linear data) has high bias: it consistently gets the wrong answer because it can't capture the true pattern.

**Variance** is sensitivity to the training data — how much the model's predictions change when trained on different samples. A very complex model (e.g. a deep tree) has high variance: it fits the training data closely but its predictions fluctuate wildly depending on which specific examples it trained on.

The tradeoff:
- **Underfitting** (high bias, low variance): model is too simple, consistently wrong
- **Overfitting** (low bias, high variance): model is too complex, fits training noise

The goal is the sweet spot in the middle: a model complex enough to capture the true signal, but not so complex that it fits noise.

Controls:
- Model complexity (depth of tree, number of parameters)
- Regularisation (penalising complexity — L1/L2 regularisation are common)
- Amount of training data (more data reduces variance)
- Ensemble methods (averaging multiple models reduces variance)

This framework applies to deep learning too — neural networks overfit, regularise, and generalise by the same principles.

---

## Hyperparameters vs parameters

This distinction matters for how you think about model configuration.

**Parameters** are values the model learns during training — the weights in a linear model, the split thresholds in a decision tree.

**Hyperparameters** are settings you choose before training that control the learning process — the depth limit of a tree, the k in k-means, the learning rate in gradient descent, the regularisation strength.

Hyperparameters are not learned from data; they're tuned manually or by automated search. **Hyperparameter tuning** (also called hyperparameter optimisation) involves trying different combinations to find what works best on the validation set.

Common methods:
- **Grid search** — try every combination of specified values (exhaustive but expensive)
- **Random search** — randomly sample from specified ranges (more efficient than grid search in practice)
- **Bayesian optimisation** — model the relationship between hyperparameters and performance to intelligently explore the space

---

## Evaluation metrics

Different problems need different metrics. Choosing the wrong metric can make a bad model look good.

**For regression:**
- **Mean Absolute Error (MAE)** — average of absolute differences between predicted and actual. Intuitive, robust to outliers.
- **Mean Squared Error (MSE)** — average of squared differences. Penalises large errors more heavily.
- **Root Mean Squared Error (RMSE)** — square root of MSE. Same units as the target.
- **R² (coefficient of determination)** — proportion of variance explained by the model. 1.0 is perfect; 0.0 means the model does no better than predicting the mean.

**For classification:**
- **Accuracy** — proportion of correct predictions. Misleading on imbalanced datasets.
- **Precision** — of all positive predictions, what fraction were actually positive?
- **Recall** — of all actual positives, what fraction did the model find?
- **F1 score** — harmonic mean of precision and recall. Balances both.
- **AUC-ROC** — Area Under the Receiver Operating Characteristic Curve. Measures discrimination ability across all thresholds. 0.5 = random; 1.0 = perfect.
- **Confusion matrix** — a table showing true positives, false positives, true negatives, false negatives. The source of all classification metrics.

The right metric depends on the cost of different error types. In fraud detection, a false negative (missing a fraud) is usually far more costly than a false positive (flagging a legitimate transaction). Recall matters more. In spam filtering, a false positive (moving a real email to spam) might be more costly to the user. Precision matters more.

---

## When to use classical ML vs deep learning

A practical heuristic:

| Situation | Consider |
|-----------|----------|
| Structured/tabular data | Classical ML (gradient boosting especially) |
| Text, images, audio, video | Deep learning |
| Small dataset (< tens of thousands of samples) | Classical ML |
| Large dataset | Deep learning benefits more from scale |
| Interpretability required | Classical ML (linear models, decision trees) |
| Fast iteration, limited compute | Classical ML |
| State-of-the-art performance is essential | Likely deep learning |
| Fine-tuning a pre-trained model | Deep learning |

When in doubt, start with classical ML. Gradient boosting on tabular data is fast, powerful, and hard to beat. If you've exhausted it and need more performance, consider deep learning.

---

## Summary: the algorithm landscape

| Algorithm | Type | Strengths |
|-----------|------|-----------|
| Linear regression | Supervised (regression) | Interpretable, fast, strong baseline |
| Logistic regression | Supervised (classification) | Interpretable, probabilistic outputs, fast |
| Decision tree | Supervised (both) | Very interpretable, handles mixed data |
| Random forest | Supervised (both) | Robust, little tuning, good baseline |
| Gradient boosting (XGBoost, LightGBM) | Supervised (both) | State-of-the-art on tabular data |
| SVM | Supervised (both) | Effective in high-dimensional spaces |
| k-NN | Supervised (both) | Intuitive, no training, flexible |
| k-Means | Unsupervised (clustering) | Simple, scalable, widely used |
| PCA | Unsupervised (dim reduction) | Fast, interpretable, linear compression |

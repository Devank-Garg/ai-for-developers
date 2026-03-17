# Quiz: Classical ML Reference

Attempt these questions before checking the answers. The goal isn't to pass a test — it's to find the gaps before they become problems later.

---

## Questions

**1.** What is the key difference between supervised and unsupervised learning?

---

**2.** You want to predict whether a customer will click on an advertisement (yes/no). Is this a regression or classification problem? What if you wanted to predict the probability of clicking instead?

---

**3.** A colleague says "linear regression assumes the data is linear, so it won't work on our problem." Is this a reason to always avoid it? What would you consider before deciding?

---

**4.** You train a decision tree with no depth limit on a dataset of 10,000 samples with 50 features. The training accuracy is 100% and the test accuracy is 62%. What is happening, and how would you address it?

---

**5.** What is the key difference between a random forest and gradient boosting? Which would you typically reach for first on a new tabular classification problem?

---

**6.** A model has high bias and low variance. What does this mean practically, and what would you do?

a) The model is overfitting — reduce model complexity
b) The model is underfitting — increase model complexity or use better features
c) The model is performing well — nothing to do
d) The model has a data quality problem — clean the data

---

**7.** What is the difference between a model parameter and a hyperparameter? Give one example of each for a decision tree.

---

**8.** You're evaluating a model and get these results:
- Precision: 0.91
- Recall: 0.43
- Accuracy: 0.97

What can you infer about the model's behaviour? Is this good or bad — and does the answer depend on context?

---

**9.** When would you prefer PCA over raw features? Name one practical use case for each of its two main applications.

---

**10.** You're building a system to detect defective products on a manufacturing line. Less than 1% of products are defective. You try logistic regression, random forest, and XGBoost. All three report over 99% accuracy. What's likely happening, and what should you do instead?

---

## Answers

**1.** In **supervised learning**, the model learns from labelled data — each training example has an input and a known correct output. The model learns to map inputs to outputs. In **unsupervised learning**, there are no labels. The model finds structure, patterns, or groupings in the data without being told what to look for. The learning signal is the data itself, not human-provided correct answers.

---

**2.** Both formulations are **classification**, not regression. Even predicting probability of clicking (a value between 0 and 1) is technically a classification problem — you're modelling the probability of belonging to the positive class, which is what logistic regression does. Regression refers to predicting a continuous numerical output with no bounded range (price, temperature, duration). The output being a number between 0 and 1 doesn't make it regression if the underlying task is class membership.

---

**3.** Not necessarily. Linear regression is worth trying for several reasons: it's fast, interpretable, and a useful baseline. Even if the true relationship isn't linear, a linear model might capture enough of the signal to be useful. You can also extend it — polynomial features, interaction terms, and log transforms can capture non-linear patterns while keeping the model structure simple. The decision comes down to how non-linear the data actually is (check residuals), how much interpretability matters, and whether the accuracy gain from a non-linear model justifies the complexity.

---

**4.** This is a classic case of **overfitting**. The tree has memorised the training data — learned specific patterns, including noise, that don't generalise to unseen examples. A tree with no depth limit can grow until every training example is at its own leaf, achieving 100% training accuracy trivially. The 38-point gap between training (100%) and test (62%) accuracy confirms overfitting. Fixes: limit tree depth (max_depth hyperparameter), require minimum samples per leaf, apply pruning, or — better — switch to a random forest or gradient boosting model, which are specifically designed to address this.

---

**5.** The key difference is how they combine trees:
- **Random forest** builds trees **in parallel**, each on a random subset of data and features. Final prediction is a vote or average. The randomness and independence reduce variance.
- **Gradient boosting** builds trees **sequentially**, each one correcting the errors of the previous ensemble. This iterative error correction produces a highly accurate but potentially more overfit model.

In practice: **start with a random forest** for quick, robust results with minimal tuning. Move to gradient boosting (specifically LightGBM or XGBoost) when you need maximum performance and are willing to tune. Gradient boosting typically outperforms random forests but requires more careful hyperparameter selection.

---

**6.** b) The model is underfitting — increase model complexity or use better features. High bias means the model consistently makes systematic errors — it can't capture the true pattern in the data. Low variance means predictions are stable across different training samples, but they're stably wrong. Solutions: use a more complex model (more tree depth, more features, a non-linear algorithm), add more informative features, or reduce regularisation. Don't confuse this with data quality — bias-variance is about model capacity relative to problem complexity.

---

**7.** **Parameters** are learned during training — they're what the model adjusts to fit the data. **Hyperparameters** are set before training and control the learning process.

For a decision tree:
- **Parameter example**: the specific feature and threshold value at a split node (e.g. "is age > 35?") — learned from data during training.
- **Hyperparameter example**: `max_depth` (maximum tree depth) or `min_samples_leaf` (minimum samples required at a leaf) — set by the developer before training, not learned.

---

**8.** The model has high precision (91%) but low recall (43%). This means: when it predicts positive, it's usually right (91% of positive predictions are correct), but it's only catching 43% of actual positives (missing 57% of them). The high accuracy (97%) is likely inflated by the majority class.

Whether this is good or bad **depends entirely on the cost of each error type**. If false negatives are costly (e.g. detecting cancer, flagging fraud), low recall (0.43) is unacceptable — you're missing more than half your positives. If false positives are very costly (e.g. blocking legitimate transactions), high precision is more important. Always define what failure mode is more acceptable before choosing your success metric.

---

**9.** PCA is useful when: features are highly correlated (it removes redundancy), you have many more features than samples (reducing dimensionality helps generalisation), or you need to reduce computational cost.

Two practical use cases:
1. **Preprocessing for another model** — reduce 500 features to 20 principal components before training a classifier. Less data to fit, faster training, potentially less overfitting.
2. **Visualisation** — reduce high-dimensional data (e.g. 300-dimensional word embeddings) to 2 or 3 dimensions for plotting. This lets you visually inspect cluster structure, outliers, and class separability.

Note: PCA components aren't directly interpretable as features — you trade interpretability for compression.

---

**10.** This is the **class imbalance problem**. With less than 1% defective products, a model that always predicts "not defective" achieves 99%+ accuracy without learning anything. All three models are likely doing exactly this — or something close to it — because accuracy rewards them for ignoring the minority class.

What to do:
- Switch to appropriate metrics: **recall** (what fraction of defects did we catch?), **precision**, **F1**, and **AUC-ROC**
- Handle the imbalance: use class weighting (`class_weight='balanced'` in scikit-learn), oversample the defect class (SMOTE), or undersample the majority
- Focus on the confusion matrix: you want to see that the model actually makes some positive predictions, not just classify everything as negative

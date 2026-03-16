# Concepts: Data Fundamentals

---

## Why data is the actual product

There's a saying in the field: "data is the moat." The architecture of a model matters. The training process matters. But in most real-world applications, the quality, size, and relevance of your training data is the single biggest determinant of how well a model performs.

This is counterintuitive for developers who are used to thinking in terms of code and logic. In software, you write better rules and you get better behaviour. In machine learning, you provide better examples and the system learns better patterns. The data *is* the program, in a sense.

This module covers what data means in an ML context — how it's structured, what makes it good or bad, and what you need to understand before building or deploying any system that learns from it.

---

## Structured vs unstructured data

**Structured data** is organised into rows and columns with defined types — spreadsheets, relational databases, CSV files. Each column has a clear meaning (age, price, category), and the values are consistent and comparable. Most classical ML algorithms expect structured data as input.

**Unstructured data** has no predefined schema. Text, images, audio, and video are unstructured. There are no rows and columns — just raw content. Deep learning, and particularly LLMs, are responsible for most of the progress in making unstructured data usable for ML.

**Semi-structured data** falls in between — JSON, XML, log files. It has some organisation (key-value pairs, hierarchies) but not the rigid grid of a relational table.

Most interesting real-world ML problems involve unstructured or semi-structured data. Most of the hard work is converting that data into a form a model can learn from.

---

## Features and labels

In supervised learning — the most common ML paradigm — your dataset consists of **features** and **labels**.

**Features** (also called **inputs**, **predictors**, or **attributes**) are the variables you use to make a prediction. If you're predicting house prices, features might include square footage, number of bedrooms, neighbourhood, and year built.

**Labels** (also called **targets** or **outputs**) are what you're trying to predict. In the house price example, the label is the sale price.

A single training example is called a **sample** or **data point**. Each sample has a feature vector (all its feature values) and a label.

```
Sample: [2400 sqft, 3 beds, "Bandra", 2015] → label: ₹1.8 crore
```

The model learns the mapping from feature vectors to labels across many training examples. At inference time, you provide features without a label, and the model predicts what the label should be.

---

## Feature engineering

**Feature engineering** is the process of creating, selecting, or transforming features to make them more useful to a model.

Raw data is rarely ready to feed into a model. You often need to:

- **Create new features** from existing ones. If you have date of birth, you might derive "age". If you have timestamps for two events, you might derive "time elapsed between them".
- **Encode categorical variables.** Most models expect numbers, not strings. A "colour" column with values "red", "green", "blue" needs to be converted to a numerical representation. Common approaches: **one-hot encoding** (a separate 0/1 column for each category) or **ordinal encoding** (mapping categories to integers).
- **Bin continuous variables.** Sometimes it's more useful to convert a continuous number (e.g. age) into discrete buckets (18–25, 26–35, 36–50) to capture non-linear relationships.
- **Scale and normalise** (covered below).
- **Remove irrelevant features.** Including features that carry no signal — or that would cause data leakage (see below) — can hurt model performance.

In classical ML, feature engineering is where most of the skill and effort lives. Deep learning automates much of it — neural networks learn their own internal representations — but it never goes away entirely.

---

## Data preprocessing: scaling and normalisation

Most ML algorithms are sensitive to the scale of input features. If one feature ranges from 0–1 and another ranges from 0–1,000,000, the model may give disproportionate weight to the larger-scale feature.

Two common approaches:

**Normalisation** (also called **min-max scaling**) rescales a feature to a fixed range, typically 0–1:

```
x_scaled = (x - x_min) / (x_max - x_min)
```

**Standardisation** (also called **z-score normalisation**) centres the feature at zero with a standard deviation of 1:

```
x_scaled = (x - mean) / std_dev
```

Standardisation is generally more robust to outliers. Normalisation preserves the original distribution shape but is sensitive to extreme values.

Note: you fit the scaling parameters (min, max, mean, std_dev) on the **training set only**, then apply them to the validation and test sets. If you fit on all the data, you leak information about the test set into the preprocessing — a common mistake.

---

## Missing data

Real-world datasets almost always have missing values. A sensor that occasionally fails, a survey question that was skipped, a field that wasn't required — missing data is the rule, not the exception.

You have a few options:

**Dropping rows or columns** — if a feature has very few non-null values, it may not be worth keeping. If a small number of samples have missing values, dropping them may be acceptable.

**Imputation** — filling in missing values with an estimate. Common strategies: replace with the mean or median (for numerical features), replace with the most frequent value (for categorical features), or use a more sophisticated model to predict the missing value from other features.

**Treating "missing" as a value** — sometimes the absence of a value is itself informative. A missing income field might mean the respondent declined to answer, which is different from a random data collection error.

The choice depends on why data is missing (missing at random vs not at random), how much is missing, and how sensitive your model is to the imputation strategy.

---

## Outliers

**Outliers** are data points that deviate significantly from the rest of the data. A house listed at ₹50 crore in a dataset of ₹50 lakh homes is an outlier. A sensor reading of 10,000°C in a dataset of room temperatures is almost certainly an error.

Outliers can distort model training — particularly models that are sensitive to scale, like linear regression. They need to be identified and handled deliberately:

- **Correct them** if they're errors (data entry mistakes, sensor failures)
- **Remove them** if they represent cases genuinely outside the model's intended scope
- **Keep them** if they're legitimate and important to model correctly
- **Use robust algorithms** that are less sensitive to outliers

Never blindly remove outliers. An outlier might be the most important signal in your data — fraud transactions, for example, are statistical outliers by definition.

---

## Data quality: the practical reality

A useful heuristic: in most production ML projects, 60–80% of the time is spent on data. Not on model architecture, not on tuning hyperparameters — on getting clean, usable data.

Common data quality problems:

**Noise** — random error or variation in the data. A sensor that's slightly miscalibrated, or a labeller who occasionally makes mistakes. Some noise is unavoidable; too much degrades model performance.

**Inconsistent formats** — dates as "2024-01-15" in one source and "15/01/24" in another. Currency in different denominations. Text in mixed case. These need standardisation.

**Duplicates** — the same record appearing multiple times, which can introduce bias.

**Label errors** — incorrect labels in a supervised dataset. If your spam filter's training data has ham labelled as spam, the model learns the wrong signal. Label quality matters as much as feature quality.

**Concept drift** — the statistical properties of the data change over time. A model trained on 2020 data may not perform well on 2024 data if the underlying patterns have shifted. This is one of the main reasons models degrade in production.

---

## Class imbalance

In classification problems, **class imbalance** occurs when one class has far more examples than others. A fraud detection dataset where 99.9% of transactions are legitimate and 0.1% are fraudulent is severely imbalanced.

This creates a problem: a model can achieve 99.9% accuracy by predicting "not fraud" for every transaction — without learning anything meaningful.

Strategies for handling class imbalance:

- **Oversample the minority class** — duplicate or synthetically generate examples of the under-represented class (e.g. **SMOTE**, Synthetic Minority Over-sampling Technique)
- **Undersample the majority class** — randomly remove examples from the over-represented class
- **Use class weights** — tell the model to penalise errors on the minority class more heavily during training
- **Choose appropriate metrics** — accuracy is misleading on imbalanced data. Use precision, recall, F1 score, or AUC-ROC instead.

---

## Data splits: training, validation, and test

This was introduced in the terminology module, but it's worth reinforcing here.

The reason for splitting data is to get an honest estimate of how well your model will perform on data it has never seen. If you evaluate on the same data you trained on, you're measuring memorisation, not generalisation.

**Training set** — what the model learns from directly. Typically 70–80% of your data.

**Validation set** — used during training to monitor progress, tune hyperparameters, and make decisions about the model architecture. The model doesn't train on this data, but your decisions as a developer are influenced by it — which creates indirect leakage over time.

**Test set** — held out completely until your final evaluation. This is your honest estimate of real-world performance. Touch it as rarely as possible.

A useful analogy: the training set is your study material, the validation set is your practice exams, and the test set is the real exam.

**Cross-validation** is an alternative approach, particularly useful when data is scarce. You train and evaluate multiple times on different splits of the data, rotating which portion is held out. The most common form is **k-fold cross-validation**: split the data into k equal parts, train on k−1 parts, evaluate on the remaining one, and repeat for all k combinations. The results are averaged.

---

## Data leakage

**Data leakage** is when information from outside the training data's intended scope is accidentally included in training, causing the model to learn patterns it shouldn't have access to in production.

It's one of the most common and damaging errors in ML, and it's subtle. Examples:

- Scaling your features using statistics from the full dataset (including test data) before splitting
- Including a feature that's derived from the label (e.g. including "hospital discharge diagnosis" as a feature when predicting hospital admission outcome)
- Using future data to train a model that's supposed to make predictions about the past (common in time-series problems)

Leakage makes models look dramatically better during evaluation than they perform in production. The fix is disciplined data management: always apply transformations based only on training data, and be rigorous about what information would realistically be available at the time of prediction.

---

## Data bias

A model learns the patterns in its training data. If that data reflects historical biases, the model will reproduce and often amplify them.

**Representation bias** — certain groups or scenarios are underrepresented in the data. A face recognition system trained mostly on lighter-skinned faces will perform worse on darker-skinned faces.

**Historical bias** — the data reflects decisions that were themselves biased. A hiring model trained on historical acceptance decisions will learn to replicate whatever biases existed in those decisions.

**Measurement bias** — the data collection process introduces systematic error. If certain populations are more likely to opt out of data collection, the dataset won't represent them accurately.

Bias in ML is not primarily a technical problem — it's a data and framing problem. The choice of what to measure, how to label it, and whose behaviour is recorded all shape what a model learns. This is covered more fully in Phase 6.

---

## Data pipelines

In production, data doesn't appear magically in the right format. A **data pipeline** is the set of processes that ingest raw data, clean it, transform it, and deliver it to the model in the right shape.

A typical pipeline might:

1. Ingest raw data from multiple sources (databases, APIs, files)
2. Clean and validate (handle missing values, outliers, format inconsistencies)
3. Feature engineer (create derived features, encode categories, scale values)
4. Split into training/validation/test sets
5. Store in a format the model can consume (tensors, NumPy arrays, feature stores)

The pipeline itself needs to be versioned and reproducible. If you change your preprocessing and can't reproduce it exactly at inference time, your model will see different data distributions in production than it was trained on — a source of subtle, hard-to-debug failures.

---

## Summary: the core concepts

| Concept | What to remember |
|---------|-----------------|
| Structured vs unstructured | Structured = tables; unstructured = text, images, audio |
| Features and labels | Features are inputs; labels are what you're predicting |
| Feature engineering | Transforming raw data into signals a model can learn from |
| Scaling/normalisation | Needed for most algorithms; fit on training data only |
| Missing data | Must be handled deliberately — impute, drop, or encode as absent |
| Outliers | Identify the cause before deciding how to handle |
| Class imbalance | Accuracy is misleading; use appropriate metrics and resampling |
| Data leakage | Information the model shouldn't have at prediction time — corrupts evaluation |
| Data bias | Models inherit the biases in their training data |
| Data pipelines | The infrastructure that prepares data reproducibly for training and inference |

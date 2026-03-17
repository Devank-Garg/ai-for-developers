# Quiz: Data Fundamentals

Attempt these questions before checking the answers. The goal isn't to pass a test — it's to find the gaps before they become problems later.

---

## Questions

**1.** A dataset contains customer records with columns for age, annual income, city, and purchase history. A second dataset contains raw customer support chat logs. Which is structured and which is unstructured? What makes them different?

---

**2.** You're building a model to predict employee attrition (whether an employee will leave the company). Your dataset includes employee tenure, department, salary band, and performance rating. Which are features and which is the label?

---

**3.** Your training dataset has a "date_of_birth" column. Your model needs to predict health risks. What feature engineering step would make this more useful, and why?

---

**4.** You're normalising a feature in your training set and get a min of 10 and max of 500. You then apply normalisation to your test set using those same values. During production, a new data point arrives with a value of 650. What happens, and is this the right approach?

---

**5.** A dataset of loan applications has 12% missing values in the "employment_years" column. What are three different strategies you could use, and what would influence your choice?

---

**6.** A fraud detection model is trained on a dataset where 99.8% of transactions are legitimate. The model achieves 99.8% accuracy on the test set. Is this a good model?

---

**7.** Which of the following is an example of data leakage?

a) Including a customer's age as a feature when predicting whether they'll churn
b) Using the same scaling parameters for training and test data
c) Including a customer's account closure date as a feature when predicting whether they'll close their account
d) Training on 80% of the data and testing on 20%

---

**8.** You train a model to predict creditworthiness. The model performs well in testing, but you later find it systematically denies loans to applicants from certain zip codes at much higher rates than others. What is this an example of, and where might it have come from?

---

**9.** What is the difference between cross-validation and a standard train/test split, and when would you prefer cross-validation?

---

**10.** A model performs very well in testing but degrades significantly six months after deployment, even though the code hasn't changed and no retraining has occurred. What is the likely cause?

---

## Answers

**1.** The customer records table is **structured** — data organised into rows and columns with defined types and meanings. The chat logs are **unstructured** — free-form text with no schema. The key difference is whether there's a predefined format that makes values directly comparable and machine-readable. Structured data is directly usable by most classical ML algorithms; unstructured data requires preprocessing (embeddings, feature extraction) or deep learning architectures.

---

**2.** The **features** are: tenure, department, salary band, and performance rating — these are the inputs the model uses to make its prediction. The **label** is attrition (whether the employee left or not) — this is what the model is trying to predict. At inference time, you provide features for a current employee and the model predicts their probability of leaving.

---

**3.** Derive an **age** feature from date_of_birth. Raw dates are not meaningful to most models — the difference from today (current age) is what carries predictive signal. Additionally, the model would have no way to generalise across different birth dates without this transformation. As a bonus, you might also derive features like "age group" (binned) if you suspect non-linear relationships with health outcomes.

---

**4.** The value 650 is beyond the original maximum (500), so it will normalise to a value greater than 1. This is expected and acceptable — using training set statistics for test/production data is the **correct approach**. Refitting the scaler on test data would constitute data leakage. In production, you must accept that values outside the training range can occur; models typically handle this gracefully, though extreme out-of-distribution values may reduce prediction quality.

---

**5.** Three strategies:
- **Impute with median** — replace missing values with the median employment years from the training set. Appropriate if data is missing at random and the median is a reasonable proxy.
- **Add a missingness indicator** — create a binary column "employment_years_missing" alongside imputed values. Appropriate if the fact that data is missing is itself predictive (e.g. self-employed people may not have filled this in).
- **Drop the column** — if missingness is too high or the feature adds little signal. Check feature importance first. Influence of choice depends on: how much data is missing, whether missingness is random or systematic, and how important the feature is to model performance.

---

**6.** No. A model that predicts "legitimate" for every transaction would also achieve 99.8% accuracy, having learned nothing useful. This is the **class imbalance** problem. You should evaluate using metrics appropriate for imbalanced classes: **precision** (of all predicted frauds, how many were actually fraud), **recall** (of all actual frauds, how many did we catch), **F1 score** (harmonic mean of precision and recall), or **AUC-ROC**. A model with 99.8% accuracy and 0% recall on fraud is useless.

---

**7.** c) Including a customer's account closure date as a feature when predicting whether they'll close their account. Account closure date only exists *after* the event you're trying to predict — it's future information being used to predict the present. At actual prediction time, this feature wouldn't exist for active customers. This is data leakage. Option (b) describes a mistake with scaling, but applying the same (training-derived) scaling parameters to test data is actually correct.

---

**8.** This is an example of **data bias** — specifically, likely a combination of **historical bias** and **representation bias**. The model may have learned from historical lending decisions that themselves reflected discriminatory practices (e.g. redlining). Or certain zip codes may have been underrepresented in the training data. Or proxy features (income, employment type) are correlated with zip code in ways the model exploits. The technical model appears to work, but the outcome reflects and amplifies pre-existing inequality. This is a core concern in responsible AI and is covered in Phase 6.

---

**9.** A standard train/test split uses the data once — train on part, evaluate on the rest. **Cross-validation** (typically k-fold) trains and evaluates multiple times on different portions of the data, then averages the results. You would prefer cross-validation when: your dataset is small (a single split would be noisy), you want a more reliable estimate of generalisation performance, or you're comparing multiple models or hyperparameter settings and need robust estimates. The downside of cross-validation is computational cost — you're training k models instead of one.

---

**10.** This is **concept drift** — the statistical relationship between features and labels has changed over time, but the model hasn't been updated to reflect the new reality. The patterns the model learned during training no longer hold in the current data. Examples: customer behaviour shifted, market conditions changed, a new product category was introduced. Mitigation involves monitoring model performance metrics in production and retraining on fresh data periodically or when drift is detected.

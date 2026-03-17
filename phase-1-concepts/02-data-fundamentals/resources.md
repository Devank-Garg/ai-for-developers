# Resources: Data Fundamentals

These are curated external resources if you want to go deeper on any concept from this module. None are required — the concepts file covers everything you need to continue the course.

---

## Foundational overviews

**Google Machine Learning Crash Course — Data Preparation**
https://developers.google.com/machine-learning/crash-course/data-prep/video-lecture
Google's free ML crash course has a dedicated section on data preparation, covering representation, feature engineering, and data quality. Practical, well-paced, and free.

**Kaggle Learn — Intro to Machine Learning (Data exploration chapters)**
https://www.kaggle.com/learn/intro-to-machine-learning
Hands-on notebooks that teach data exploration and feature understanding through live datasets. Good for building intuition by actually touching data rather than just reading about it.

---

## Specific topics

**Feature Engineering**
*Feature Engineering for Machine Learning* by Alice Zheng and Amanda Casari (O'Reilly) — the most thorough book on the subject. If you're working in production ML and feature quality matters (it always does), this is worth reading. Preview chapters are sometimes available on the O'Reilly site.

**Data Leakage**
"Leakage in data mining: Formulation, detection, and avoidance" (Kaufman et al.) — academic but readable. Also: Kaggle's data leakage guide (search "Kaggle data leakage") gives practitioner-focused examples of how leakage sneaks in and how to catch it.

**Class Imbalance**
"Learning from Imbalanced Data" — search for the survey paper by Haibo He and E.A. Garcia. The imbalanced-learn Python library (https://imbalanced-learn.org) also has excellent documentation that doubles as a tutorial.

**Concept Drift**
Evidently AI blog (https://evidentlyai.com/blog) — practical writing on monitoring ML models in production, including concept drift detection. Particularly useful when you reach Phase 5 (Building Applications).

---

## Interactive tools

**Pandas Profiling / ydata-profiling**
https://github.com/ydataai/ydata-profiling
A Python library that generates a full exploratory data analysis report from a DataFrame in a single line of code. Run this on any new dataset to quickly understand distributions, missing values, correlations, and outliers.

**Facets (Google)**
https://pair-code.github.io/facets/
A browser-based tool for visualising and understanding datasets. Particularly useful for spotting class imbalances, feature distributions, and representation gaps.

---

## Reference

**Scikit-learn preprocessing documentation**
https://scikit-learn.org/stable/modules/preprocessing.html
The definitive reference for scaling, encoding, and imputation in Python. Even if you use other frameworks, scikit-learn's preprocessing API is the standard that others mirror.

**tidy data (Hadley Wickham, 2014)**
https://vita.had.co.nz/papers/tidy-data.pdf
A seminal paper on principles of clean data organisation. Although it's from the R/statistics world, the principles apply directly to preparing data for ML. Short and worth reading.

---

## If you want to go much deeper

**"The Data Cascade" (Sambasivan et al., 2021)**
Search for this paper — it documents how data quality issues compound through ML pipelines in production and cause real-world failures. A grounding read on why data quality is treated as an engineering discipline, not an afterthought.

**Designing Machine Learning Systems (Chip Huyen)**
https://www.oreilly.com/library/view/designing-machine-learning/9781098107956/
One of the best books on production ML. The chapters on data engineering and feature management are especially relevant to this module. The author's blog (huyenchip.com) also has freely available posts on many of the same topics.

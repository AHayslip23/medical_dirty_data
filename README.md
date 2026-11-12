# medical_dirty_data
Python practice on dirty data with medical information.

This project takes a messy patient dataset (`realworld_medical_dirty.csv`, 100 rows), cleans it with pandas, engineers new features, explores the results with matplotlib and seaborn, clusters patients with K-Means, and builds and validates classification models with scikit-learn. All work is in `Medical Data Practice.ipynb`.

## Dataset

| Column | Description |
|---|---|
| Patient_ID | Patient identifier (dropped as PII) |
| Age | Patient age |
| Gender | Patient gender, entered inconsistently (`Male`, `MALE`, `male`, `FEMALE`, blanks) |
| Blood_Pressure | Systolic blood pressure (mmHg) |
| Cholesterol | Total cholesterol (mg/dL) |
| BMI | Body mass index |
| Smoker | Smoking status (`Yes`, `No`, `Y`, `N`, blanks) |
| Diagnosis | Diagnosis, if one was given |
| Admission_Date | Hospital admission date (all in 2023) |
| Notes | Follow-up notes |

## Commit History

### Initial commit (Sep 24, 2026)
- Created the repository with a starter README and `.gitattributes`.

### Commit #1: Data cleaning (Sep 24, 2026)
Uploaded the raw dataset and the first version of the notebook, focused on cleaning.

- **Removed PII:** dropped the `Patient_ID` column.
- **Dates:** converted `Admission_Date` to datetime and created `Month Year` and `Year` columns from it.
- **Categorical nulls:** filled missing `Notes`, `Diagnosis` and `Smoker` values with descriptive labels, and standardized `Smoker` (`Y`/`N` → `Yes`/`No`).
- **Gender cleanup:** filled blanks with `Unknown` and standardized inconsistent capitalization (`MALE`, `male`, `FEMALE`) into `Male` / `Female`.
- **Numeric missing-data analysis:** measured null counts for `Age`, `Blood_Pressure`, `Cholesterol` and `BMI` (14–20% missing each), then tested whether the missingness was related to other columns:
  - chi-square tests against categorical columns
  - Mann-Whitney U tests against the other numeric columns
  - correlation of missingness between columns
- **Median imputation:** filled numeric nulls with each column's median, and added `*_was_missing` indicator columns first so imputed values can still be identified.

### Commit #2: Feature engineering and exploratory data analysis (Nov 10, 2026)
Finished feature engineering and began exploratory data analysis.

**Cleaning refinements**
- Shortened the fill-in labels (`No Follow Up Notes`, `No Diagnosis`, `Not Sure`, `Unknown`).
- Reformatted `Month Year` as `"January 2023"` style text and converted `Year` to an integer.

**Feature engineering**
- **Gender dummies:** created `Gender_Female`, `Gender_Male` and `Gender_Unknown` boolean columns, keeping the original `Gender` column.
- **Missing-information flags:** `Missing Follow Up Notes`, `Smoking Diagnosis is Unclear` and `Missing Diagnosis`.
- **Clinical categories:** wrote classification functions for:
  - `BP_Category`: AHA systolic categories (Normal → Hypertensive Crisis)
  - `Cholesterol_Category`: Desirable / Borderline High / High
  - `BMI_Category`: Underweight / Normal / Overweight / Obese
  - `Age_Group`: age bands (18-29 through 65+)
- **Outlier check:** wrote an IQR-based function to count outliers in each numeric column.

**Exploratory data analysis**
- Histograms of the four numeric columns.
- Correlation heatmap of the numeric columns (no strong correlations found).
- Pivot tables of smoking status by diagnosis and by gender, with percent smoking.
- Line charts of monthly hospitalizations in 2023, split into younger (18-49) and older (55+) adults.
- Count plot of patients by diagnosis.
- Boxplot of age by smoking status.
- Scatter plot of BMI vs. blood pressure, colored by blood pressure category.

### Commit #3: More EDA, K-Means clustering and classification (Nov 11, 2026)
Final commit. Added more exploratory charts, clustered patients by blood pressure and cholesterol, and built and validated classification models.

**Exploratory data analysis**
- Line chart of monthly hospitalizations by gender.
- Line chart of total monthly hospitalizations for 2023.

**K-Means clustering (Blood Pressure × Cholesterol)**
- Used the elbow method (WCSS for k = 1–10) to choose the number of clusters, and settled on k = 6.
- Fit K-Means, reviewed the centroids, and added `Cluster` and `Cluster_Label` columns to the dataset.
- Plotted the clusters with their centroids and summarized each cluster's blood pressure and cholesterol (count, mean, std, min, max).

**Classification: predicting smoking status (Yes vs. No, 80 patients)**
- **Features:** Age, Blood_Pressure, Cholesterol, BMI, gender dummies and one-hot encoded Diagnosis. Patients with an unknown smoking status were excluded.
- **Setup:** 80/20 stratified train/test split and 10-fold stratified cross-validation, scored on ROC AUC, accuracy, recall and precision.
- **Model comparison:** Logistic Regression, Decision Tree, KNN, SVM (RBF), Random Forest, Histogram Gradient Boosting and XGBoost. Cross-validated ROC AUC ranged from 0.25 (KNN) to 0.58 (Decision Tree).
- **Checking the best model:** before tuning the decision tree, tested whether its score was real:
  - permutation test: ROC AUC 0.582 vs. 0.497 with shuffled labels, p = 0.154 (not significant)
  - repeated CV across 10 different fold splits: 0.42–0.58
- **Tuning:** grid search over `max_depth`, `min_samples_leaf`, `criterion` and `ccp_alpha`, with nested cross-validation for an honest estimate:

  | Score | ROC AUC |
  |---|---|
  | Best grid-search score (optimistic) | 0.597 |
  | Nested CV (honest estimate) | 0.518 ± 0.191 |
  | Held-out test set | 0.453 |

- **Feature importance:** the tuned tree split mostly on BMI (36%), Age (27%) and Blood Pressure (18%).

**Lessons from the validation work**
- A calibrated SVM appeared to score 0.70, but the calibration step was inverting a below-chance score. The plain SVM scored about 0.30, so the comparison uses the plain SVM.
- An earlier decision tree score of 0.617 didn't hold up across other fold splits, so it was a lucky split rather than a real pattern.
- Running nested CV with `n_jobs=-1` on both the inner and outer loops made the cell hang. Parallelizing only the outer loop fixed it.

## Project Conclusion
- **Cleaning and feature engineering:** the dataset was cleaned of PII, inconsistent labels and missing values. Imputation was tracked with `_was_missing` flags, and clinical categories, age groups and clusters were added.
- **Modeling:** no model reliably predicted smoking status (nested CV ROC AUC ≈ 0.52, permutation p > 0.05). Side analyses found the same for predicting a missing diagnosis, other category targets, and regression on the numeric columns. Each numeric column takes only 5 distinct values, which suggests the dataset is synthetic, with no real relationships between columns. The feature importances above describe how the tree split the training data, not real predictors of smoking.
- **Takeaway:** this dataset works well for practicing data cleaning, EDA and model validation, but not for building a predictive model. The same pipeline (model comparison, nested CV, permutation tests) can be reused on a dataset with real signal, such as UCI Heart Disease or the Medical Cost Personal dataset.

## Tools
Python, pandas, NumPy, SciPy, scikit-learn, XGBoost, matplotlib, seaborn, Jupyter Notebook

# medical_dirty_data
Python practice on dirty data with medical information.

This project takes a messy patient dataset (`realworld_medical_dirty.csv`, 100 rows), cleans it with pandas, engineers new features, and explores the results with matplotlib and seaborn. All work is in `Medical Data Practice.ipynb`.

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

## Tools
Python, pandas, NumPy, SciPy, matplotlib, seaborn, Jupyter Notebook

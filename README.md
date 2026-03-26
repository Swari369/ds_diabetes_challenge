# Pima Indians Diabetes — Classification Challenge

## Project Overview

This project tackles a **binary classification problem** using the Pima Indians Diabetes Database. The objective is to predict whether a patient has diabetes or not, based on a set of diagnostic medical measurements. The data is retrieved from a PostgreSQL database, preprocessed, and evaluated across multiple machine learning algorithms to identify the best-performing model.

---

## Dataset

- **Source:** Pima Indians Diabetes Database — originally from the National Institute of Diabetes and Digestive and Kidney Diseases.
- **Reference:** Smith, J.W., et al. (1988). *Using the ADAP learning algorithm to forecast the onset of diabetes mellitus.* Proceedings of the Symposium on Computer Applications and Medical Care. IEEE.
- **Population:** Female patients, minimum age 21, of Pima Indian heritage.
- **Records:** 768 patients
- **Target Variable:** `outcome` — `1` (Diabetes) / `0` (No Diabetes)

### Features

| Feature | Description |
|---|---|
| `age` | Age of the patient (years) |
| `pregnancies` | Number of times pregnant |
| `bmi` | Body Mass Index |
| `bloodpressure` | Diastolic blood pressure (mm Hg) |
| `glucose` | Plasma glucose concentration (2-hour oral glucose tolerance test) |
| `insulin` | 2-hour serum insulin (mu U/ml) |
| `diabetespedigreefunction` | Likelihood of diabetes based on family history |
| `skinthickness` | Triceps skin fold thickness (mm) |

---

## Database Structure

Data is loaded from a **PostgreSQL database** using the `diabetes` schema. The schema consists of 4 tables joined via patient ID:

```sql
SELECT
    p."Age", p.pregnancies, p.bmi,
    bm.bloodpressure, bm.glucose, bm.insulin,
    po.diabetespedigreefunction, po.outcome,
    s.skinthickness
FROM patient p
JOIN blood_metrics bm     ON p.id = bm.patientid
JOIN pedigree_outcome po  ON p.id = po.patientid
JOIN skin s               ON p.id = s.patientid;
```

| Table | Contents |
|---|---|
| `patient` | Age, pregnancies, BMI |
| `blood_metrics` | Blood pressure, glucose, insulin |
| `pedigree_outcome` | Diabetes pedigree function, outcome label |
| `skin` | Skin thickness |

---

## Project Structure

```
.
├── data/
│   └── ml_diabetes.csv              # Exported dataset from database
├── diabetes_cleaned_data.csv        # Final cleaned dataset
├── 2_Diabetes_Challenge.ipynb       # Main notebook
└── README.md                        # Project documentation
```

---

## Libraries Used

```python
pandas
numpy
psycopg2
python-dotenv
scikit-learn
matplotlib
seaborn
```

---

## Methodology

### 1. Data Extraction
- Connected to a PostgreSQL database using environment variables (`.env` file).
- Joined 4 tables to form a single flat dataset using SQL.
- Exported the result to `data/ml_diabetes.csv`.

### 2. Data Cleaning
- **Zero-value replacement:** Columns like `bmi`, `bloodpressure`, `glucose`, `insulin`, and `skinthickness` cannot realistically be 0. These were replaced with `NaN` to reflect true missing values.
- **Imputation:** Missing values were filled using **median imputation** (robust to skewed distributions).

### 3. Exploratory Analysis
- Plotted histograms of all numerical features.
- Used boxplots to detect outliers in `bmi`, `bloodpressure`, `glucose`, and `skinthickness`.
- Verified data logic (e.g., BMI vs. Skin Thickness correlation).

### 4. Preprocessing Pipeline
All models used a consistent `sklearn` Pipeline:

```
SimpleImputer (strategy='median')
     ↓
StandardScaler / RobustScaler
     ↓
Classifier
```

- An **80/20 stratified train-test split** was applied (`random_state=42`).
- `RobustScaler` was also tested as an outlier-resistant alternative to `StandardScaler`.

### 5. Model Training & Evaluation
All models were evaluated using **5-fold cross-validation** on the training set and then assessed on the held-out test set.

**Evaluation Metric: Accuracy** (with classification reports and confusion matrices for deeper analysis).

---

## Models & Results

| Model | CV Accuracy |
|---|---|
| **Random Forest (Tuned)** | **75.98%** ✅ Best |
| SVM (RBF Kernel) | 70.52% |
| SVM (Linear) | 69.79% |
| KNN (k=5) | 69.62% |
| Logistic Regression | 69.54% |
| Decision Tree | 66.61% |

### Hyperparameter Tuning (Random Forest)
A `GridSearchCV` was run over the following grid:

```python
param_grid = {
    'classifier__n_estimators': [50, 100, 200],
    'classifier__max_depth': [None, 10, 20],
    'classifier__min_samples_split': [2, 5]
}
```

---

## Key Insights

- **Glucose** is the most important predictor of diabetes across both the Decision Tree and Random Forest models.
- **BMI** and the **Diabetes Pedigree Function** are the next most influential features.
- The **Random Forest (Tuned)** model is the best performer, achieving ~76% cross-validation accuracy.
- Applying `RobustScaler` in place of `StandardScaler` improves resilience to outliers detected during EDA.
- A **Decision Tree visualisation** (max depth = 3) reveals that glucose alone provides the primary split in the model's logic.

---

## How to Run

1. Clone the repository and navigate to the project folder.
2. Copy your `.env` file (with database credentials) into the project root.
3. Install the required libraries:
   ```bash
   pip install pandas numpy psycopg2-binary python-dotenv scikit-learn matplotlib seaborn
   ```
4. Open and run the notebook:
   ```bash
   jupyter notebook 2_Diabetes_Challenge.ipynb
   ```

> **Note:** If you do not have access to the PostgreSQL database, use the pre-exported CSV file at `data/ml_diabetes.csv` by loading it directly with `pd.read_csv('data/ml_diabetes.csv')`.

---

## Author

**Hardik**
Diabetes Classification Challenge — Pima Indians Diabetes Database

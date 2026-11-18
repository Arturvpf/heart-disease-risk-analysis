# Heart Disease Risk Pattern Analysis

> Academic project developed for the **Statistics and Probability for Computer Science** course — CIn/UFPE

---

## Overview

This project performs an end-to-end **statistical and probabilistic analysis** and builds leak-free **Naive Bayes** screening classifiers to identify **10-year coronary heart disease (CHD) risk patterns** from clinical, demographic, and behavioral patient data in the **Framingham Heart Study** cohort (`Heart_Disease_Dataset.csv`, $N = 4{,}240$ patients, 16 variables).

The target variable is `CHDRisk` (10-year coronary heart disease incidence: `yes` = 15.2%, `no` = 84.8%).

---

## Team

| Name | Email |
|------|-------|
| Artur Fernandes | arturvpf3@gmail.com |
| Breno Ramos | brsg@cin.ufpe.br |
| Caio Vilas Boas | ccnvb@cin.ufpe.br |
| João Nascimento | jvsn2@cin.ufpe.br |

---

## Repository Structure

```text
.
├── data/
│   └── Heart_Disease_Dataset.csv      # Framingham Heart Study cohort (4,240 records, 16 variables)
├── heart-disease-risk-analysis.ipynb  # Full analysis: preprocessing, EDA, statistical tests & modeling
├── requirements.txt                   # Project dependencies
├── .gitignore                         # Ignored environments and Jupyter checkpoints
└── README.md                          # Project documentation
```

---

## Methodology

### 1. Data Integrity & Leak-Free Preprocessing
- **Duplicate auditing:** Verification and removal of exact duplicate records.
- **Domain-consistent smoking reconciliation:** Logical reconciliation between `smokingStatus` and `cigsPerDay` (`cigsPerDay > 0` $\Rightarrow$ `smokingStatus = 'yes'`; missing `cigsPerDay` for active smokers imputed via the *smokers'* median of `20` cigarettes/day rather than the zero-inflated cohort median of `0`).
- **Leak-free imputation (`ClinicalPreprocessor`):** Mode imputation for categorical/binary indicators and median imputation for continuous biomarkers, fitted **exclusively on training folds** during cross-validation and model evaluation.

### 2. Exploratory Data Analysis & Clinical Discretization
Continuous biomarkers (several exhibiting strong right-skewness, such as `glucose` with skewness $+6.55$, or zero-inflation, such as `cigsPerDay`) are bucketed into established clinical reference intervals to estimate conditional risk rates $P(\text{CHDRisk}=\text{yes} \mid X)$:

| Variable | Discretized Feature | Clinical Categories | Guideline |
|----------|---------------------|---------------------|-----------|
| `age` | `age_group` | `32–40`, `41–48`, `49–55`, `56+` years | Framingham age quartiles |
| `cigsPerDay` | `cigs_group` | `0 (Non-smoker)`, `1–6`, `7–15`, `16–29`, `30+` cigs/day | Smoking dose-response |
| `totChol` | `chol_group` | `Desirable (<200)`, `Borderline (200–239)`, `High (>=240)` mg/dL | NCEP ATP III |
| `BMI` | `bmi_group` | `Underweight (<18.5)`, `Normal (18.5–24.9)`, `Overweight (25–29.9)`, `Obese (>=30)` kg/m² | WHO BMI classification |
| `glucose` | `glucose_group` | `Low (<70)`, `Normal (70–99)`, `Pre-diabetes (100–125)`, `Diabetes (>=126)` mg/dL | ADA fasting glucose |
| `sysBP`, `diaBP` | `bp_group` | `Normal (<120/<80)`, `Elevated (120–129/<80)`, `Stage 1 HTN (130–139/80–89)`, `Stage 2 HTN (>=140/>=90)` | AHA/ACC staging |

### 3. Inferential Hypothesis Testing & Statistical Feature Selection
- **Continuous predictors (`Welch's t-test`, `Mann–Whitney U`, `Cohen's d`):** Significant differences ($\alpha = 0.05$) confirmed for `age` ($d = 0.656$), `sysBP` ($d = 0.559$), `diaBP` ($d = 0.378$), `glucose` ($d = 0.251$), `totChol` ($d = 0.221$), `BMI` ($d = 0.199$), and `cigsPerDay` ($d = 0.150$). Resting `heartRate` ($p = 0.142$, $d = 0.063$) is statistically indistinguishable across groups and excluded.
- **Categorical predictors (`Pearson's Chi-Square`, `Cramér's V`):** Strongest associations observed for `age_group` ($V = 0.219$), `prevalentHyp` ($V = 0.177$), `bp_group` ($V = 0.174$), and `glucose_group` ($V = 0.123$). Notably, binary `smokingStatus` alone fails to reject independence ($p = 0.221$), whereas dose-response `cigs_group` is significant ($p = 1.32 \times 10^{-3}$).
- **Training-only ranking:** `SelectKBest(score_func=chi2)` fitted strictly on the 80% stratified training set selects the top clinical features without test-set leakage.

### 4. Probabilistic Classification & Class Balancing
- **Stratified split:** 80% training (`3,392` patients) / 20% held-out test (`848` patients), preserving the natural 15.2% CHD prevalence.
- **Class balancing:** `RandomUnderSampler` applied **inside `imblearn.pipeline.Pipeline`** (training folds only) to overcome the majority-class prior bias.
- **Classifiers evaluated:**
  1. `CategoricalNB` (Laplace-smoothed, exact category cardinalities) — Unbalanced vs. Undersampled (5 original bucketed features vs. $\chi^2$ Top-10 clinical features)
  2. `GaussianNB` — Undersampled continuous biomarkers
  3. `LogisticRegression` — Undersampled Framingham linear log-odds benchmark

---

## Summary of Results

Evaluated via 5-fold stratified cross-validation on `X_train` and on the untouched held-out test set (`n = 848`, 15.2% positive prevalence):

| Model | 5-Fold CV ROC-AUC | Test Accuracy | Sensitivity (Recall+) | Specificity (TNR) | Precision (PPV) | F1 (CHD+) | Test ROC-AUC |
|-------|-------------------|---------------|-----------------------|-------------------|-----------------|-----------|--------------|
| `CategoricalNB` — Unbalanced (5 Bucketed) | 0.694 ± 0.024 | 0.847 | 0.039 | 0.992 | 0.455 | 0.071 | 0.658 |
| `CategoricalNB` — Undersampled (5 Bucketed) | 0.691 ± 0.031 | 0.598 | 0.605 | 0.597 | 0.212 | 0.314 | 0.662 |
| **`CategoricalNB` — Undersampled ($\chi^2$ Top-10)** | **0.707 ± 0.030** | **0.671** | **0.605** | **0.683** | **0.255** | **0.359** | **0.687** |
| `GaussianNB` — Undersampled (Continuous) | 0.722 ± 0.027 | 0.750 | 0.364 | 0.819 | 0.266 | 0.307 | 0.666 |
| `Logistic Regression` — Undersampled (Benchmark) | 0.731 ± 0.033 | 0.659 | 0.612 | 0.668 | 0.248 | 0.353 | 0.695 |

Key findings:
- **Resolving the Accuracy Paradox:** Without balancing, `CategoricalNB` reaches 84.7% accuracy by predicting `no` almost universally, missing **96.1%** of positive CHD cases (`Sensitivity = 3.9%`). Undersampling the training folds increases sensitivity more than **15-fold** to **60.5%** while achieving a **90.6% Negative Predictive Value (NPV)**.
- **Clinical Discretization Advantage:** Incorporating $\chi^2$-selected clinical categories (`age_group`, `bp_group`, `prevalentHyp`, `sex`, `glucose_group`, `chol_group`, `cigs_group`, `diabetes`, `BPMeds`, `prevalentStroke`) enables `CategoricalNB` to achieve the **highest positive-class F1-score (`0.359`)** across all evaluated models—surpassing `GaussianNB` in sensitivity (`60.5%` vs. `36.4%`, due to robustness against extreme right-skewness in fasting glucose) and matching the `Logistic Regression` Framingham benchmark (`F1 = 0.353`, `Sensitivity = 61.2%`).

---

## Dataset

The cohort analyzed is derived from the **Framingham Heart Study (10-Year CHD Risk)** dataset:
- **Public Reference:** [Framingham Heart Study Dataset — Kaggle](https://www.kaggle.com/datasets/aasheesh200/framingham-heart-study-dataset)
- **Local Copy:** Included directly in this repository at `data/Heart_Disease_Dataset.csv` (also compatible with Google Colab at `/content/sample_data/Heart_Disease_Dataset.csv`).

---

## Getting Started

### Prerequisites
- Python 3.10+
- Jupyter Notebook, JupyterLab, or Google Colab

### Install dependencies

```bash
pip install -r requirements.txt
```

### Run the notebook

```bash
jupyter notebook heart-disease-risk-analysis.ipynb
```

---

## Tech Stack

- **Python 3**
- **Pandas & NumPy** — data manipulation and numerical computing
- **SciPy** — parametric and non-parametric statistical hypothesis testing
- **Matplotlib & Seaborn** — statistical visualization
- **Scikit-learn** — preprocessing pipelines, $\chi^2$ feature selection, modeling, and evaluation
- **Imbalanced-learn** — leak-free training-fold class balancing (`RandomUnderSampler`)

---

## License

Academic project — for educational use only. CIn/UFPE.

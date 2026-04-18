# Heart Disease Risk Pattern Analysis

> Academic project developed for the **Statistics and Probability for Computer Science** course — CIn/UFPE

---

## Overview

This project performs a statistical analysis and builds a predictive classifier to identify **coronary heart disease (CHD) risk patterns** from clinical and behavioral patient data.

The dataset (`Heart_Disease_Dataset.csv`) contains variables such as age, BMI, cholesterol levels, glucose, smoking habits, and hypertension history. The target variable is `CHDRisk` (coronary heart disease risk: yes/no).

---

## Team

| Name | Email |
|------|-------|
| Artur Fernandes | avpf@cin.ufpe.br |
| Breno Ramos | brsg@cin.ufpe.br |
| Caio Vilas Boas | ccnvb@cin.ufpe.br |
| João Nascimento | jvsn2@cin.ufpe.br |

---

## Repository Structure

```
.
├── ESTATISTICA.ipynb       # Main notebook with the full analysis
├── requirements.txt        # Project dependencies
└── README.md               # This file
```

> The dataset `Heart_Disease_Dataset.csv` is not included in this repository. See the [Dataset](#dataset) section below.

---

## Methodology

### 1. Preprocessing
- Removal of duplicate entries
- Imputation of null values (`sex` and `smokingStatus`) using the mode
- Data integrity verification

### 2. Exploratory Data Analysis (EDA)
Numerical variables were bucketed into clinically meaningful ranges:

| Variable | Categories |
|----------|-----------|
| `age` | 32–40, 41–48, 49–55, 55+ years |
| `cigsPerDay` | 1–6, 7–15, 15–30, 30+ cigarettes/day |
| `totChol` | Ideal (<200), High (200–239), Dangerous (>=240) |
| `BMI` | Underweight, Normal, Overweight, Obese |
| `glucose` | Low, Ideal, Dangerous, Pre-diabetes |

Distributions of `prevalentHyp` (hypertension) and `sex` in relation to CHD risk were also analyzed.

### 3. Feature Selection
Features selected for the model:
```
age, cigsPerDay, totChol, BMI, glucose
```

### 4. Naive Bayes Classifier
- Training data balanced via **RandomUnderSampler** (original dataset: ~85% negative / ~15% positive)
- Train/test split: **80% / 20%** with stratification
- Model: `CategoricalNB` (scikit-learn)
- Evaluation: accuracy score, classification report, and confusion matrix

---

## Results

The model achieved better performance for **true negatives**, which reflects the natural class imbalance of the condition in the population. The confusion matrix highlights the hit rates and false positive/negative cases.

---

## Dataset

The dataset used is the **Heart Disease Dataset**, publicly available on Kaggle:

[Heart Disease Dataset — Kaggle](https://www.kaggle.com/datasets/ritwikb3/heart-disease-cleveland)

After downloading, place the file at:
```
/content/sample_data/Heart_Disease_Dataset.csv
```
*(or adjust the path in the notebook to match your environment)*

---

## Getting Started

### Prerequisites
- Python 3.8+
- Jupyter Notebook or Google Colab

### Install dependencies

```bash
pip install -r requirements.txt
```

### Run the notebook

```bash
jupyter notebook ESTATISTICA.ipynb
```

Or upload the `.ipynb` file directly to **Google Colab**.

---

## Tech Stack

- **Python 3**
- **Pandas** — data manipulation
- **NumPy** — numerical operations
- **Matplotlib / Seaborn** — data visualization
- **Scikit-learn** — modeling and evaluation
- **Imbalanced-learn** — class balancing

---

## License

Academic project — for educational use only. CIn/UFPE, 2024.

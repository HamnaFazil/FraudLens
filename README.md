# FraudLens

**Credit card fraud detection built the way a bank would use it:** every transaction gets a risk score, the riskiest ones go to a human analyst within a daily review limit, and every alert comes with a reason.

> **Status:** Day 2 of 10 complete (setup, exploratory analysis, time-based split, baselines). Work in progress.

---

## Problem

A bank analyst can review only a limited number of alerts per day (N). A missed fraud loses the transaction amount, and every false alert costs analyst time and annoys a genuine customer.

**Goal:** maximize the money saved within the daily alert budget, and explain every alert.

## Why accuracy is not used

Fraud is extremely rare (about 0.17% of transactions). A model that predicts "never fraud" scores 99.87% accuracy on the test set while catching nothing, so accuracy is meaningless here.

## Success metrics

- **PR-AUC** (area under the precision-recall curve)
- **Precision and recall at k** (reviewing only the top-k highest-risk transactions)
- **Net savings:** fraud money caught minus the cost of reviewing alerts

## Data

- **Source:** [Credit Card Fraud Detection (Kaggle, ULB Machine Learning Group)](https://www.kaggle.com/datasets/mlg-ulb/creditcardfraud)
- **Size:** 284,807 transactions, of which 492 are fraud (0.1727%)
- **Period:** two days in September 2013, European cardholders
- **Features:** `V1`-`V28` (anonymized by PCA), `Time`, `Amount`, and the label `Class` (1 = fraud)

**Limitations**
- The features are anonymized, so explanations will be in terms of components rather than plain business terms such as merchant type.
- The data covers only two days, so long-term drift cannot be measured directly.

**Citation:** Dal Pozzolo, Caelen, Johnson and Bontempi. *Calibrating Probability with Undersampling for Unbalanced Classification.* IEEE CIDM, 2015.

The dataset is not stored in this repo because of its size. Download `creditcard.csv` from Kaggle and place it in `data/`.

## Planned approach

| Step | What | Why it matters |
|---|---|---|
| 1 | Time-based train/test split | Mimics real deployment and avoids leaking future information |
| 2 | Baselines (simple rule, logistic regression) | A benchmark every later model must beat |
| 3 | Unsupervised (Isolation Forest, autoencoder) vs supervised (LightGBM) | Shows which fraud each approach catches |
| 4 | Cost-aware thresholds under an alert budget | Decisions based on money, not accuracy |
| 5 | SHAP explanations | A reason behind every alert |
| 6 | Adversarial test | A simulated fraudster adapts; measure the drop and the recovery |
| 7 | Drift check | Does the model hold up as behavior changes? |
| 8 | Streamlit app | A live demo with an alert-budget slider |

## Day 1 findings (exploratory analysis)

- **Class imbalance:** 492 frauds out of 284,807 transactions (0.1727%), about 578 normal transactions per fraud.
- **Data quality:** 0 missing values, 31 columns, and rows already sorted by `Time`, so a time-based split is straightforward.
- **Transaction amount:** fraud is not concentrated among the largest transactions (confirmed by the Amount-rule baseline below). `<add one sentence on what the boxplot showed>`
- **Time of day:** `<one sentence on the fraud-rate-by-hour chart>`
- **Most informative features:** `<the top features from your correlation chart>`

Charts are in [`reports/`](reports/):

![Class balance](reports/class_balance.png)
![Amount by class](reports/amount_by_class.png)
![Fraud rate by hour](reports/fraud_rate_by_hour.png)
![Top features](reports/top_features.png)

## Baseline results (time-based split, 75% train / 25% test)

The data is split by time: the first 75% of transactions train the model and the last 25% test it, which mimics real deployment. Test set: 71,202 transactions, 94 frauds (0.132%). Training set: 398 frauds (0.186%). Review budget: 200 alerts at an assumed $5 per review.

| Model | PR-AUC | Precision@200 | Recall@200 | Net savings@200 |
|---|---|---|---|---|
| Rule: high Amount | 0.0015 | 0.000 | 0.000 | -$1,000 |
| Logistic Regression | 0.7614 | 0.385 | 0.819 | $7,448 |

- A model that always predicts "not fraud" scores 99.87% accuracy while catching nothing, so accuracy is not used.
- Large transactions are not where the fraud is: the Amount rule performs at random level (PR-AUC 0.0015 against a test fraud rate of about 0.0013).
- Logistic regression is a strong baseline. In its top 200 alerts, 77 were real fraud and 123 were false alarms, and it caught 77 of the 94 test frauds. Later models must beat it to justify their complexity.
- **Caveats:** the test set has only 94 frauds, so small differences between models should be read with care. The 200-alert budget and $5 review cost are assumptions that will be varied later.

![Precision-recall baselines](reports/pr_baselines.png)

## Project structure

```
FraudLens/
├── data/           # creditcard.csv goes here (not tracked by git)
├── notebooks/      # 01_eda.ipynb, 02_baselines.ipynb, ...
├── src/            # reusable code (metrics.py, ...)
├── app/            # Streamlit app
├── reports/        # saved charts and figures
├── PROJECT_LOG.md  # daily notes
├── requirements.txt
└── README.md
```

## How to run (Windows PowerShell)

```powershell
git clone https://github.com/<your-username>/FraudLens.git
cd FraudLens
python -m venv .venv
.venv\Scripts\Activate.ps1
pip install -r requirements.txt
```

Download `creditcard.csv` into `data\`, then open the notebooks in VS Code and run the cells in order (`01_eda.ipynb`, then `02_baselines.ipynb`).

## Progress

- [x] Day 1: project setup, data loading, exploratory analysis
- [x] Day 2: time-based split, metrics, baselines
- [ ] Day 3: Isolation Forest, autoencoder, LightGBM
- [ ] Day 4: hybrid model
- [ ] Day 5: cost-aware thresholds and alert budget
- [ ] Day 6: SHAP explanations
- [ ] Day 7: adversarial test
- [ ] Day 8: drift check
- [ ] Day 9: Streamlit app
- [ ] Day 10: final write-up and demo

Daily notes are in [PROJECT_LOG.md](PROJECT_LOG.md).

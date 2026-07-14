# Credit Card Default Risk: PD Modeling, Probability Calibration and Expected Loss

## Project Overview

This project develops an end-to-end credit risk modeling workflow to predict whether a credit card customer will default in the following month.

The project combines:

- exploratory data analysis;
- feature engineering;
- machine learning model comparison;
- Probability of Default calibration;
- classification-threshold analysis;
- borrower-level Expected Loss estimation;
- data-driven risk segmentation;
- portfolio loss concentration analysis.

The objective is not only to classify default risk, but also to convert model outputs into interpretable borrower-level and portfolio-level credit risk measures.

---

## Dataset

The project uses the **Default of Credit Card Clients** dataset, containing 30,000 credit card customers in Taiwan.

The dataset includes:

- approved credit limits;
- demographic characteristics;
- repayment status;
- monthly bill amounts;
- monthly payment amounts;
- next-month default status.

The target variable is:

```text
default = 1: customer defaults in the following month
default = 0: customer does not default
```

All monetary values are measured in New Taiwan Dollars.

---

## Project Workflow

1. Data understanding and cleaning
2. Exploratory data analysis
3. Feature engineering
4. Multicollinearity assessment
5. Logistic Regression baseline
6. Model comparison
7. Probability calibration
8. Model explainability
9. Threshold analysis
10. Expected Loss estimation
11. PD risk segmentation
12. Expected Loss concentration analysis

---

## Feature Engineering

Behavioral features were created to summarize customer repayment and balance patterns:

| Feature | Description |
|---|---|
| `avg_pay_status` | Average repayment status across recent months |
| `avg_bill_amt` | Average monthly bill amount |
| `avg_pay_amt` | Average monthly payment amount |
| `credit_utilization` | Average bill amount divided by credit limit |
| `payment_to_bill_ratio` | Average payment amount divided by average bill amount |

Recent repayment status was retained because short-horizon default risk is strongly related to recent payment behavior.

---

## Model Comparison

The following models were evaluated:

- Logistic Regression
- Random Forest
- XGBoost
- LightGBM

| Model | ROC-AUC | Gini | Default Recall | Default F1 |
|---|---:|---:|---:|---:|
| Random Forest | 0.7776 | 0.5551 | 0.6148 | 0.5350 |
| XGBoost | 0.7765 | 0.5531 | 0.6258 | 0.5340 |
| LightGBM | 0.7765 | 0.5531 | 0.6328 | 0.5348 |
| Balanced Logistic Regression | 0.7153 | 0.4307 | 0.6354 | 0.4630 |

Random Forest was selected as the final model because it achieved the highest ROC-AUC and Gini, although its performance was close to XGBoost and LightGBM.

---

## Probability Calibration

Class-imbalance adjustment improved default detection but caused the raw model probabilities to overestimate portfolio default risk.

Platt Scaling was applied using a separate calibration set.

| Metric | Before Calibration | After Calibration |
|---|---:|---:|
| Average predicted PD | 42.31% | 22.08% |
| Observed default rate | 22.12% | 22.12% |
| Brier score | 0.1788 | 0.1365 |
| Log loss | 0.5428 | 0.4335 |

Calibration improved the reliability of predicted PD while leaving ROC-AUC unchanged.

---

## Threshold Analysis

Multiple classification thresholds were compared using precision, recall, F1-score, false positives, and false negatives.

Among the tested thresholds, **0.30** produced the highest F1-score:

| Threshold | Precision | Recall | F1-score |
|---:|---:|---:|---:|
| 0.50 | 0.6216 | 0.3877 | 0.4776 |
| 0.40 | 0.5803 | 0.4666 | 0.5173 |
| 0.30 | 0.5297 | 0.5560 | 0.5425 |
| 0.20 | 0.4331 | 0.6670 | 0.5252 |

The selected threshold should be interpreted as a statistical operating point rather than a universal business threshold.

---

## Expected Loss Framework

Borrower-level Expected Loss is calculated as:

```text
Expected Loss = PD × LGD × EAD
```

In this project:

- **PD** is obtained from the calibrated Random Forest model;
- **LGD** is assumed to be 45%;
- **EAD** is proxied by `BILL_AMT1`, the most recent bill-statement balance.

Because the dataset does not contain recovery, collateral, collection-cost, or workout information, LGD cannot be estimated directly.

EAD sensitivity is evaluated using illustrative Credit Conversion Factor scenarios:

```text
Scenario EAD = Drawn EAD + CCF × Undrawn Amount
```

The CCF scenarios show how Expected Loss changes if customers draw additional unused credit before default.

---

## Risk Segmentation and Loss Concentration

PD risk bands are estimated from the calibration sample using a shallow decision tree rather than fixed arbitrary cutoffs.

The resulting boundaries are applied unchanged to the test portfolio to create:

- Low Risk
- Medium Risk
- High Risk

Borrowers are also ranked by Expected Loss to measure how much of total portfolio loss is concentrated among the largest individual risk contributors.

---

## Key Findings

- Recent repayment behavior is among the strongest predictors of next-month default.
- Tree-based models outperform the Logistic Regression benchmark in ranking default risk.
- Class weighting improves default detection but can distort raw probability estimates.
- Platt Scaling substantially improves PD calibration.
- Probability of Default and Expected Loss capture different dimensions of credit risk.
- Customers with moderate PD may still create high Expected Loss when exposure is large.
- Portfolio loss analysis should consider both borrower risk and exposure concentration.

---

## Limitations

- The target represents next-month default rather than 12-month or lifetime PD.
- The dataset is historical and comes from one credit card portfolio.
- LGD is assumed rather than estimated from recovery data.
- `BILL_AMT1` is only a proxy for drawn EAD.
- CCF scenarios are illustrative rather than empirically estimated.
- Expected Loss cannot be backtested because realized loss data are unavailable.
- The model has not been validated on an external or later-period dataset.

This project should therefore be interpreted as an educational credit risk analytics implementation rather than a production or regulatory model.

---

## Future Work

Future improvements may include:

- temporal and external validation;
- 12-month and lifetime PD modeling;
- portfolio-specific LGD and CCF estimation;
- macroeconomic scenario analysis;
- model and calibration drift monitoring;
- an interpretable Weight of Evidence scorecard;
- automated scoring and portfolio-monitoring dashboards.

---

## Repository Structure

```text
credit-risk-pd-model/
├── README.md
├── requirements.txt
├── .gitignore
├── data/
│   └── default_credit_card_clients.xls
└── notebooks/
    └── 01_credit_risk_pd_model.ipynb
```

---

## How to Run

Clone the repository:

```bash
git clone https://github.com/meidayln/credit-risk-pd-model.git
cd credit-risk-pd-model
```

Install the required packages:

```bash
pip install -r requirements.txt
```

Open:

```text
notebooks/01_credit_risk_pd_model.ipynb
```

Then run all notebook cells from top to bottom.

---

## Tools

- Python
- pandas
- NumPy
- Matplotlib
- Seaborn
- scikit-learn
- statsmodels
- XGBoost
- LightGBM
- SHAP
- Jupyter Notebook

---

## Author

**Le Nguyen Minh Duy**

Portfolio project focused on credit risk modeling, probability calibration, and Expected Loss analysis.

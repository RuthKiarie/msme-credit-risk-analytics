# MSME Loan Portfolio Risk & Performance Dashboard

A self-directed project simulating the day-to-day analytics work of a Junior
Data Scientist at an embedded-finance / MSME-lending company: turning raw loan and repayment data into
portfolio risk metrics, a credit scorecard prototype and a stakeholder-ready
dashboard.

## Why this project

This mirrors the five core responsibility areas of a portfolio/credit
analytics role:

1. **Data Analysis & Reporting** - SQL extraction, cleaning, joins, EDA
2. **Credit & Portfolio Analytics** - PAR30/PAR90, DPD, repayment rate,
   disbursement/collections trends, segmentation
3. **Insights & Reporting** - Power-BI-style dashboard + one-page stakeholder
   report
4. **Model Experimentation Support** - a credit scorecard prototype
   (logistic regression) with honest, documented evaluation
5. **Documentation & Best Practices** - reusable SQL templates, a full data
   dictionary, and a reproducible pipeline


## Tech_Stack: 
Python, Pandas, OpenPyXL, python-docx, Excel.

## Data

The dataset is **synthetic** (no real borrower data), generated with a
reproducible random seed to have realistic risk structure; default/late
repayment probability is correlated with sector, region, bureau score and
business age, so the resulting PAR/segmentation numbers behave the way a
real MSME lending portfolio's would.

| Table | Rows | Description |
|---|---|---|
| `borrowers.csv` | 850 | One row per MSME borrower |
| `loans.csv` | 1,048 | One row per loan (some borrowers have multiple loans) |
| `repayments.csv` | 8,004 | One row per scheduled installment |

Full column-level documentation is in the **Data Dictionary** sheet of the
dashboard workbook.

## Pipeline

```
build.py (+ generate_data)     → builds the synthetic borrowers/loans/repayments CSVs
                               → loads CSVs into a SQLite database (company_any_portfolio.db)
01_loan_risk_status            → per-loan DPD & outstanding balance view
02_par_metrics                 → PAR30/PAR90, overall + by sector + by region
03_repayment_and_trends        → repayment rate, monthly disbursement/collections
04_borrower_segmentation       → sector x loan-size segmentation
run_analysis                   → runs the SQL layer, exports result CSVs
build_scorecard                → trains + evaluates the credit scorecard (scikit-learn)
build_dashboard_stage*.py      → builds the formula-driven Excel dashboard
```

To reproduce end to end:
```bash
python3 build.py
python3 generate_data
python3 run_analysis
python3 build_scorecard
python3 build_dashboard_stage1.py && python3 build_dashboard_stage2.py \
  && python3 build_dashboard_stage3.py && python3 build_dashboard_stage4.py
```

## Key definitions

- **PAR30 / PAR90 (Portfolio at Risk)**: outstanding balance of loans with
  at least one installment currently overdue by 30+ / 90+ days, divided by
  total outstanding portfolio balance.
- **DPD (Days Past Due)**: at the loan level, the worst (max) days overdue
  among that loan's currently-unpaid installments as of the snapshot date.
- **Repayment rate**: total amount collected to date / total amount that
  has fallen due to date (collection efficiency).
- **Outstanding balance**: sum of all installment amounts not yet paid
  (overdue unpaid + not-yet-due future installments).

## Results snapshot (as of 2026-09-01)

| Metric | Value |
|---|---|
| Total outstanding portfolio | KES 14.85M |
| PAR30 | 4.2% |
| PAR90 | 1.8% |
| Repayment rate | 91.5% |
| Total disbursed to date | KES 45M |
| Active borrowers | 1240 |

Highest-risk sector: **Manufacturing** (PAR30 21.3%). Highest-risk region:
**Rural - Other** (PAR30 19.9%).

## Credit scorecard prototype

A logistic regression trained only on features known **at origination**
(no repayment-behaviour leakage) to predict whether a loan becomes 90+ days
past due:

- AUC-ROC: **0.610**
- Precision (bad-loan class): 15.4% · Recall: 40.0%
- Top risk drivers: `bureau_score` (protective), `sector_Manufacturing`
  (higher risk), `loan_term_months` (higher risk)

This is presented honestly as a **first-pass baseline for review and
iteration** — in line with a "model experimentation support" scope, not a
production-ready model.



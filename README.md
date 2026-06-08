# 🏦 FinSight — Credit Risk Analytics & NPA Prediction

> **End-Term Examination Assignment**  
> Post Graduate Certificate Programme in Big Data Analytics (PGCP-BDA)  
> Centre for Development of Advanced Computing (C-DAC), Mumbai — 2025

---

## 📋 Table of Contents

- [Project Overview](#-project-overview)
- [Business Context](#-business-context)
- [Dataset Description](#-dataset-description)
- [Project Structure](#-project-structure)
- [Question-wise Coverage](#-question-wise-coverage)
- [Key Findings](#-key-findings)
- [Tech Stack](#-tech-stack)
- [Getting Started](#-getting-started)
- [Results Summary](#-results-summary)
- [Business Recommendations](#-business-recommendations)
- [Regulatory Context](#-regulatory-context)
- [Author](#-author)

---

## 🔍 Project Overview

**FinSight** is a comprehensive, end-to-end credit risk analytics pipeline built on a synthetic Indian retail banking dataset of **2,000,000 loan accounts** across **9 relational CSV files**. The project simulates the full workflow of a data scientist embedded in a bank's Credit Risk division — from raw data ingestion and quality remediation, through exploratory profiling, feature engineering, statistical modelling, and finally, data-driven business recommendations.

The two primary modelling targets are:

| Target Variable | Definition | Regulatory Relevance |
|---|---|---|
| `npa_flag` | Binary: 1 = Non-Performing Asset (default), 0 = Performing | Probability of Default (PD) — IFRS 9 / IndAS 109 |
| `lgd_pct` | Loss Given Default as % of outstanding exposure | LGD component of ECL model under RBI Prudential Norms |

---

## 🏛️ Business Context

Non-Performing Assets (NPAs) remain one of the most critical challenges facing Indian banks. Under the **RBI's Prudential Norms** and the **IFRS 9 / IndAS 109 Expected Credit Loss (ECL)** framework, banks must quantify:

- **PD** — Probability of Default
- **LGD** — Loss Given Default
- **EAD** — Exposure at Default

This project operationalises PD and LGD estimation using classical machine learning and econometric techniques applied to retail loan data resembling the portfolios of Indian commercial banks.

---

## 📂 Dataset Description

The dataset consists of nine CSV files, each representing a distinct operational data domain. All tables are linked via the `loan_id` primary key.

| # | File Name | Description | Key Fields |
|---|---|---|---|
| 1 | `loans_master.csv` | Master loan registry — base table | `loan_id`, `loan_amnt`, `grade`, `issue_date`, `loan_status` |
| 2 | `customer_bureau.csv` | Credit bureau attributes at origination | `cibil_score`, `credit_hist_years`, `delinq_2yrs`, `pub_rec` |
| 3 | `loan_performance.csv` | Performance tracking & NPA status | `npa_flag`, `lgd_pct`, `recoveries_inr`, `loan_approved_date` |
| 4 | `monthly_emi_track.csv` | Monthly EMI payment trail | `emi_to_income_ratio`, `payment_status`, `outstanding_amnt` |
| 5 | `payment_history.csv` | Historical payment behaviour | `payment_date`, `amount_paid`, `delinquency_months` |
| 6 | `collateral_assets.csv` | Collateral details for secured loans | `collateral_value`, `collateral_type`, `valuation_agency` |
| 7 | `credit_card_behavior.csv` | Revolving credit usage patterns | `revol_util_pct`, `bc_util`, `all_util`, `bc_open_to_buy` |
| 8 | `loan_enquiry_bureau.csv` | Credit bureau enquiry history | `inq_last_6mths`, `enq_velocity_score` |
| 9 | `branch_region_economy.csv` | Branch and regional macro data | `region`, `branch_id`, `gdp_growth_rate`, `unemployment_rate` |

> **Total Records:** 2,000,000 loans  
> **Master Table:** `loans_master.csv`  
> **Join Key:** `loan_id` (left merge, master-anchored)

---

## 📁 Project Structure

```
FinSight-Credit-Risk-Analytics/
│
├── PtFinsightAssignment.ipynb        # Main Jupyter Notebook (full pipeline)
│
├── data/                              # Source CSV files (not tracked in Git — see .gitignore)
│   ├── loans_master.csv
│   ├── customer_bureau.csv
│   ├── loan_performance.csv
│   ├── monthly_emi_track.csv
│   ├── payment_history.csv
│   ├── collateral_assets.csv
│   ├── credit_card_behavior.csv
│   ├── loan_enquiry_bureau.csv
│   └── branch_region_economy.csv
│
├── outputs/                           # Generated artefacts
│   ├── FinSight_Final_Merged.parquet  # Cleaned, merged analytical dataset
│   ├── Q2a_CIBIL_Distribution.png
│   ├── Q2b_NPA_by_Grade.png
│   ├── Q3_Correlation_Heatmap.png
│   ├── Q4a_VIF_Table.png
│   ├── Q4b_OLS_Coefficients.png
│   ├── Q4c_Regularised_Models.png
│   └── Q4d_Diagnostic_Plots.png
│
├── FinSight_Executive_Summary.docx    # 8–10 page executive summary
├── README.md                          # This file
└── .gitignore
```

---

## 📊 Question-wise Coverage

### Q1 — Data Acquisition, Joining & Cleaning

**1(a) Memory-Optimised Ingestion**
- Loaded all 9 CSV files using `pd.read_csv()` with `chunksize=100,000`
- Applied numeric downcasting (float64→float32, int64→int8/int16) to reduce RAM footprint
- Saved each dataset as Apache Parquet for efficient downstream I/O
- Reported memory usage before and after downcasting and individual Parquet file sizes

**1(b) Sequential Multi-Table Join**
- Anchored on `loans_master.csv` (base: 2,000,000 rows)
- Left-merged all 8 child tables sequentially on `loan_id`
- Asserted row count = 2,000,000 after every individual merge
- Identified and quantified orphan records per child table

**1(c) Data Quality — Eight Injected Issues**

| # | Column | Issue | Fix |
|---|---|---|---|
| 1 | `annual_inc_inr` | Missing (NaN) values | Median imputation |
| 2 | `lgd_pct` | Stored as % instead of decimal | Divide by 100 |
| 3 | `ltv_ratio_pct` | Impossible values (<0 or >200) | Winsorise / median |
| 4 | `int_rate_pct` | ≤5% for non-Grade-A loans | Grade-wise median |
| 5 | `dti_pct` | DTI > 50% (extreme outlier) | Replace with median |
| 6 | `emi_to_income_ratio` | EMI > monthly income (>1.0) | Cap at 1.0 / median |
| 7 | `loan_approved_date` | Approval date after issue date | Date ordering correction |
| 8 | `recoveries_inr` | Recoveries on non-NPA loans | Set to 0 |

---

### Q2 — Exploratory Data Analysis (EDA)

15+ visualisations covering:

- CIBIL score distribution (KDE + boxplot, defaulted vs performing)
- EMI-to-income ratio by NPA status
- Loan grade distribution and grade-level default rates (grouped bar chart)
- Loan purpose default rate ranking
- LGD distribution (histogram + CDF)
- Default rate by loan tenure (36 vs 60 months)
- Time-series default rate trend (2015–2022, COVID spike highlighted)
- Correlation heatmap — all numeric features vs `npa_flag` and `lgd_pct`
- CIBIL score vs LGD scatter plot
- Geographic/regional NPA concentration

---

### Q3 — Feature Engineering

Ten domain-informed engineered features:

| Feature | Formula | Rationale |
|---|---|---|
| `emi_to_income_ratio` | Monthly EMI / Monthly Income | Debt service burden; primary default trigger |
| `loan_to_income_ratio` | Loan Amount / Annual Income | Leverage at origination |
| `real_interest_rate` | `int_rate_pct` − Inflation Rate | Actual purchasing-power-adjusted borrowing cost |
| `rate_spread_pct` | `int_rate_pct` − RBI Repo Rate | Bank's risk premium; top LGD predictor (r = +0.0794) |
| `collateral_coverage_ratio` | Collateral Value / Outstanding Loan | Direct LGD suppressor |
| `delinq_severity_score` | f(delinquency months, delinq_2yrs) | Composite payment failure index |
| `enq_velocity_score` | Bureau enquiries in last 6 months | Credit-seeking desperation signal |
| `income_stability_ratio` | Employment Length / Loan Term | Income predictability over loan lifetime |
| `credit_depth_score` | Total accounts / Credit history years | Credit sophistication metric |
| `covid_issue_year_flag` | Binary: 1 if issue year = 2020 | Pandemic-era origination risk indicator |

**VIF Analysis** — Features with VIF > 10 removed to prevent multicollinearity:
- `int_rate_pct` — REMOVED (VIF = INF; collinear with rate_spread_pct)
- `credit_util_composite` — REMOVED (near-linear combination of util features)
- `dti_pct` — REMOVED (overlaps with emi_to_income_ratio and loan_to_income_ratio)

---

### Q4 — Statistical & Regression Modelling

**4(a) VIF Analysis and Feature Selection**
- Computed VIF for all candidate features
- Removed 4 features with VIF > 10; retained clean feature set

**4(b) Baseline OLS Model (statsmodels)**
- Fitted OLS on VIF-filtered features
- Reported: R², Adjusted R², F-statistic, Durbin-Watson, Jarque-Bera, Condition Number
- Interpreted top-5 coefficients with business translation
- Produced OLS coefficient plot with 95% confidence intervals

**4(c) Regularised Models — Ridge, Lasso, ElasticNet**
- All models trained via `sklearn.Pipeline` (StandardScaler + model)
- 5-fold cross-validated `GridSearchCV` over alpha grid (log-space: 0.0001–100)
- ElasticNet additionally searched over l1_ratio = [0.1, 0.3, 0.5, 0.7, 0.9]
- Reported: Best alpha, CV RMSE, Test RMSE, Test R²
- Lasso zero-coefficient features identified and explained

**4(d) 4-Panel Regression Diagnostic Plot**
- Panel (i): Residuals vs Fitted — linearity + homoscedasticity
- Panel (ii): Normal Q-Q Plot — normality of residuals
- Panel (iii): Scale-Location — homoscedasticity confirmation
- Panel (iv): Cook's Distance — influential observation detection

---

### Q5 — Business Recommendations

Five independently evidenced, quantified credit risk recommendations (see [Business Recommendations](#-business-recommendations) section below).

---

## 🔑 Key Findings

| Finding | Evidence | Impact |
|---|---|---|
| CIBIL < 600 cohort drives disproportionate NPAs | KDE separation, Cohen's d effect size | Tightening origination floor reduces NPA formation by 15–20% |
| EMI-to-income ratio predicts both PD and LGD | OLS positive coefficient (p < 0.05) | FOIR cap at 40% prevents over-leveraged origination |
| Grade system is monotonically correct | A → G default rate increases consistently | Risk-adjusted pricing by sub-grade justified |
| COVID-2020 vintage carries higher LGD | t = 9.98, p = 1.88×10⁻²³ | Dedicated recovery team + ECL model adjustment needed |
| Collateral suppresses LGD more than CIBIL | CIBIL vs LGD R² ≈ 0; collateral ratio significant | Collateral-first underwriting for high-risk purposes |
| rate_spread_pct is top LGD predictor | Highest correlation (r = +0.0794) | Bank's own risk pricing is the best LGD signal |

---

## 🛠️ Tech Stack

```python
# Core
pandas >= 2.0
numpy >= 1.24

# Statistical Modelling
statsmodels >= 0.14
scipy >= 1.11

# Machine Learning
scikit-learn >= 1.3

# Visualisation
matplotlib >= 3.7

# Storage
pyarrow >= 14.0     # Parquet I/O

# Standard Library
os, pathlib, warnings
```

---

## 🚀 Getting Started

### 1. Clone the Repository

```bash
git clone https://github.com/<your-username>/finsight-credit-risk-analytics.git
cd finsight-credit-risk-analytics
```

### 2. Install Dependencies

```bash
pip install pandas numpy statsmodels scipy scikit-learn matplotlib pyarrow
```

Or with conda:

```bash
conda create -n finsight python=3.11
conda activate finsight
conda install pandas numpy statsmodels scipy scikit-learn matplotlib pyarrow
```

### 3. Add the Dataset

Place all nine CSV files in the `data/` directory (files are not tracked in Git due to size):

```
data/
├── loans_master.csv
├── customer_bureau.csv
├── loan_performance.csv
├── monthly_emi_track.csv
├── payment_history.csv
├── collateral_assets.csv
├── credit_card_behavior.csv
├── loan_enquiry_bureau.csv
└── branch_region_economy.csv
```

### 4. Run the Notebook

```bash
jupyter notebook PtFinsightAssignment.ipynb
```

> **Note:** Run cells sequentially from top to bottom. Q1 generates the merged Parquet file that subsequent questions depend on. Q4b–Q4d use synthetic data for demonstration — replace the data loading block with actual Parquet loading for production runs.

---

## 📈 Results Summary

### Model Comparison — LGD Prediction

| Model | Regularisation | Best Alpha | CV RMSE | Test R² |
|---|---|---|---|---|
| OLS (Baseline) | None | — | — | ~0.25–0.35 |
| Ridge | L2 | Cross-validated | Low | Comparable to OLS |
| **Lasso** | **L1** | **Cross-validated** | **Lowest** | **Best / Comparable** |
| ElasticNet | L1 + L2 | Cross-validated | Low | Comparable to Lasso |

### OLS Diagnostic Summary

| Test | Statistic | Conclusion |
|---|---|---|
| Durbin-Watson | ~2.0 | No significant autocorrelation ✅ |
| Jarque-Bera | p < 0.05 | Normality violated — expected for bounded LGD ⚠️ |
| Condition Number | < 30 | Multicollinearity not severe after VIF filtering ✅ |
| Residuals vs Fitted | Mild pattern | Mild heteroscedasticity — robust SE recommended ⚠️ |

---

## 💡 Business Recommendations

### Recommendation 1 — Tighten CIBIL Score Floor
**Action:** Set hard CIBIL minimum of 650 for unsecured loans.  
**Impact:** 15–20% NPA reduction → ₹10–18 Crore annual provision savings on ₹5,000 Crore book.

### Recommendation 2 — FOIR Cap at 40%
**Action:** Hard reject if EMI-to-Income > 50%; senior approval required for 40–50%.  
**Impact:** ₹15–30 Crore provision release on the high-FOIR segment.

### Recommendation 3 — Grade-Based Risk-Adjusted Pricing
**Action:** Minimum 50 bps step between sub-grades; mandatory collateral for Grade F/G; cap Grade G at 5% of book.  
**Impact:** 175 bps increase on ₹500 Crore E/F/G book → ₹8.75 Crore incremental NII.

### Recommendation 4 — COVID-Vintage NPA Recovery
**Action:** Dedicated recovery team with up to 30% OTS haircut; LGD add-on in ECL models.  
**Impact:** 10–20% faster resolution of 2020-vintage NPAs.

### Recommendation 5 — Collateral-First Underwriting
**Action:** Mandatory collateral for high-risk loan purposes; auto-decline if coverage ratio < 0.8; monthly LTV revaluation.  
**Impact:** Near-zero LGD for collateralised segment → ₹20–40 Crore annual saving on ₹1,000 Crore high-risk book.

---

## ⚖️ Regulatory Context

This project is framed within the following Indian banking regulatory frameworks:

| Framework | Relevance |
|---|---|
| **RBI Prudential Norms on Income Recognition, Asset Classification and Provisioning (IRACP)** | NPA classification criteria; provisioning requirements |
| **IFRS 9 / IndAS 109 Expected Credit Loss (ECL)** | PD × LGD × EAD framework for provision computation |
| **SARFAESI Act, 2002** | Collateral liquidation rights — basis for LGD recovery assumptions |
| **RBI Consumer Credit Guidelines** | FOIR norms referenced in Recommendation 2 |
| **Basel III Capital Adequacy Framework** | Internal Ratings-Based (IRB) approach for credit risk capital |

---

## 👤 Author

**PGCP-BDA Candidate**  
Centre for Development of Advanced Computing (C-DAC), Mumbai  
Post Graduate Certificate Programme in Big Data Analytics  
Batch: 2024–2025

---

## 📄 Licence

This project is submitted as an academic assignment for the PGCP-BDA programme at C-DAC Mumbai. All dataset files are synthetic and generated for educational purposes only. No real customer data is used or represented.

---

*FinSight — Illuminating Credit Risk, One Loan at a Time.*

# Client Credit Risk & Loan Portfolio Analysis 
| PySpark, Power BI, Python
---

##### This end-to-end banking analytics project delivers a comprehensive assessment of client credit risk, deposit behavior, and loan exposure across 3,000 clients spanning 1995–2021. By engineering income and risk classification features in PySpark, the platform enables financial stakeholders to identify high-risk client profiles, evaluate loan concentration by banking relationship type, and develop data-driven strategies for credit policy optimisation and client retention.

---

![Banking Clients Overview](https://github.com/aleksandra20050404/Banking-Clients-Loan-Approval/blob/main/img/banking_clients_overview.jpg)

## Author
Aleksandra Vislova

[![LinkedIn](https://img.shields.io/badge/-LinkedIn-0072b1?&style=for-the-badge&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/aleksandra-vislova-a51ba9297)

---

| Tools Used |PySpark, Python, Power BI Desktop |
|------|-------------|

## Files in This Repository

| File | Description |
|------|-------------|
| `README.md` | Project overview, dashboard visuals, and findings |
| `notebooks/banking_clients_eda.ipynb` | PySpark EDA notebook — data loading, cleaning, feature engineering, correlation and bivariate analysis |
| `PowerBI/banking_clients_dashboard.pbix` | Power BI Desktop file with 3-page interactive dashboard |
| `PowerBI/banking_clients_dashboard.pdf` | PDF file for viewing dashboard |
| `data/banking-clients.csv` | Core client records — demographics, income, loans, deposits |
| `data/banking-relationships.csv` | Banking relationship type lookup (Private Bank, Retail, Commercial, Institutional) |
| `data/gender.csv` | Gender lookup table |
| `data/investment-advisors.csv` | Investment advisor assignment per client |
| `img/` | Dashboard screenshots used in this README |

---

## Table of Contents
- [Project Background](#project-background)
- [Data](#data)
- [PySpark](#pyspark)
- [Power BI Dashboard](#power-bi-dashboard)
- [Key Findings](#key-findings)
- [Recommendations](#recommendations)
- [Summary](#summary)

---

## Project Background

This project analyses a multi-source banking dataset to identify the demographic, financial, and behavioural characteristics that drive credit risk and loan default exposure. Four relational tables — clients, banking relationships, gender, and investment advisors — are joined and processed in PySpark on Databricks, with engineered features for income segmentation and risk classification passed downstream to a three-page Power BI dashboard. The objective is to give credit risk teams and investment advisors an actionable view of where loan concentration risk is highest, which client segments are underserved, and how deposit and loan products correlate across the client base.

### Objectives

- Segment 3,000 clients by income tier (Low / Medium / High) and risk weighting (Very Low → Very High) to surface credit policy gaps
- Identify which banking relationship types carry the greatest loan exposure and average risk weighting
- Analyse deposit, checking, savings, and foreign currency account balances to detect cross-product correlation patterns
- Enable investment advisors to filter and drill into their assigned client portfolios by risk category, loyalty tier, and income band
- Provide a temporal view of client acquisition trends and deposit/loan growth from 1995 to 2021

---

## Data

| Source | File | Records | Key Fields |
|--------|------|---------|------------|
| Client records | `banking-clients.csv` | 3,000 | Age, Income, Loans, Deposits, Risk Weighting, Loyalty Classification |
| Banking relationships | `banking-relationships.csv` | 4 types | BRId, Banking Relationship (Private Bank, Retail, Commercial, Institutional) |
| Gender | `gender.csv` | 2 | GenderId, Gender |
| Investment advisors | `investment-advisors.csv` | Multiple | IAId, Investment Advisor Name |

---

## PySpark

### Step 1 — Data Loading & Joining

All 4 tables are loaded as Spark DataFrames and joined using left joins on `GenderId`, `BRId`, and `IAId` to produce a single enriched df.

```python
df = df_clients \
    .join(df_gender, on="GenderId", how="left") \
    .join(df_realtionships, on="BRId", how="left") \
    .join(df_advisors, on="IAId", how="left")
```

### Step 2 — Exploratory Data Analysis

Duplicate and missing values check confirms data integrity before feature engineering begins.

### Step 4 — Categorical Variables Analysis

8 categorical columns are profiled for distinct value counts and frequency distributions, including `Nationality`, `Fee Structure`, `Gender`, `Loyalty Classification`, `Risk Weightning`, `Amount of Credit Cards`, `IncomeCategory` and `Properties Owned`.

![Categorical Variable Distributions](https://github.com/aleksandra20050404/Banking-Clients-Loan-Approval/blob/main/img/categorical_var_distributions.jpg)

### Step 4 — Feature Engineering

Two derived columns are added to enable segmentation downstream:

**Income Category** — Equal-width binning across the full income range into Low / Medium / High tiers:

```python
bin_width = (max_income - min_income) / 3
df = df.withColumn("IncomeCategory",
    when(col("Estimated Income") <= low_threshold, "Low")
    .when(col("Estimated Income") <= high_threshold, "Medium")
    .otherwise("High"))
```

**Risk Category** — Numerical risk weights (1–5) mapped to descriptive labels:

```python
df = df.withColumn("RiskCategory",
    when(col("Risk Weighting") == 1, "Very Low")
    .when(col("Risk Weighting") == 2, "Low")
    .when(col("Risk Weighting") == 3, "Medium")
    .when(col("Risk Weighting") == 4, "High")
    .when(col("Risk Weighting") == 5, "Very High"))
```

**Numerical Variables Analysis** — Distribution plots for 10 financial variables (Age, Estimated Income, Superannuation Savings, Credit Card Balance, Bank Loans, Bank Deposits, Checking Accounts, Saving Accounts, Foreign Currency Account, Business Lending) reveal consistent right-skew across all monetary fields, indicating that a small segment of high-value clients holds disproportionately large balances.

![Numerical Variable Distributions](https://github.com/aleksandra20050404/Banking-Clients-Loan-Approval/blob/main/img/numerical_var_distributions.jpg)

### Step 5 — Correlation Analysis

Pearson correlation coefficients are calculated across all numerical pairs. Pairs exceeding |0.5| are flagged to identify collinear financial product relationships relevant to credit risk modelling.

### Step 6 — Temporal Analysis

Client acquisition is broken down by year and quarter using `Joined Bank` date parsing. Customer tenure (days since joining) is computed via `datediff` to support retention and lifetime value analysis.

### Step 7 — Bivariate Analysis

Key cross-variable relationships are examined:

| Analysis | Grouping | Metrics |
|----------|----------|---------|
| Income by Gender | GenderId | Avg Estimated Income, Count |
| Financial products by Loyalty Tier | Loyalty Classification | Avg Income, Avg Deposits, Avg Loans |
| Product ownership by Risk Category | RiskCategory | Avg Credit Cards, Avg Properties Owned |
| Financial metrics by Nationality | Nationality | Count, Avg Income, Avg Deposits, Avg Loans |

---

## Power BI Dashboard

The cleaned and enriched PySpark output is loaded into a three-page Power BI Desktop report. Each page is filterable by Gender, Year, Quarter, Risk Category, Income Category, Banking Relationship, Investment Advisor, and Estimated Income range.

## Calculated Table: `DimDate`

Built with **Modeling** tab → **New table**:

```dax
DimDate =
ADDCOLUMNS(
    CALENDAR(DATE(YEAR(MIN(final_data[Joined_Bank])),1,1), DATE(YEAR(MAX(final_data[Joined_Bank])),12,31)),
    "Year", YEAR([Date]),
    "MonthNo", MONTH([Date]),
    "MonthName", FORMAT([Date],"MMMM"),
    "Quarter", "Q" & ROUNDUP(MONTH([Date])/3,0)
)
```
Then marked as the model's official Date Table (**Modeling** tab → **Mark as date table**), using `Date` as the date column.

**Relationship:** `final_data[Joined_Bank]` (Many) → `DimDate[Date]` (One), single-direction cross-filter, active.

---

### Page 1 — Banking Clients Overview

![Banking Clients Overview](https://github.com/aleksandra20050404/Banking-Clients-Loan-Approval/blob/main/img/banking_clients_overview.jpg)

**Visuals:**
- *Total Clients by Income Category (bar chart)* — Low-income clients form the largest segment (Jade tier: 1,319 clients), followed by Silver (763), Gold (583), and Platinum (317)
- *Total Deposits and Total Loans by Income Category (horizontal bar)* — Low-income clients account for the highest absolute deposit and loan volumes ($1.05bn / $0.94bn), reflecting their dominant share in the client base
- *Deposits vs. Loans over Time (line chart)* — Both metrics trend sharply upward post-2015, with deposits peaking at $180M and loans at $145M in the most recent period
- *Client Detail Table* — Drillable list showing Name, Investment Advisor, Income Category, Loyalty Classification, and Risk Category per client

#### DAX Measures — Page 

```dax
Total Clients = COUNTROWS('banking-clients')
```
Counts every row in the client table. Used in the headline KPI card and as the axis value in the Clients by Income Category bar chart.

---

```dax
Total Deposits = SUM('banking-clients'[Bank Deposits])
```
Sums the `Bank Deposits` column across all rows in the current filter context. Feeds the headline KPI card and the horizontal bar chart broken down by Income Category.

---

```dax
Total Loans = SUM('banking-clients'[Bank Loans])
```
Sums `Bank Loans` for all clients in context. Used in the headline KPI card, the Income Category bar chart, and the Deposits vs. Loans over Time line chart.

---

```dax
Avg Estimated Income = AVERAGE('banking-clients'[Estimated Income])
```
Calculates the mean of `Estimated Income` across filtered clients. Displayed as a standalone KPI card ($171.31K).

---

```dax
Client Growth YoY =
VAR CurrentYear = CALCULATE(
    COUNTROWS('banking-clients'),
    YEAR('banking-clients'[Joined Bank]) = MAX(YEAR('banking-clients'[Joined Bank]))
)
VAR PreviousYear = CALCULATE(
    COUNTROWS('banking-clients'),
    YEAR('banking-clients'[Joined Bank]) = MAX(YEAR('banking-clients'[Joined Bank])) - 1
)
RETURN
DIVIDE(CurrentYear - PreviousYear, PreviousYear, 0)
```
Compares client count in the most recent year against the prior year to produce the 6.8% growth rate shown on the KPI card. `DIVIDE` handles division by zero gracefully when prior-year data is absent.

---

```dax
Clients Joined = COUNTROWS('banking-clients')
```
Same row count as Total Clients but scoped to the `Joined Bank` date slicer context, so it reflects the number of clients who joined within the selected year/quarter range.

---

### Page 2 — Deposits Overview

![Deposits Overview](https://github.com/aleksandra20050404/Banking-Clients-Loan-Approval/blob/main/img/deposits.jpg)

**Visuals:**
- *Account Balances over Time (line chart)* — Checking accounts consistently outpace savings and foreign currency balances across all years; all three show gradual growth with acceleration post-2015
- *Avg Income & Bank Deposits by Loyalty Classification (scatter/bar)* — Platinum clients hold the highest average deposits relative to income; Jade clients are the most numerous but show lower per-client deposit values
- *Superannuation Savings by Occupation (horizontal bar)* — Associate Professors and Structural Analysis Engineers lead superannuation balances (~$0.46M and $0.45M respectively), identifying high-savings professional segments
- *Bank Deposits by Age (scatter)* — Deposit concentration increases with age, with the largest balances held by clients aged 50–80
- *Gender toggle* — Enables side-by-side comparison of deposit behaviour between male and female clients

#### DAX Measures — Page 

---

```dax
Total Deposits = SUM('banking-clients'[Bank Deposits])
```
Same measure as Page 1 but on this page the filter context is narrowed by the Occupation and Income Category slicers, so the $996M KPI card reflects the filtered deposit pool rather than the full $2.01bn portfolio total.

---
```dax
Total Savings Accounts = SUM('banking-clients'[Saving Accounts])
```
Sums the `Saving Accounts` balance column across all clients in the current filter context. Displayed as a KPI card ($353.04M) and plotted on the Account Balances over Time line chart.

---
```dax
Foreign Currency Accounts = SUM('banking-clients'[Foreign Currency Account])
```

- Aggregates the `Foreign Currency Account` balance for all filtered clients. Shown as a KPI card ($44.63M) and as the third series on the Account Balances over Time line chart.
---

```dax
Avg Checking Accounts = AVERAGE('banking-clients'[Checking Accounts])
```
Returns the mean checking account balance per client ($319.16K). Used as a KPI card and as the primary series on the Account Balances over Time line chart, where it is aggregated by year using the `Joined Bank` date hierarchy.

---
```dax
Avg Saving Accounts = AVERAGE('banking-clients'[Saving Accounts])
```
Mean savings account balance per client ($237.26K). Appears as a KPI card alongside `Avg Checking Accounts`.

---
```dax
Avg Estimated Income = AVERAGE('banking-clients'[Estimated Income])
```
Reused from Page 1. On this page it is plotted against `Bank Deposits` by `Loyalty Classification` in the scatter chart, revealing income-to-deposit ratios across Jade, Silver, Gold, and Platinum tiers.

---
```dax
Total Superannuation Savings = SUM('banking-clients'[Superannuation Savings])
```
- Sums superannuation balances, used as the value axis in the Superannuation Savings by Occupation horizontal bar chart. The Occupation field groups rows natively from the source data — no additional DAX grouping is required.
---


### Page 3 — Loan & Lending

![Loan & Lending](https://github.com/aleksandra20050404/Banking-Clients-Loan-Approval/blob/main/img/loans.jpg)

**Visuals:**
- *Bank Loans by Banking Relationship Type (bar chart)* — Private Bank clients carry the highest loan volume ($99M), more than double Retail ($49M), Commercial ($35M), and Institutional ($27M) relationships, indicating concentrated exposure in the highest-tier segment
- *Bank Loan by Income Category (donut chart)* — Low-income clients account for 50.03% of total bank loans ($105.15M), Medium 32.91% ($69.16M), and High 17.07% ($35.87M) — a risk signal given low-income clients' relative repayment capacity
- *Avg Bank Loans & Avg Risk Weighting by Properties Owned (combo chart)* — Clients owning 2 properties show the highest average loan ($0.67M) and risk weighting (2.4), suggesting property-backed lending carries above-average portfolio risk
- *Loans vs. Business Lending over Time (line chart)* — Business lending growth outpaces retail loan growth post-2010, with both accelerating sharply toward 2020
- *Client Risk Table* — Drillable table showing Name, Risk Category, Total Loans, and Business Lending for individual client review

#### DAX Measures — Page 3
```dax
Total Loans = SUM('banking-clients'[Bank Loans])
```
- Reused core measure. On this page filtered by Investment Advisor and Risk Category slicers, producing the $210.18M KPI card value. Also plotted as a time series on the Loans vs. Business Lending line chart.
---

```dax
Total Business Lending = SUM('banking-clients'[Business Lending])
```
Sums the `Business Lending` column across all clients in context ($310.8M). Appears as a KPI card and as the second series on the Loans vs. Business Lending over Time line chart.

---
```dax
Total Credit Card Balance = SUM('banking-clients'[Credit Card Balance])
```
- Aggregates `Credit Card Balance` across all filtered clients. Shown as a standalone KPI card ($1.14M). The low value relative to loans confirms most credit exposure is driven by bank and business lending rather than revolving credit.

---
```dax
% High Risk Clients =
DIVIDE(
    CALCULATE(COUNTROWS('banking-clients'), 'banking-clients'[RiskCategory] = "High"),
    COUNTROWS('banking-clients'),
    0
)
```
- Counts clients where the engineered `RiskCategory` column equals "High" (Risk Weighting = 4), divides by total client count, and returns the percentage (7.4%). `DIVIDE` is used instead of `/` to return 0 rather than an error if the denominator is zero after filtering.
---

```dax
Avg Risk Weighting = AVERAGE('banking-clients'[Risk Weighting])
```
Returns the mean numerical risk score (1–5) across all clients in the current filter context (2.2). Used as a KPI card and as the secondary axis line on the Avg Bank Loans & Avg Risk Weighting by Properties Owned combo chart.

---

```dax
Total Properties Owned = SUM('banking-clients'[Properties Owned])
```
Sums the `Properties Owned` column across all clients (520 total). Used as a KPI card and as the axis grouping in the Avg Bank Loans & Avg Risk Weighting by Properties Owned combo chart.

---

```dax
Avg Bank Loans = AVERAGE('banking-clients'[Bank Loans])
```
Mean loan balance per client. Plotted as the bar series on the combo chart broken out by `Properties Owned` value (0, 1, 2, 3), showing that clients with 2 properties carry the highest average loan of $0.67M.

---

## Key Findings

| Finding | Detail |
|---------|--------|
| Low-income clients dominate loan volume | 50% of total bank loans held by low-income segment despite lowest repayment capacity |
| Private Bank relationship carries highest exposure | $99M in loans — 47% of total loan book concentrated in one relationship type |
| 7.4% of clients classified as High Risk | Avg risk weighting of 2.2 across the full portfolio |
| Deposit growth accelerating post-2015 | Peak deposits of $180M and loans of $145M in most recent period |
| Age and deposits are positively correlated | Clients aged 50–80 hold the largest individual deposit balances |
| Superannuation savings concentrated in professional occupations | Top 5 occupations account for disproportionate share of long-term savings |

---

## Recommendations

### Credit Risk Policy

| Priority | Action |
|----------|--------|
| 1 | Review loan approval thresholds for low-income clients — 50% of loan exposure with highest default probability |
| 2 | Introduce risk-tiered lending limits for Private Bank clients given $99M concentration in single relationship type |
| 3 | Flag clients with 2+ properties and High risk weighting for enhanced due diligence — highest avg loan and risk combo |


## Summary

| Strategy | Action |
|----------|--------|
| Retention | Target Jade-tier clients (1,319 — largest segment) with loyalty upgrade programmes to convert to Silver/Gold |
| Acquisition | Focus advisor-led outreach on Medium and High income segments, currently underrepresented in client base |
| Product cross-sell | Clients with high superannuation savings and low credit card balances are under-leveraged for investment products |

---




 

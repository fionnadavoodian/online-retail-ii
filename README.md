# Online Retail II — Net Sales, Retention & Customer Segmentation

## Overview

This project analyzes the [Online Retail II](https://archive.ics.uci.edu/dataset/502/online+retail+ii) dataset, a set of transactional records from a UK-based online retailer covering 2009–2011. The goal is to move from raw, messy transactional data to a clean analytical dataset and derive actionable insights around revenue, customer retention, and customer segmentation — the kind of end-to-end workflow a data/business analyst would run for a retail stakeholder.

## Business Questions

- **Net sales**: What is net revenue after accounting for returns and cancelled orders, and how does it trend over time?

- **Repeat purchase behavior**: What share of customers make more than one purchase, and how does that compare to one-time buyers?
- **Cohort retention**: How well does the business retain customers over time when grouped into monthly acquisition cohorts?
- **Segmentation**: What distinct customer segments emerge (e.g. via RFM analysis), and how do they differ in value and behavior?

## Notebook Map

**Execution order matters.** `03` loads `orders_per_customer.csv` and `05` loads both `orders_per_customer.csv` and `revenue_per_customer.csv` — files produced by `02` and `03` respectively, not recomputed. Run notebooks in numeric order (`01` → `02` → `03` → `04` → `05`); `03` and `05` will fail with a missing-file error if run out of order. `04` has no such dependency — it only needs `01`'s output.

| Notebook | Input | Produces | Answers |
|---|---|---|---|
| `01_data_cleaning` | `data/raw/online_retail_II.csv` | `revenue_df.csv`, `customer_df.csv` (cleaned, flagged, with `line_revenue`) | — (cleaning notebook; no analytical question) |
| `02_kpi_definition` | `revenue_df.csv`, `customer_df.csv` | Gross/Net Sales, Return Rate, AOV, Repeat Purchase Rate, Cohort Month | What is net revenue after returns, and what share of customers are repeat buyers? |
| `03_customer_analysis` | `customer_df.csv` | order-frequency distribution, repeat-vs-one-time comparison, top-customer rankings, country/time breakdowns | How are orders distributed across customers, do repeat buyers spend differently, who are the top customers, and how does behavior vary by country and over time? |
| `04_cohort_analysis` | `customer_df.csv` | cohort retention table/rate, revenue-by-cohort heatmap | What share of each cohort stays active month over month, and does spend per transaction change as customers age within a cohort? |
| `05_segmentation` | `customer_df.csv` | RFM table, tercile scores, `rfm_level` tiers, tier revenue concentration | Which customers are highest-value by Recency/Frequency/Monetary, and how concentrated is revenue across value tiers? |
| `06_product_cooccurrence_and_cancellations` | `customer_df.csv` | Product family mapping, cancellation match classification (same-StockCode / same-family / no-match) | Does product co-occurrence (buying multiple colour/pattern variants together) explain the cancellation pattern, or is it a reorder/fulfillment issue, or unrelated? |

## Tech Stack

- Python
- pandas
- matplotlib / seaborn
- Jupyter

## Findings

Full write-up with charts: [reports/final_report.md](reports/final_report.md)

- **RFM segmentation:** the Top tier (1,803 customers, 41.6%) generates 84.6% of revenue; Middle (1,909 customers, 44.0%) generates 14.0%; Low (626 customers, 14.4%) generates 1.4%.
- **International customers, not friction:** international customers are ~9.7% of the customer base but generate ~18.0% of revenue. Every reliable international country matches or beats the UK on both AOV and repeat rate — international demand isn't being suppressed by friction, it's a reach/acquisition constraint, not a retention one.
- **Product co-occurrence & cancellations:** the hypothesis that cancellations reflect customers sampling multiple product variants and keeping one is **not supported** — same-family matches account for just 0.2% of cancelled invoices.

## How to Reproduce

1. Clone the repo:
   ```
   git clone https://github.com/fionnadavoodian/online-retail-ii.git
   cd online-retail-ii
   ```
2. Install dependencies:
   ```
   pip install -r requirements.txt
   ```
3. Run the notebooks in order, `01` through `06` — see the [Notebook Map](#notebook-map)'s execution-order note above; `03` and `05` depend on files produced by earlier notebooks and will fail with a missing-file error if run out of order.
4. Outputs land in:
   - `data/processed/` — cleaned datasets and intermediate per-customer CSVs
   - `reports/figures/` — chart images referenced by `reports/final_report.md`

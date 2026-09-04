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

## Metrics Glossary

| Metric | First Defined In | Variable Name | Notes |
|---|---|---|---|
| Cancelled flag | `01` | `is_cancelled` | `Invoice` starts with 'C' |
| Zero-value flag | `01` | `is_zero_value` | Price=0 & Qty>0, real customer orders only |
| Line revenue | `01` | `line_revenue` | `Quantity × Price`, carried into both processed CSVs |
| Gross Sales | `02` | `revenue_gross_sales` | Non-cancelled `line_revenue`, full transaction set |
| Returns | `02` | `returns` | Cancelled `line_revenue` (negative) |
| Net Sales | `02` | `net_sale` | Gross + Returns |
| Return Rate | `02` | `return_rate` | \|Returns\| / Gross Sales × 100 |
| AOV | `02` | `aov` | Gross Sales ÷ distinct non-cancelled invoices |
| Customer-Attributed Gross Sales | `02` | `customer_gross_sales` | Gross Sales restricted to known Customer ID |
| Orders per Customer | `02` | `orders_per_customer` | Saved to `orders_per_customer.csv`; loaded (not recomputed) in `03` and `05` |
| Repeat Customer flag | `02` | `is_repeat_customer` | Boolean Series; **recomputed in `03`** as a column, merged explicitly on Customer ID |
| Repeat Purchase Rate | `02` | `repeat_purchase_rate` | % of customers with >1 order |
| Cohort Month | `02` | `cohort_month` | First non-cancelled purchase month per customer; **same concept recomputed in `04`** as `FirstPurchaseMonth` via a different (heavier) implementation |
| Revenue per Customer | `03` | `revenue_per_customer` | First true per-customer revenue table. Saved to `revenue_per_customer.csv`; loaded (not recomputed) in `05` |
| Top Customers by Revenue | `03` | `top_customers` | |
| Top Customers by Order Volume | `03` | `number_orders` | |
| First Purchase Month (cohort) | `04` | `customer_df['FirstPurchaseMonth']` | See Cohort Month above — same thing, different notebook |
| Periods Since Acquisition | `04` | `customer_df['PeriodSinceAcquisition']` | Months between `InvoiceMonth` and `FirstPurchaseMonth` |
| Cohort Retention Counts | `04` | `cohort_df` | Pivot: cohort × period → active customer count |
| Cohort Retention Rate | `04` | `normalized_cohort_df` | `cohort_df` normalized to each cohort's period-0 size |
| Median Revenue by Cohort | `04` | `average_sales` | Cohort × period → median `line_revenue` |
| RFM Recency | `05` | `customer_recency` (col `Customer_recency`) | Days since last non-cancelled purchase |
| RFM Frequency | `05` | `customer_frequency` (col `Customer_frequency`) | Loaded from `orders_per_customer.csv` and renamed — same metric as Orders per Customer above, not recomputed |
| RFM Monetary | `05` | `customer_monetary` (col `Customer_monetary`) | Loaded from `revenue_per_customer.csv` and renamed — same metric as Revenue per Customer above, not recomputed |
| RFM Table | `05` | `rfm_table` | Recency + Frequency + Monetary merged, one row/customer |
| RFM Tercile Scores | `05` | `recency_quartile`, `frequency_quartile`, `monetary_quartile` (cols on `data_q`) | Named "quartile" but actually 3 buckets (`q=3`), not 4 |
| RFM Score | `05` | `rfm_score` | Sum of the three tercile scores (range 3–9) |
| RFM Segment String | `05` | `rfm_segment` | Concatenated tercile scores, e.g. `"333"` |
| RFM Tier | `05` | `rfm_level` | Top / Middle / Low, from `rfm_score` |
| Tier Revenue Concentration | `05` | `tier_summary` | % of customers vs. % of revenue per tier |

## Tech Stack

- Python
- pandas
- matplotlib / seaborn
- Jupyter

## Findings

_TBD — to be filled in as the analysis progresses._

## How to Reproduce

_TBD — setup and run instructions to be added once the analysis pipeline is finalized._

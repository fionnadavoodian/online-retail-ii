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
| RFM Tercile Scores | `05` | `recency_tercile`, `frequency_tercile`, `monetary_tercile` (cols on `data_q`) | q=3 buckets |
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

Full write-up with charts: [reports/final_report.md](reports/final_report.md)

- **Data quality:** the raw export needed real cleanup before it could support analysis — 5,268 duplicate rows, 3 bad-debt invoices, and 1,336 internal write-offs removed; a further 1,174 Price=0 rows split into 1,134 dropped (no Customer ID) and 40 kept-but-flagged (`is_zero_value`).
- **Revenue:** Net Sales of £9,737,087 after an 8.41% return rate on £10,631,067 Gross Sales. AOV is £532.54. 65.58% of identified customers (2,845 of 4,338) are repeat buyers.
- **Purchase patterns:** repeat customers average £2,908 in revenue vs. £411 for one-time buyers (~7x gap); the top 10 customers alone account for ~17% of total non-cancelled revenue. The business is 88.8% UK by transaction volume.
- **Cohort retention:** retention falls from 100% at acquisition to roughly 15–25% by month 1, then stabilizes in a 20–40% band for later months. Revenue per line item stays flat (~£5–16.50) across every cohort and period, regardless of how retention moves.
- **RFM segmentation:** the Top tier (1,803 customers, 41.6%) generates 84.6% of revenue; Middle (1,909 customers, 44.0%) generates 14.0%; Low (626 customers, 14.4%) generates 1.4%.
- **Product co-occurrence & cancellations:** the hypothesis that cancellations reflect customers sampling multiple product variants and keeping one is **not supported** — same-family matches account for just 0.2% of cancelled invoices. Instead, 47.0% look like same-item reorders/corrections and 52.8% have no related order at all.

## How to Reproduce

_TBD — setup and run instructions to be added once the analysis pipeline is finalized._

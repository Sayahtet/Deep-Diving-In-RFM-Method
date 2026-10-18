# Customer RFM Analysis and Segmentation

A customer-segmentation project using **Recency, Frequency, and Monetary (RFM)** analysis in Python, with a Power BI report comparing basic median-based and advanced rule-based segments.

> **Analysis scope:** The notebook’s executed outputs describe 2,000 transactions from **January 1 through December 31, 2025**, associated with 812 customers. Monetary values are reported in the dataset’s raw units. The source currency and source-data provenance were not established in the supplied files. RFM is descriptive behavioral analysis; segment labels such as “At Risk” are not validated churn predictions.

## Project files

| File | Description |
|---|---|
| `RFM_analysis.ipynb` | Python notebook covering data inspection, RFM aggregation, basic and advanced segmentation, and exploratory repeat/return analysis. |
| `RFMCustomerSegmentation.pbix` | Power BI report with segment summaries and visual comparisons. |
| `README.md` | Project methodology, calculation steps, findings, and limitations. |

The notebook reads an Excel input from a machine-specific Windows path (`RFM_Raw_Data.xlsx`). That source workbook was not included with the uploaded project files, so the notebook is not fully portable as-is. Update the path to the location of an authorized copy of the source data before rerunning.

## Dataset overview

The notebook reports the following input schema and coverage:

- **Rows:** 2,000 transaction records
- **Customers:** 812 unique customer IDs
- **Date range:** 2025-01-01 to 2025-12-31 (365 calendar days)
- **Fields:** Transaction ID, Date, Product ID, Product Name, Product Category, Quantity, PPU, Amount, Customer ID, and Region
- **Missing values:** The notebook's `info()` output shows 2,000 non-null values in each of the 10 columns.
- **Customer-level total Amount:** 59,851,600,000 (reconciles to the advanced segment summary)

The notebook drops `Transaction ID` and `Product ID` for its customer-level analysis. The remaining data is grouped by `Customer ID`. The files do not establish whether the data is real, anonymized, or synthetic, and do not state a currency or transaction direction. Accordingly, this README does not describe the data as verified banking data, customer spending, bank revenue, or profit.

## RFM definitions and calculation steps

The analysis is performed per customer, using the 2,000 transaction rows in the one-year observation window.

### 1. Define the snapshot date

The notebook sets the analysis date to one day after the latest transaction date:

```text
Analysis date = max(transaction date) + 1 day
              = 2025-12-31 + 1 day
              = 2026-01-01
```

This convention makes a transaction on December 31 have a recency of one day.

### 2. Aggregate customer-level metrics

For each customer `c`:

```text
Recency(c)   = Analysis date − latest transaction date for c
Frequency(c) = number of transaction rows for c in the observation window
Monetary(c)  = sum of Amount for c in the observation window
```

Examples from the notebook output:

| Customer | Latest transaction | Recency | Frequency | Monetary total |
|---|---:|---:|---:|---:|
| Customer-002 | 2025-12-25 | 7 days | 3 | 17,425,000 |
| Customer-004 | 2025-11-04 | 58 days | 5 | 69,590,000 |
| Customer-998 | 2025-10-13 | 80 days | 5 | 852,750,000 |

Frequency is a count of transaction records, not necessarily distinct shopping occasions: the notebook does not demonstrate transaction-ID uniqueness checks or a separate order/event definition. Monetary is the sum of `Amount`, not a documented bank revenue or profit measure.

### 3. Basic segmentation: median split

The notebook computes the median of each of the three customer-level metrics and classifies values as high or low:

```text
R = High if Recency < median Recency; otherwise Low
F = High if Frequency > median Frequency; otherwise Low
M = High if Monetary > median Monetary; otherwise Low
```

The direction differs for Recency because a smaller number of days means a more recent transaction. Values exactly equal to the median are classified as **Low** by these rules. The notebook then assigns one of eight combinations:

| RFM pattern | Segment label in notebook |
|---|---|
| H-H-H | Champions |
| H-H-L | Loyal High Value |
| H-L-H | Big Spenders (Occasional) |
| H-L-L | Potential Loyalists |
| L-H-H | At Risk High Value |
| L-H-L | Former Loyal Low Spend |
| L-L-H | Lost High Spenders |
| L-L-L | Lost / Low Value |

The notebook does not show the numeric median cutoffs in its saved outputs, so they are not invented here. The basic segment counts below are the executed notebook’s counts.

### 4. Advanced segmentation: explicit business-style bands

The advanced method applies these cutoffs:

```text
R = High if Recency ≤ 60 days; otherwise Low
F = High if transaction count ≥ 3; otherwise Low
M = High if the customer's cumulative share in descending Monetary order is ≤ 80.1%; otherwise Low
```

Customers are sorted from highest to lowest Monetary before computing the running cumulative share. The advanced segmentation then applies the same eight H/L combinations and the corresponding labels in the notebook (for example, “Champions,” “At Risk H Value,” and “Lost / L Value”).

The 60-day and 3-transaction boundaries are analysis choices, not universally valid business standards. The Monetary rule is a **cumulative Pareto-style cutoff**, not an individual Amount threshold or a standard five-quantile RFM score. Customer assignment around the cumulative boundary can depend on ordering, including how tied amounts are handled.

## Findings

### Overall customer activity

- The observed population contains **812 customers** and **2,000 transactions**.
- Average transaction count is approximately **2.46 transactions per customer** (`2,000 ÷ 812`).
- **292 customers (35.96%)** made one transaction; **520 (64.04%)** made at least two; **313 (38.55%)** made at least three; and **163 (20.07%)** made at least four. These are cumulative repeat-frequency counts from the notebook.
- The notebook’s maximum observed customer frequency is 10 transactions.

### Advanced RFM segment results

The following totals are from the notebook’s advanced segment aggregation. Shares are calculated against the reported 812 customers and total Amount of 59,851,600,000.

| Advanced segment | Customers | Customer share | Total Amount | Amount share |
|---|---:|---:|---:|---:|
| Champions | 66 | 8.13% | 31,807,055,000 | 53.14% |
| At Risk H Value | 47 | 5.79% | 14,921,080,000 | 24.93% |
| Lost / L Value | 371 | 45.69% | 4,954,610,000 | 8.28% |
| Former Loyal L Spend | 101 | 12.44% | 2,765,895,000 | 4.62% |
| Loyal H Value | 99 | 12.19% | 2,565,690,000 | 4.29% |
| Potential Loyalists | 107 | 13.18% | 1,666,380,000 | 2.78% |
| Lost H Spenders | 13 | 1.60% | 733,000,000 | 1.22% |
| Big Spenders (Occasional) | 8 | 0.99% | 437,890,000 | 0.73% |
| **Total** | **812** | **100.00%** | **59,851,600,000** | **100.00%** |

**Interpretation:** Champions are only 8.13% of customers but contribute 53.14% of the summed Amount. The 47 customers classified as At Risk H Value contribute another 24.93%. Together, these two groups are 13.92% of customers and account for **78.07%** of Amount. This concentration makes them reasonable cohorts to investigate for retention-oriented experiments, but the “At Risk” label is not evidence that customers will churn.

Lost / L Value is the largest advanced segment: **371 customers (45.69%)**, contributing **8.28%** of Amount. This label reflects the chosen thresholds and this observed period; it should not be treated as a verified lost-customer state.

### Basic segment counts

The notebook's median-based method produced these counts:

| Basic segment | Customers |
|---|---:|
| Lost / Low Value | 229 |
| Champions | 183 |
| Potential Loyalists | 122 |
| Lost High Spenders | 86 |
| At Risk High Value | 75 |
| Big Spenders (Occasional) | 62 |
| Loyal High Value | 36 |
| Former Loyal Low Spend | 19 |
| **Total** | **812** |

The basic and advanced approaches intentionally produce different segment sizes because they use different scoring cutoffs. They are two classifications of the same customer cohort, not evidence that customer behavior changed.

### Exploratory recency and repeat analysis

The notebook bins recency as `0–30`, `31–60`, `61–90`, and `90+` days and reports return rates of 39.87%, 40.16%, 27.52%, and 20.33%, respectively. The notebook commentary concludes that 60 days is a useful maximum return interval. This should be treated as **exploratory only**: the reported interval-level rates and the way the `Return` flag is constructed need additional validation before selecting an operational follow-up window.

The repeat-frequency table shows a steep drop between the first and second observed transaction: the repeat rate falls from 100% at the first transaction to 64.04% at the second (a **35.96 percentage-point** decrease). It falls to 38.55% at the third transaction and 20.07% at the fourth. This describes frequency distribution in the observed year; it is not a cohort-retention or causal analysis.

## Power BI report

The supplied `.pbix` contains two report pages:

1. **RFM SEGMENTATION** — segment summaries for the basic and advanced classifications, with customer-count and amount-related metrics, plus date and region controls.
2. **SUMMARY FOR BOTH RFM METHODS** — side-by-side visuals intended to compare segment-level customer counts and Amount across methods, with a segment-field selector.

The report definitions include measures/visuals for total customers, average transaction amount, and total Amount. The summary page also contains a narrative highlighting the 812 customers, 2,000 transactions, 59,851,600,000 total Amount, and concentration in Champions and At Risk H Value.

## Credit Dataset 
 ### Saya Charm Myae Aung(Anlytica)
- [Dataset Link](https://www.kaggle.com/datasets/charmmyaeaung/raw-sales-dataset-for-rfm-customer-segmentation)
## Tools

- Python: pandas, NumPy, Matplotlib, and Jupyter Notebook
- Microsoft Power BI

# Sales Data Cleaning and Exploratory Analysis: Report

> **Scope:** 1,250 raw records exported from the company's sales systems, cleaned and analysed with Python and Pandas.
> The data is synthetic (generated with a fixed seed by `src/generate_raw_data.py`) but built to reproduce real export problems.
> Every number below is produced by `notebooks/sales_analysis.ipynb` from `data/cleaned_sales_data.csv`.

## 1. Executive summary

After cleaning, the company has **1,187 valid orders** worth **$85,667.10** in revenue
(02 Jan 2025 to 31 Dec 2025), an **average order value of $72.17**
and **2,408 units** sold.

| KPI | Value |
|---|---:|
| Total revenue | $85,667.10 |
| Total orders | 1,187 |
| Total units sold | 2,408 |
| Average order value | $72.17 |
| Median order value | $47.99 |
| Average units per order | 2.03 |

**Key findings**

1. **Strong year-end seasonality.** November ($10,651) and December ($12,977) together earn 28.0% of dated revenue.
   Each is well above the Jan-Oct monthly average of $6,073 (November is 1.8x and December 2.1x that average).
2. **Revenue is driven by a few high-ticket lines.** Electronics and Furniture bring 66.8% of revenue.
   The top 5 products (Smart Watch, Standing Desk, Bluetooth Headphones, Office Chair, Bookshelf) bring 43.7%.
3. **Volume and value tell different stories.** Office Supplies is 27.5% of orders but only 9.2% of revenue
   (average order $24.08). Notebook Pack is the best seller by units (286),
   while Smart Watch leads by revenue ($9,607).
4. **West is the largest region** (29.3% of revenue); **Central is the smallest** (11.2%).
   Average order values are similar across regions ($68.71 to $77.34), so the gap comes from the number of orders, not their size.
5. **Cards dominate payments.** Credit and debit cards are 57.9% of orders; Credit Card alone is 36.7%.

## 2. Data quality: before and after

The raw file had **233 of 1,250 rows (19%) with at least one missing value**, 50 duplicate records,
5 different date formats and a Total_Sales that did not equal Quantity x Unit_Price in 62 rows (plus 45 missing totals). The "After" column below was **measured on the cleaned file**, not assumed.

| Metric | Before | After |
|---|---:|---:|
| Total rows | 1,250 | 1,187 |
| Exact duplicate rows | 28 | 0 |
| Hidden duplicates (same Order_ID, different formatting) | 22 | 0 |
| Rows with at least one missing value | 233 | 20 |
| Cells with extra / repeated spaces | 799 | 0 |
| Cells with inconsistent capitalisation | 467 | 0 |
| Misspelled values (typos) | 19 | 0 |
| Distinct date formats | 5 | 1 |
| Unreadable or out-of-range dates | 9 | 0 |
| Money values stored as text ($ or comma) | 99 | 0 |
| Distinct Region spellings | 51 | 5 |
| Distinct Product spellings | 149 | 21 |
| Quantity <= 0 or above cap | 24 | 0 |
| Unit_Price <= 0 or implausibly high | 22 | 0 |
| Total_Sales != Quantity x Unit_Price | 62 | 0 |

**Row reconciliation (every raw row is accounted for)**

| Step | Rows |
|---|---:|
| Rows in raw file | 1,250 |
| - Exact duplicate rows removed | 28 |
| - Re-formatted duplicate rows removed | 22 |
| - Invalid records quarantined | 13 |
| = Rows in cleaned dataset | 1,187 |

Remaining missing values after cleaning: **20 undated orders** (the date cannot be derived, so it is left blank). All other gaps were either rebuilt from other columns or labelled `Unknown`.

## 3. Cleaning decisions

Rule of thumb: **fix** what can be proved from the same row/customer/product, **label** what cannot be derived, **quarantine** what is invalid and unrepairable. Nothing was deleted silently.

| Issue | Records | Action | Decision | Rationale |
|---|---:|---|---|---|
| Exact duplicate rows | 28 | Removed | Kept the first occurrence; logged in Flagged_Records | Identical copies of the same order would double-count revenue. |
| Hidden duplicates (same order, different formatting) | 22 | Removed | Matched on the standardised Order_ID; verified all other fields were identical first | Capitalisation, date format or '$' signs hid these from a plain duplicated() check. |
| Inconsistent capitalisation and extra spaces in text | 799 | Standardised | Trimmed, collapsed repeated spaces, mapped variants to the most common spelling | 'west', ' WEST' and 'West' must be one group in every pivot table and chart. |
| Misspelled Product / Region values | 19 | Corrected | Only values 1 edit away from exactly ONE frequent value; each fix is listed on the Data_Quality sheet | A conservative rule avoids 'fixing' a real value into a wrong one. |
| Order_ID / Customer_ID with wrong case, spaces or missing hyphen | 223 | Standardised | Re-formatted to ORD-##### and CUST-#### | Needed so that duplicates and repeat customers can be matched. |
| Dates written in 5 different formats | 1,238 | Standardised | Parsed to real datetimes; slash dates read as DAY/MONTH/YEAR (verified from the data) | Text dates cannot be sorted or grouped by month. |
| Impossible, unreadable or out-of-range dates | 9 | Set to missing + flagged | Not guessed (for example '2052' is not assumed to mean 2025) | Inventing a date could put revenue in the wrong month; the order still counts in totals. |
| Missing Order_Date | 12 | Left blank + flagged | No imputation | No other column can tell us the date. |
| Prices / totals stored as text ('$29.99', '1,250.50') | 99 | Parsed to numbers | Removed '$' and thousands commas | Text cannot be summed. |
| Missing Category | 24 | Filled from Product | Each product belongs to exactly one category (verified) | Provable from the same row, so this is not a guess. |
| Missing Region | 30 | Filled from Customer_ID | Used only customers whose other orders all share one region (verified) | Provable from the same customer's other orders. |
| Missing Customer_ID | 36 | Labelled 'UNKNOWN' | Not imputed | Cannot be derived; the order still counts in revenue. |
| Missing Product | 12 | Labelled 'Unknown Product' | Not imputed | Cannot be derived reliably (price alone is not unique). |
| Missing Payment_Method | 30 | Labelled 'Unknown' | Not imputed | Guessing the most common method would invent data. |
| Invalid Quantity (negative, zero, missing, 999) | 31 | Recovered | Quantity = Total_Sales / Unit_Price, accepted only if a whole number from 1 to 100 | The relationship Total = Quantity x Price gives proof when price and total are valid. |
| Invalid Unit_Price (negative, zero, 9999.99, decimal shift, missing) | 37 | Recovered | Unit_Price = Total_Sales / Quantity, accepted only within +-15% of the product's typical price | The product's typical price is a second, independent piece of evidence. |
| Missing Unit_Price AND Total_Sales | 6 | Estimated + flagged | Used the product's typical (median) price | Only estimate in the project; its revenue is reported separately in the KPI table. |
| Invalid numbers that cannot be repaired | 13 | Quarantined (excluded from analysis) | 6 negative qty + negative total (possible returns), 4 zero-quantity lines, 3 zero-price lines; listed with reasons in Flagged_Records | Not deleted silently: they are kept for the business to review. Netting possible returns needs a business rule. |
| Total_Sales missing or wrong | 102 | Recalculated | Total_Sales = Quantity x Unit_Price (60 wrong, 42 missing) | A derived column must always agree with its inputs. |

### Records that were removed or set aside (not silently deleted)

* **50 duplicate rows** were removed (each points to the row it duplicates).
* **13 invalid records** were quarantined and excluded from every KPI. They are listed with reasons in `data/flagged_records.csv` and on the `Flagged_Records` sheet.
  Their reported totals sum to $-434.88 (negative = possible returns). If the business confirms they are valid returns, they can be netted off revenue.

| Raw row | Order_ID | Product | Qty | Price | Total | Reason |
|---:|---|---|---:|---:|---:|---|
| 14 | ORD-10012 | Gel Pen Set | -1.0 | 5.99 | -5.99 | Negative quantity with negative total: possible return/refund, needs business confirmation |
| 329 | ORD-10321 | Printer Paper Ream | -3.0 | 6.49 | -19.47 | Negative quantity with negative total: possible return/refund, needs business confirmation |
| 341 | ORD-10333 | Bluetooth Headphones | 0.0 | 53.99 | 0.0 | Quantity is zero and Total_Sales is zero: empty order line, cannot be verified |
| 422 | ORD-10413 | USB-C Hub | 1.0 | 0.0 | 0.0 | Unit_Price invalid (zero/negative) and Total_Sales is also zero: cannot be verified (possible free item) |
| 447 | ORD-10438 | USB-C Hub | 0.0 | 34.5 | 0.0 | Quantity is zero and Total_Sales is zero: empty order line, cannot be verified |
| 467 | ORD-10458 | Dumbbell Set | 3.0 | 0.0 | 0.0 | Unit_Price invalid (zero/negative) and Total_Sales is also zero: cannot be verified (possible free item) |
| 527 | ORD-10516 | Wireless Mouse | -2.0 | 24.99 | -49.98 | Negative quantity with negative total: possible return/refund, needs business confirmation |
| 684 | ORD-10669 | USB-C Hub | 0.0 | 34.5 | 0.0 | Quantity is zero and Total_Sales is zero: empty order line, cannot be verified |
| 773 | ORD-10756 | Printer Paper Ream | 0.0 | 6.49 | 0.0 | Quantity is zero and Total_Sales is zero: empty order line, cannot be verified |
| 801 | ORD-10784 | Bookshelf | 1.0 | 0.0 | 0.0 | Unit_Price invalid (zero/negative) and Total_Sales is also zero: cannot be verified (possible free item) |
| 818 | ORD-10800 | Stapler | -2.0 | 11.99 | -23.98 | Negative quantity with negative total: possible return/refund, needs business confirmation |
| 848 | ORD-10830 | Dumbbell Set | -4.0 | 69.99 | -279.96 | Negative quantity with negative total: possible return/refund, needs business confirmation |
| 989 | ORD-10968 | Desk Lamp | -2.0 | 27.75 | -55.5 | Negative quantity with negative total: possible return/refund, needs business confirmation |

### Sensitivity of the results to the cleaning choices

* Orders with an **estimated price** (6 rows) carry $607.90 (0.7% of revenue).
* **Undated orders** (20 rows, $1,310.20, 1.5% of revenue) are in all totals but not in the monthly trend.
* **Unknown** product / payment / customer labels cover 12 / 30 / 36 orders.

## 4. Exploratory analysis

### Best-selling products (top 10 by revenue)

| Product | Category | Orders | Units | Revenue | Share |
|---|---|---:|---:|---:|---:|
| Smart Watch | Electronics | 49 | 65 | $9,606.85 | 11.2% |
| Standing Desk | Furniture | 26 | 28 | $8,868.20 | 10.4% |
| Bluetooth Headphones | Electronics | 62 | 118 | $6,973.82 | 8.1% |
| Office Chair | Furniture | 38 | 50 | $6,866.60 | 8.0% |
| Bookshelf | Furniture | 45 | 55 | $5,136.06 | 6.0% |
| Dumbbell Set | Sports & Fitness | 41 | 72 | $5,011.28 | 5.8% |
| Coffee Maker | Home & Kitchen | 52 | 87 | $4,723.63 | 5.5% |
| Blender | Home & Kitchen | 44 | 86 | $4,103.14 | 4.8% |
| Portable Charger | Electronics | 77 | 136 | $4,035.14 | 4.7% |
| Webcam HD | Electronics | 50 | 89 | $3,981.61 | 4.6% |

Top 5 by units: Notebook Pack (286), Printer Paper Ream (271), Gel Pen Set (269), Wireless Mouse (144), Portable Charger (136).

### Categories

| Category | Orders | Units | Revenue | Share | Avg order |
|---|---:|---:|---:|---:|---:|
| Electronics | 397 | 678 | $33,109.74 | 38.6% | $83.40 |
| Furniture | 177 | 247 | $24,108.19 | 28.1% | $136.20 |
| Home & Kitchen | 191 | 316 | $13,272.99 | 15.5% | $69.49 |
| Office Supplies | 327 | 990 | $7,873.55 | 9.2% | $24.08 |
| Sports & Fitness | 95 | 177 | $7,302.63 | 8.5% | $76.87 |

### Monthly sales trend

| Month | Orders | Units | Revenue | Avg order | MoM |
|---|---:|---:|---:|---:|---|
| Jan | 75 | 155 | $5,431.01 | $72.41 |  |
| Feb | 75 | 139 | $4,766.02 | $63.55 | -12.2% |
| Mar | 92 | 182 | $6,636.27 | $72.13 | +39.2% |
| Apr | 94 | 197 | $7,319.07 | $77.86 | +10.3% |
| May | 83 | 154 | $5,589.21 | $67.34 | -23.6% |
| Jun | 96 | 198 | $5,731.43 | $59.70 | +2.5% |
| Jul | 102 | 197 | $7,072.22 | $69.34 | +23.4% |
| Aug | 86 | 166 | $5,837.60 | $67.88 | -17.5% |
| Sep | 84 | 162 | $6,314.70 | $75.17 | +8.2% |
| Oct | 88 | 181 | $6,030.83 | $68.53 | -4.5% |
| Nov | 132 | 295 | $10,651.05 | $80.69 | +76.6% |
| Dec | 160 | 345 | $12,977.49 | $81.11 | +21.8% |

Dated orders: 1,167 ($84,356.90). Undated: 20 ($1,310.20). Together they equal total revenue.

### Regions

| Region | Orders | Units | Revenue | Share | Avg order |
|---|---:|---:|---:|---:|---:|
| West | 359 | 685 | $25,088.81 | 29.3% | $69.89 |
| East | 228 | 497 | $17,291.66 | 20.2% | $75.84 |
| South | 220 | 450 | $17,014.35 | 19.9% | $77.34 |
| North | 243 | 517 | $16,695.38 | 19.5% | $68.71 |
| Central | 137 | 259 | $9,576.90 | 11.2% | $69.90 |

### Payment methods

| Payment method | Orders | Share of orders | Revenue | Avg order |
|---|---:|---:|---:|---:|
| Credit Card | 436 | 36.7% | $29,377.87 | $67.38 |
| Debit Card | 251 | 21.1% | $18,724.89 | $74.60 |
| PayPal | 225 | 19.0% | $18,666.38 | $82.96 |
| Cash | 137 | 11.5% | $9,017.53 | $65.82 |
| Bank Transfer | 108 | 9.1% | $7,439.02 | $68.88 |
| Unknown | 30 | 2.5% | $2,441.41 | $81.38 |

### Other observations

* The top 10% of orders (119 orders) account for 34.4% of revenue; only 2 orders exceed $500.
* 80.1% of the 302 identifiable customers placed two or more orders.

## 5. Charts

All charts are saved in `output/charts/` (and embedded in the workbook's `Charts` sheet).

| # | Chart | File |
|---|---|---|
| 1 | Monthly revenue trend | `01_monthly_revenue_trend.png` |
| 2 | Monthly order volume | `02_monthly_orders.png` |
| 3 | Top products by revenue | `03_top_products_revenue.png` |
| 4 | Top products by units | `04_top_products_units.png` |
| 5 | Revenue by category | `05_revenue_by_category.png` |
| 6 | Revenue by region | `06_revenue_by_region.png` |
| 7 | Payment method distribution | `07_payment_method_distribution.png` |
| 8 | Category x region heatmap | `08_category_region_heatmap.png` |
| 9 | Order value distribution | `09_order_value_distribution.png` |
| 10 | Data quality issues found | `10_data_quality_issues.png` |

![Monthly revenue](../output/charts/01_monthly_revenue_trend.png)
![Top products](../output/charts/03_top_products_revenue.png)

## 6. Recommendations

These follow from the findings above; they are suggestions to test, not proven causal effects.

1. **Plan inventory, staffing and promotions around November-December.** Demand in those two months is 1.8x to 2.1x a typical Jan-Oct month.
2. **Protect the high-ticket lines** (Smart Watch, Standing Desk): they generate the most revenue from relatively few orders, so stock-outs would be costly.
3. **Raise the value of Office Supplies orders** (e.g. bundles or add-ons): they are 27.5% of orders but 9.2% of revenue.
4. **Investigate the Central region.** Its order values are in line with other regions, so the shortfall is in order count. The data cannot say why (customer base, marketing, stores); that needs business input.
5. **Fix data capture at the source.** Make Order_ID, date, product, quantity and price mandatory; enforce one date format and a pick-list for Region / Payment_Method / Category; reject negative quantities and prices; calculate Total_Sales in the system instead of typing it; prevent re-imports that duplicate orders.

## 7. Assumptions and limitations

* One row is one order line, so *orders* equal distinct `Order_ID`s; currency is assumed to be USD; the data covers calendar year 2025.
* Slash dates are read as DAY/MONTH/YEAR. The first number reaches 31 while the second never exceeds 12, which rules out month-first.
* Possible returns (negative quantity and negative total) were quarantined, not netted; their treatment needs a business rule.
* Prices rebuilt from `Total / Quantity` are trusted only if they fall within +-15% of the product's usual price; a missing price with no usable total uses the product's median price (flagged).
* The dataset is a single year, so year-over-year growth cannot be measured, and seasonality is observed once.
* With a synthetic dataset the patterns (for example Q4 peak) are designed into the generator; the *method* is what this project demonstrates.

## 8. Reproducibility and validation

Run `python src/generate_raw_data.py`, then execute `notebooks/sales_analysis.ipynb` top to bottom. The last notebook section recomputes every KPI independently
from `data/cleaned_sales_data.csv`, checks the Excel workbook's stored values against it, and verifies that this report quotes the same numbers.

# Sales Data Cleaning and Exploratory Data Analysis

An end-to-end data analyst project: a **messy retail sales export** (1,250 rows) is inspected, cleaned with Pandas, analysed, charted and delivered as a formatted Excel workbook plus a written report.

**Tools:** Python 3, Pandas, NumPy, Matplotlib, Seaborn, Excel (XlsxWriter), Jupyter

## The business problem

A retail company sent a sales export from several systems. Before management can trust any number in it, the data must be cleaned, and the cleaning must be **explained and auditable**. Then the company wants answers: how much did we sell, what sells best, when, where and how do customers pay?

## Results at a glance

| KPI | Value |
|---|---:|
| Total revenue | **$85,667.10** |
| Total orders | **1,187** |
| Total units sold | **2,408** |
| Average order value | **$72.17** |

* **Seasonality:** November and December earn 28% of dated revenue (1.8x and 2.1x a typical Jan-Oct month).
* **Best categories:** Electronics (38.6% of revenue) and Furniture (28.1%). Top product by revenue: Smart Watch; by units: Notebook Pack.
* **Regions:** West is largest (29.3%), Central smallest (11.2%); average order values are similar, so the gap is order count.
* **Payments:** Credit Card is 36.7% of orders; cards in total are 57.9%.

### Data quality, before and after

| Problem | Before | After |
|---|---:|---:|
| Rows | 1,250 | 1,187 |
| Duplicate records (exact + hidden) | 50 | 0 |
| Rows with a missing value | 233 | 20 (undated orders only) |
| Cells with extra spaces | 799 | 0 |
| Cells with inconsistent capitalisation | 467 | 0 |
| Date formats in one column | 5 | 1 |
| Invalid quantity or price | 46 | 0 |
| Total_Sales not equal to Quantity x Unit_Price | 62 | 0 |
| Distinct Region spellings | 51 | 5 |

Every raw row is accounted for: **1,250 raw = 1,187 cleaned + 50 duplicates removed + 13 quarantined**.

## Project structure

```
Sales_Data_Analysis/
├── data/
│   ├── raw_sales_data.csv          # the messy export, exactly as delivered
│   ├── cleaned_sales_data.csv      # analysis-ready data (+ Cleaning_Flags column)
│   └── flagged_records.csv         # every removed duplicate / quarantined record, with the reason
├── notebooks/
│   └── sales_analysis.ipynb        # the whole analysis, executed, with outputs and charts
├── output/
│   ├── sales_analysis.xlsx         # Excel deliverable (see below)
│   └── charts/                     # 10 PNG charts
├── reports/
│   └── analysis_report.md          # written findings, decisions, recommendations
├── src/
│   ├── generate_raw_data.py        # seeded generator of the messy dataset
│   └── excel_export.py             # builds the formatted workbook
├── README.md
└── requirements.txt
```

### Excel workbook (`output/sales_analysis.xlsx`)

| Sheet | Content |
|---|---|
| **Summary** | KPIs, top performers, row reconciliation, assumptions. All **live formulas** on Cleaned_Data |
| **Analysis** | Monthly trend, category, region, payment, product tables and a category x region matrix, with native Excel charts. Live formulas |
| **Cleaned_Data** | The cleaned dataset with filters, frozen header, currency/date formats and a `Cleaning_Flags` audit column |
| **Raw_Data** | The original file, untouched, as text |
| **Data_Quality** | Before/after statistics, the full cleaning decision log, and typo corrections |
| **Flagged_Records** | Removed duplicates and quarantined records, each with the reason |
| **Charts** | The ten charts as images |

## How the cleaning works

The guiding rule: **never silently delete or invent data.**

| Situation | Treatment |
|---|---|
| Value can be **proved** from other data (category from product, region from the customer's other orders, quantity or price from `Total = Qty x Price`) | **Fix** and flag in `Cleaning_Flags` |
| Value cannot be derived (customer, product, payment method) | **Label** `Unknown`; the revenue still counts |
| Date is impossible or missing | Set to missing, flag it, keep the order in totals but out of the monthly trend |
| Numbers are invalid and cannot be repaired (possible returns, zero-value lines) | **Quarantine**: excluded from KPIs, kept in `flagged_records.csv` with a reason |
| Duplicates (including "hidden" ones differing only in formatting) | Remove the repeat, keep the first, log which row it duplicates |
| `Total_Sales` wrong or missing | Always recalculate as `Quantity x Unit_Price` |

The full decision log with counts and rationale is in `reports/analysis_report.md` (section 3) and on the `Data_Quality` sheet.

## Quality assurance

The final notebook section re-reads the files from disk and runs **34 automated checks**, including: unique Order_IDs, no duplicate rows, every `Total_Sales` equal to `Quantity x Unit_Price`, row reconciliation, every KPI and breakdown recomputed independently from the CSV, the Excel workbook's values compared with pandas, and the report quoting the same totals. The workbook's formulas were also recalculated in LibreOffice and matched the pandas figures with zero errors.

## How to reproduce

```bash
# 1. install dependencies (Python 3.10+)
pip install -r requirements.txt

# 2. (optional) regenerate the raw data: seeded, so the file is identical every time
python src/generate_raw_data.py

# 3. run the analysis: either open notebooks/sales_analysis.ipynb in Jupyter and "Run All", or headless:
jupyter nbconvert --to notebook --execute --inplace notebooks/sales_analysis.ipynb
```

Running the notebook rewrites `data/cleaned_sales_data.csv`, `data/flagged_records.csv`, `output/sales_analysis.xlsx`, `output/charts/*.png` and `reports/analysis_report.md`. Tested with Python 3.13, pandas 3.0, NumPy 2.5, Matplotlib 3.11, Seaborn 0.13.

## Assumptions and limitations

* The dataset is **synthetic**: generated to reproduce realistic export problems, so the business patterns (such as the Q4 peak) are designed in. The project demonstrates the *method*, not real company performance.
* One row is one order line, so orders equal distinct `Order_ID`s. Currency is assumed to be USD and the data covers calendar year 2025 only.
* Slash dates are read as DAY/MONTH/YEAR (checked from the data: the first number reaches 31, the second never exceeds 12).
* Possible returns were quarantined rather than netted off revenue; that needs a business rule.
* Prices rebuilt from `Total / Quantity` are accepted only within ±15% of the product's usual price; six orders with a missing price and no usable total were estimated from the product's median price and are flagged (0.7% of revenue).

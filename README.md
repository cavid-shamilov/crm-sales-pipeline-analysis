# crm-sales-pipeline-analysis
End-to-end CRM sales pipeline analysis using SQL Server and Power BI to evaluate conversion rates, win/loss performance, sales cycles, and sales team effectiveness.

## Data Quality Audit

### Step 1: Dataset Overview

The project uses four related CSV files to analyze CRM sales pipeline performance.

| Table | Records | Columns | Description |
|---|---:|---:|---|
| `accounts` | 85 | 7 | Company and account information |
| `sales_pipeline` | 8,800 | 8 | Sales opportunities, stages, dates, and deal values |
| `sales_teams` | 35 | 3 | Sales agents, managers, and regional offices |
| `products` | 7 | 3 | Product names, series, and suggested retail prices |

*Record counts exclude header rows.*

**Initial observations**
- All four CSV files were successfully separated into their expected columns.
- The `accounts.year_established` column was initially interpreted as dates by Excel. The import process was adjusted to preserve the establishment years as text values.
- Dates in `sales_pipeline` appeared in different display formats in Excel and the original CSV. This was recorded for further validation; the actual date values have not yet been independently verified.

**Next step:** Review the data dictionary and assess missing values, duplicates, data types, and data validity across all four tables.

---

### Step 3: Missing Values Audit

A missing-value audit was conducted across all four tables.

| Table | Column | Missing values | Assessment |
|---|---|---:|---|
| `sales_pipeline` | `account` | 1,425 (16.19%) | Present only in Prospecting and Engaging stages; requires further investigation |
| `sales_pipeline` | `engage_date` | 500 (5.68%) | Missing values occur only in Prospecting; potentially expected |
| `sales_pipeline` | `close_date` | 2,089 (23.74%) | Missing values occur only in open opportunities |
| `sales_pipeline` | `close_value` | 2,089 (23.74%) | Missing values occur only in open opportunities |
| `accounts` | `subsidiary_of` | 70 (82.35%) | Potentially valid for companies without a parent company |
| All other audited columns | — | 0 | No missing values detected |

**Key findings**

- Missing `close_date` and `close_value` values align with opportunities that have not yet reached a final outcome.
- Missing `engage_date` values occur only in the Prospecting stage, which may be consistent with the sales process.
- Missing account assignments require further investigation before deciding whether they affect analysis.
- Missing `subsidiary_of` values may indicate that a company has no parent company; this interpretation has not yet been confirmed.
- No missing values were found in the `products` table or any of the three columns in `sales_teams`.

**Data handling decision:** No missing values were filled with arbitrary values or deleted during this audit. Each missing-value pattern will be assessed according to its business meaning before analysis.

**Status:** Completed.

---

### Step 4: Duplicate Records Audit

Duplicate checks were performed across the four tables to identify repeated key values and fully duplicated records.

**Key findings**
- No duplicate values were found in `sales_pipeline.opportunity_id`.
- No duplicate account names were found in `accounts.account`.
- No duplicate product names were found in `products.product`.
- No duplicate sales agent names were found in `sales_teams.sales_agent`.
- No fully duplicated rows were identified in the `sales_pipeline` or `accounts` tables.

Repeated values in non-key columns, such as `deal_stage`, `product`, and `sales_agent` in the sales pipeline, are expected because multiple opportunities can share the same stage, product, or sales representative.

**Conclusion:** No duplicate-record issues were identified in the checks performed. No records were removed during this audit.

**Status:** Completed.

---

### Step 5: Data Types and Formats Audit

The data types and display formats of key columns were reviewed in Excel.

**Key findings**
- Sales date values were recognized by Excel as valid dates. The dates were displayed in day-month-year (DMY) format.
- `close_value`, `sales_price`, `revenue`, and `employees` were recognized as numeric values, as confirmed using Excel's `ISNUMBER()` function.
- `year_established` values were preserved correctly as years and retained as text to avoid unintended date conversions.
- Numeric values were displayed without currency symbols, with periods used as thousands separators. This was treated as a display-format choice rather than a data-quality issue because the tested values were recognized as numbers.

**Conclusion:** No data-type issues were identified in the tested columns. No values were modified during this audit.

**Status:** Completed.

---

### Step 6: Data Validity Audit

Business rules and value constraints were reviewed across the CRM dataset.

**Key findings**
- The `deal_stage` column contains only the expected categories: `Prospecting`, `Engaging`, `Won`, and `Lost`.
- No negative values were identified in `close_value`.
- All `sales_price` values were positive.
- No zero or negative values were identified in `employees`.
- `year_established` values were reviewed and considered plausible.
- Won and Lost opportunities had populated `close_date` values, while Prospecting and Engaging opportunities had missing closing dates.
- Won opportunities had positive `close_value` values, while Lost opportunities had `close_value = 0`.
- The date-order check found no cases where `close_date` preceded `engage_date`.
- Missing `close_value` values aligned with open opportunities.
- No suspicious extreme values were reported in the reviewed `accounts.revenue` and `accounts.employees` columns.

**Conclusion:** No major data validity issues were identified in the checks performed. Missing values and business-rule exceptions were assessed in context rather than automatically replaced or deleted.

**Status:** Completed.

---

### Step 7: Primary Keys and Relationships Audit

The key relationships between the four tables were reviewed to assess referential integrity.

**Key findings**
- `sales_pipeline.opportunity_id` contains no duplicate values and can serve as the opportunity identifier.
- All populated `sales_pipeline.account` values were matched to `accounts.account`.
- A product-name inconsistency (`GTXPro` versus `GTX Pro`) was identified and corrected to match the canonical product name.
- All `sales_pipeline.sales_agent` values were matched to `sales_teams.sales_agent`.
- The `account`, `product`, and `sales_agent` fields in `sales_pipeline` can be used to link opportunities to the corresponding lookup tables.

**Conclusion:** The tested relationships were validated after standardizing the product name. The original source data should be preserved, and the correction documented in the cleaning workflow.

**Status:** Completed for the relationships tested.

---



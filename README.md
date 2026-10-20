# CRM Sales Pipeline Analysis

End-to-end CRM sales pipeline analysis using SQL Server and Power BI to evaluate conversion rates, win/loss performance, sales cycles, and sales team effectiveness.

## Data Quality Audit

The dataset was reviewed before importing it into SQL Server. The audit covered dataset structure, data dictionary, missing values, duplicates, data types, validity, relationships, and business logic.

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
- Sales dates were subsequently verified as valid Excel date values, displayed in day-month-year (DMY) format.

### Step 2: Data Dictionary Review

The data dictionary was reviewed to understand the meaning of each field and its role in the analysis.

**`accounts`**
- `account`: Company name
- `sector`: Industry
- `year_established`: Year the company was established
- `revenue`: Annual company revenue in millions of USD
- `employees`: Number of employees
- `office_location`: Company headquarters
- `subsidiary_of`: Parent company, where applicable

**`products`**
- `product`: Product name
- `series`: Product series
- `sales_price`: Suggested retail price

**`sales_teams`**
- `sales_agent`: Sales representative
- `manager`: Sales manager
- `regional_office`: Regional office

**`sales_pipeline`**
- `opportunity_id`: Unique opportunity identifier
- `sales_agent`: Sales representative assigned to the opportunity
- `product`: Product associated with the opportunity
- `account`: Associated company
- `deal_stage`: Current or final opportunity stage
- `engage_date`: Date the Engaging stage was initiated
- `close_date`: Date the opportunity was won or lost
- `close_value`: Revenue recorded for the deal

The documented sales stages are `Prospecting`, `Engaging`, `Won`, and `Lost`.

### Step 3: Missing Values Audit

A missing-value audit was conducted across all four tables.

| Table | Column | Missing values | Assessment |
|---|---|---:|---|
| `sales_pipeline` | `account` | 1,425 (16.19%) | Occurs in Prospecting and Engaging; account assignment requires consideration |
| `sales_pipeline` | `engage_date` | 500 (5.68%) | Occurs only in Prospecting; consistent with the stage definition |
| `sales_pipeline` | `close_date` | 2,089 (23.74%) | Occurs in open opportunities |
| `sales_pipeline` | `close_value` | 2,089 (23.74%) | Occurs in open opportunities |
| `accounts` | `subsidiary_of` | 70 (82.35%) | May indicate companies without a parent company |
| All other audited columns | — | 0 | No missing values detected |

**Key findings**
- Missing `close_date` and `close_value` values align with opportunities that have not reached a final outcome.
- Missing `engage_date` values occur only in the Prospecting stage.
- Missing account assignments require further consideration before account-level analysis.
- Missing `subsidiary_of` values may be legitimate, but the reason for each missing value has not been independently confirmed.
- No missing values were found in the `products` table or any of the three columns in `sales_teams`.

**Data handling decision:** Missing values were not automatically replaced with zeros or deleted. Their treatment will depend on the requirements of each analysis.

### Step 4: Duplicate Records Audit

Duplicate checks were performed to identify repeated key values and fully duplicated records.

**Key findings**
- No duplicate values were found in `sales_pipeline.opportunity_id`.
- No duplicate account names were found in `accounts.account`.
- No duplicate product names were found in `products.product`.
- No duplicate sales agent names were found in `sales_teams.sales_agent`.
- No fully duplicated rows were identified in the `sales_pipeline` and `accounts` tables.

Repeated values in non-key columns, such as `deal_stage`, `product`, and `sales_agent` in the sales pipeline, are expected because multiple opportunities can share the same stage, product, or sales representative.

**Conclusion:** No duplicate-record issues were identified in the checks performed. No records were removed during this audit.

### Step 5: Data Types and Formats Audit

The data types and display formats of key columns were reviewed in Excel.

**Key findings**
- Sales date values were recognized as valid Excel dates and displayed in DMY format.
- `close_value`, `sales_price`, `revenue`, and `employees` were recognized as numeric values using Excel's `ISNUMBER()` function.
- `year_established` values were preserved correctly as years and retained as text to avoid unintended date conversions.
- Numeric values were displayed without currency symbols, with periods used as thousands separators. The tested values were recognized as numbers, so the display format did not prevent numeric calculations.

**Conclusion:** No data-type issues were identified in the tested values. The appropriate SQL Server data types will be assigned during import.

### Step 6: Data Validity Audit

Business rules and value constraints were reviewed across the CRM dataset.

**Key findings**
- The `deal_stage` column contains only the expected categories: `Prospecting`, `Engaging`, `Won`, and `Lost`.
- No negative values were identified in `close_value`.
- All `sales_price` values were positive.
- No zero or negative values were identified in `employees`.
- `year_established` values were reviewed and considered plausible.
- `revenue` and `employees` were reviewed for suspicious extreme values; none were reported.
- No cases were identified where `close_date` preceded `engage_date`.

**Conclusion:** No major data validity issues were identified in the checks performed.

### Step 7: Primary Keys and Relationships Audit

The key fields and relationships between the tables were reviewed to assess referential integrity.

**Key findings**
- `sales_pipeline.opportunity_id` contains no duplicate values and can serve as the opportunity identifier.
- All populated `sales_pipeline.account` values were matched to `accounts.account`.
- A product-name inconsistency (`GTXPro` versus `GTX Pro`) was identified and standardized to match the product name in the `products` table.
- All `sales_pipeline.sales_agent` values were matched to `sales_teams.sales_agent`.

The expected relationships are:

| Relationship | Expected cardinality |
|---|---|
| `accounts.account` → `sales_pipeline.account` | One-to-many |
| `products.product` → `sales_pipeline.product` | One-to-many |
| `sales_teams.sales_agent` → `sales_pipeline.sales_agent` | One-to-many |

**Conclusion:** The tested relationships were validated after standardizing the product name. The original source data should be preserved, and the correction documented in the cleaning workflow.


### Step 8: Business Logic Validation

Business rules were reviewed to confirm that opportunity stages, closing dates, and deal values were logically consistent.

**Key findings**
- `Prospecting` and `Engaging` opportunities have missing `close_date` and `close_value` values.
- `Won` opportunities have a populated `close_date` and a positive `close_value`.
- `Lost` opportunities have a populated `close_date` and a `close_value` of zero.
- The reviewed date-order check found no cases where `close_date` preceded `engage_date`.
- `close_value` does not have to equal `products.sales_price`. The former represents recorded deal revenue, while the latter represents the suggested retail price. Differences between these values are not automatically data quality issues.

**Conclusion:** The reviewed values were consistent with the documented sales pipeline logic. Missing deal values were not arbitrarily replaced.

### Overall Audit Conclusion

The preliminary data quality audit identified a small number of items requiring explicit handling, including missing account assignments, missing values associated with open opportunities, and a product-name inconsistency.

The tested identifiers, data types, date relationships, and business rules were generally consistent with the expected structure of the CRM sales dataset.

The data is ready to proceed to the SQL Server import stage, with the identified cleaning decisions documented and the source files preserved for traceability.

---

Task 1. Calculate the number of opportunities in each deal_stage.

```sql
select deal_stage,
count(opportunity_id) as [Opportunity Count]
from sales_pipeline
group by deal_stage
order by [Opportunity Count] desc
```

Task 2. Calculate the total number of opportunities in the dataset.

```sql
select count(opportunity_id) as [Total Opportunity]
from sales_pipeline
```

Task 3. Calculate the number and percentage of open opportunities (Prospecting, Engaging) versus closed opportunities (Won, Lost).
```sql
with CategorizedPipeline as (
select
opportunity_id,
case 
when deal_stage in ('Prospecting', 'Engaging') then 'Open Opportunities'
when deal_stage in ('Won', 'Lost') then 'Closed Opportunities'
end as opportunity_status
from sales_pipeline
)
select 
opportunity_status,
count(opportunity_id) as opportunity_count,
cast(100.0 * count(opportunity_id) / SUM(COUNT(opportunity_id)) over() as decimal(5,2)) as percentage
from CategorizedPipeline
group by opportunity_status
```

Task 4. Calculate the total number of won deals and lost deals.

```sql
select deal_stage,
count(opportunity_id) as [Opportunity Count]
from sales_pipeline
where deal_stage in ('won','lost')
group by deal_stage
```

Task 5. For each deal_stage, calculate the opportunity count, total close_value, and average close_value. Interpret the results carefully for open stages.

```sql
select deal_stage,
count(opportunity_id) as [Opportunity Count],
sum(close_value) as [Total Close Value],
avg(close_value) as [Average Close Value]
from sales_pipeline
group by deal_stage
order by [Opportunity Count] desc
```

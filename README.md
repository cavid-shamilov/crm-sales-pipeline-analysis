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



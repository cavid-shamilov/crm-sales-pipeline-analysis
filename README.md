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

Task 6. Calculate the win rate using only closed opportunities (Won and Lost) as the denominator.

```sql
select 
cast(round(sum(case when deal_stage = 'Won' then 1 else 0 end)*100.0/count(opportunity_id), 2) as decimal(10,2)) as [Overall Win Rate]
from sales_pipeline 
where deal_stage IN ('Won', 'Lost')
```
Task 7. Calculate the loss rate using only closed opportunities as the denominator.

```sql
select 
cast(round(sum(case when deal_stage = 'lost' then 1 else 0 end)*100.0/count(opportunity_id), 2) as decimal(10,2)) as [Overall Lost Rate]
from sales_pipeline 
where deal_stage IN ('Won', 'Lost')
```

Task 8. Calculate the ratio of won opportunities to lost opportunities.

```sql
select 
cast(
round(
sum(case when deal_stage = 'won' then 1 else 0 end)*1.0/
sum(case when deal_stage = 'lost' then 1 else 0 end), 2) as decimal(10,2)) as [Won-to-Lost Ratio]
from sales_pipeline 
where deal_stage in ('lost','won')
```
Task 9. Calculate the win rate for each product, using only closed opportunities for that product.

```sql
select product,
cast(round(sum(case when deal_stage = 'Won' then 1 else 0 end)*100.0/count(opportunity_id), 2) as decimal(10,2)) as [Win Rate],
sum(case when deal_stage = 'Won' then 1 else 0 end) as [Won Opportunity Count],
sum(case when deal_stage = 'Lost' then 1 else 0 end) as [Lost Opportunity Count]
from sales_pipeline 
where deal_stage IN ('Won', 'Lost')
group by product
```

Task 10. Calculate each sales agent's win rate and closed opportunity count. Identify the agents with the highest and lowest win rates, while considering sample size.

```sql
select sales_agent,
cast(round(sum(case when deal_stage = 'Won' then 1 else 0 end)*100.0/count(opportunity_id), 2) as decimal(10,2)) as [Win Rate],
count(opportunity_id) as [Closed Opportunity Count],
sum(case when deal_stage = 'Won' then 1 else 0 end) as [Won Opportunity Count],
sum(case when deal_stage = 'Lost' then 1 else 0 end) as [Lost Opportunity Count]
from sales_pipeline 
where deal_stage IN ('Won', 'Lost')
group by sales_agent
order by [Win Rate] desc
```

Task 11. Calculate total revenue from Won opportunities only.

```sql
select sum(close_value) as [Total Won Revenue]
from sales_pipeline
where deal_stage='Won'
```

Task 12. Calculate the average close_value for won opportunities.

```sql
select avg(close_value) as average_close_value 
from sales_pipeline
where deal_stage='Won'
```

Task 13. Identify the top 10 won opportunities by close_value. Include the opportunity ID, sales agent, product, account, and close value.

```sql
select top 10 opportunity_id,
sales_agent,
product,
account
close_value
from sales_pipeline	
where deal_stage='Won'
order by close_value desc
```

Task 14. Calculate each product's won revenue and its percentage contribution to total won revenue.

```sql
select product,
sum(close_value) as [Won Revenue],
sum(sum(close_value)) over () as [Total Won Revenue],
cast(round(sum(close_value)*100.0/sum(sum(close_value)) over (), 2) as decimal(10,2)) as [Revenue Contribution Percentage]
from sales_pipeline
where deal_stage='Won'
group by product
```

Task 15. Identify the top five products by won revenue and calculate their combined percentage of total won revenue.

```sql
with ProductRevenue as (
select 
product, 
sum(close_value) as revenue
from sales_pipeline
where deal_stage = 'Won'
group by product
),
TotalRevenue as (
    select SUM(revenue) as total_rev FROM ProductRevenue
),
Top5Products as (
select TOP 5 product, revenue
from ProductRevenue
order by  revenue desc
)
select
t.product,
t.revenue,
CAST(ROUND(t.revenue * 100.0 / tr.total_rev, 2) AS DECIMAL(5,2)) AS [Individual Percentage],
(select CAST(ROUND(SUM(revenue) * 100.0 / tr.total_rev, 2) AS DECIMAL(5,2)) FROM Top5Products) AS [Top 5 Combined Percentage]
from Top5Products t
cross join TotalRevenue tr;
```

Task 16: Calculate the number of opportunities by year of engage_date.

```sql
select year(engage_date) as [Engage Year],
count(opportunity_id) as [Count Opportunity]		
from sales_pipeline
group by year(engage_date)
```

Task 17. Calculate total won revenue by year of close_date.

```sql
select year(close_date) as [Close Year],
sum(close_value) as [Won Revenue]	
from sales_pipeline
where deal_stage='Won'
group by year(close_date)
```

Task 18. Calculate won revenue by year and month of close_date, including the year and month number so the results can be sorted chronologically.

```sql
select year(close_date) as [Close Year],
month(close_date) as [Close Month Number],
datename(month,close_date) as [Close Month],
sum(close_value) as [Won Revenue]	
from sales_pipeline
where deal_stage='Won'
group by year(close_date), month(close_date), datename(month,close_date)
order by year(close_date), month(close_date), datename(month,close_date)
```

Task 19. Calculate monthly won revenue and month-over-month (MoM) percentage growth. Use LAG() and handle months with no recorded won revenue appropriately.

```sql
WITH MonthBounds AS (
    SELECT
        DATEFROMPARTS(
            YEAR(MIN(close_date)),
            MONTH(MIN(close_date)),
            1
        ) AS StartMonth,
        DATEFROMPARTS(
            YEAR(MAX(close_date)),
            MONTH(MAX(close_date)),
            1
        ) AS EndMonth
    FROM dbo.sales_pipeline
    WHERE deal_stage = 'Won'
      AND close_date IS NOT NULL
),
MonthCalendar AS (
    SELECT StartMonth AS MonthStart
    FROM MonthBounds
    WHERE StartMonth IS NOT NULL

    UNION ALL

    SELECT DATEADD(MONTH, 1, mc.MonthStart)
    FROM MonthCalendar mc
    CROSS JOIN MonthBounds mb
    WHERE mc.MonthStart < mb.EndMonth
),
MonthlyRevenue AS (
    SELECT
        DATEFROMPARTS(
            YEAR(close_date),
            MONTH(close_date),
            1
        ) AS MonthStart,
        SUM(close_value) AS Current_Revenue
    FROM dbo.sales_pipeline
    WHERE deal_stage = 'Won'
      AND close_date IS NOT NULL
    GROUP BY
        YEAR(close_date),
        MONTH(close_date)
),
RevenueFilled AS (
    SELECT
        mc.MonthStart,
        COALESCE(mr.Current_Revenue, 0) AS Current_Revenue
    FROM MonthCalendar mc
    LEFT JOIN MonthlyRevenue mr
        ON mc.MonthStart = mr.MonthStart
),
RevenueWithLag AS (
    SELECT
        MonthStart,
        Current_Revenue,
        LAG(Current_Revenue) OVER (
            ORDER BY MonthStart
        ) AS Previous_Revenue
    FROM RevenueFilled
)
SELECT
    YEAR(MonthStart) AS [Year],
    MONTH(MonthStart) AS [Month],
    Current_Revenue,
    Previous_Revenue,
    CASE
        WHEN Previous_Revenue IS NULL
          OR Previous_Revenue = 0
        THEN NULL
        ELSE CAST(
            ROUND(
                (Current_Revenue - Previous_Revenue)
                * 100.0 / Previous_Revenue,
                2
            ) AS DECIMAL(10,2)
        )
    END AS [MoM Growth %]
FROM RevenueWithLag
ORDER BY MonthStart
OPTION (MAXRECURSION 0);
```

Task 20. Calculate the monthly win rate based on opportunities closed in each month. Use won and lost opportunities only.

```sql
WITH MonthlyPerformance AS (
    SELECT 
        YEAR(close_date) AS [Year],
        MONTH(close_date) AS [Month],
        COUNT(opportunity_id) AS [Total Closed],
        SUM(CASE WHEN deal_stage = 'Won' THEN 1 ELSE 0 END) AS [Won Count]
    FROM sales_pipeline
    WHERE deal_stage IN ('Won', 'Lost') AND close_date IS NOT NULL
    GROUP BY YEAR(close_date), MONTH(close_date)
)
SELECT 
    [Year],
    [Month],
    [Total Closed],
    [Won Count],
    CAST(ROUND([Won Count] * 100.0 / [Total Closed], 2) AS DECIMAL(10,2)) AS [Monthly Win Rate %]
FROM MonthlyPerformance
ORDER BY [Year], [Month];
```

Task 21. Calculate the total number of opportunities for each product.

```sql
select product,
count(opportunity_id) as [Opportunity Count]
from sales_pipeline
group by product
```

Task 22. For each product, calculate the counts of won, lost, and open opportunities.

```sql
select product,
sum(case when deal_stage in ('Prospecting', 'Engaging') then 1 else 0 end) as [Open Opportunities],
sum(case when deal_stage='Won' then 1 else 0 end) as [Won Opportunity Count],
sum(case when deal_stage='Lost' then 1 else 0 end) as [Lost Opportunity Count]
from sales_pipeline
group by product
```

Task 23. Rank products by win rate, considering only closed opportunities. Include each product's closed opportunity count.

```sql
with Opportunity_Count as(
select product,
count(opportunity_id) as [Closed Opportunity Count],
sum(case when deal_stage='Won' then 1 else 0 end) as [Won Opportunity Count],
sum(case when deal_stage='Lost' then 1 else 0 end) as [Lost Opportunity Count]
from sales_pipeline
where deal_stage in ('Won','lost')
group by product
)
select product,
[Closed Opportunity Count],
cast(round([Won Opportunity Count]*100.0/[Closed Opportunity Count], 2) as decimal(10,2)) as Win_Rate,
cast(round([Lost Opportunity Count]*100.0/[Closed Opportunity Count], 2) as decimal(10,2)) as Lost_Rate,
rank() over ( order by [Won Opportunity Count]*100.0/[Closed Opportunity Count] desc) as Product_Rank
from Opportunity_Count
```

Task 24. Calculate the average close_value for won opportunities for each product.

```sql
select product,
avg(close_value) as [ Average Close Value ]
from sales_pipeline
where deal_stage='Won'
group by product
order by [Average Close Value] desc
```

Task 25. Compare each product's sales_price with its average won deal value. Calculate the absolute difference and percentage difference, and explain why the two figures may differ.

```sql
select sp.product,
sales_price,
avg(close_value) as [ Average Close Value ],
abs(avg(close_value) - p.sales_price) as [Absolute Difference],
cast(round((avg(close_value)-sales_price) * 100.0/NULLIF(p.sales_price, 0), 2) as decimal(10,2)) as Percentage_Difference
from sales_pipeline as sp
inner join products as p
on sp.product=p.product
where deal_stage='Won'
group by sp.product,sales_price
order by Percentage_Difference desc
```

Task 26. Calculate each sales agent's total opportunity count and rank agents by volume.

```sql
select sales_agent,
count(opportunity_id) as [Opportunity Count] ,
rank() over(order by count(opportunity_id) desc) as [Sales Agent Rank]
from sales_pipeline
group by sales_agent
```

Task 27. Calculate each agent's total won revenue and rank them from highest to lowest.

```sql
select sales_agent,
sum(close_value) as [Total Won Revenue]
from sales_pipeline
where deal_stage='Won'
group by sales_agent
order by [Total Won Revenue] desc
```

Task 28. Calculate each agent's average won deal value and include their number of won deals.

```sql
select sales_agent,
avg(close_value) as [Average Won Revenue],
count(opportunity_id) as [Won Deals]
from sales_pipeline
where deal_stage='Won'
group by sales_agent
order by  [Average Won Revenue] desc
```

Task 29. Create one summary per sales agent containing total opportunities, closed opportunities, won deals, lost deals, win rate, and won revenue.

``` sql
select sales_agent,
count(opportunity_id) as [Total Opportunities],
sum(case when deal_stage='won' then close_value else 0 end) as [Won Revenue],
sum(case when deal_stage='Won' then 1 else 0 end) as [Won Deals],
sum(case when deal_stage='Lost' then 1 else 0 end) as [Lost Deals],
sum(case when deal_stage in ('lost', 'won') then 1 else 0 end) as [Closed Opportunities],
cast(round(sum(case when deal_stage='Won' then 1.0 else 0 end)* 100.0/nullif(sum(case when deal_stage in ('lost', 'won') then 1 else 0 end), 0) , 2) as decimal(10,2)) as [Win Rate]
from sales_pipeline
group by sales_agent
order by [Win Rate] desc
```

Task 30. Use a window function to identify the top three sales agents in each regional_office by won revenue.

```sql
with AgentRevenue as (
select
st.regional_office,
sp.sales_agent,
SUM(sp.close_value) as [Total Won Revenue]
from sales_pipeline as sp
inner join sales_teams as st
on sp.sales_agent = st.sales_agent
where sp.deal_stage = 'Won'
group by st.regional_office,
sp.sales_agent
),
RankedAgents as (
select
regional_office,
sales_agent,
[Total Won Revenue],
rank() over (partition by regional_office order by [Total Won Revenue] desc) as [Sales Agent Rank]
from AgentRevenue
)
select
regional_office,
sales_agent,
[Total Won Revenue],
[Sales Agent Rank]
from RankedAgents
where [Sales Agent Rank] <= 3
order by regional_office,
[Sales Agent Rank]
```

Task 31. Calculate total opportunities by regional_office

```sql
select regional_office,
count (opportunity_id) as [Total Opportunities]
from sales_pipeline as sp
inner join sales_teams as st
on sp.sales_agent=st.sales_agent
group by regional_office
```

Task 32. Calculate total won revenue by regional office and rank the offices.

```sql
with OfficeRevenue as (
select
st.regional_office,
sum(sp.close_value) as [Total Won Revenue]
from sales_pipeline as sp
inner join sales_teams as st
on sp.sales_agent = st.sales_agent
where sp.deal_stage = 'Won'
group by st.regional_office
)

select
regional_office,
[Total Won Revenue],
rank() over ( order by [Total Won Revenue] desc) as [Regional Office Rank]
from OfficeRevenue
```

Task 33. Calculate each regional office's win rate using closed opportunities only.

```sql
select regional_office ,
cast(round(sum(case when deal_stage = 'Won' then 1 else 0 end)*100.0/count(opportunity_id), 2) as decimal(10,2)) as [Win Rate] 
from sales_pipeline as sp
inner join sales_teams as st
on sp.sales_agent=st.sales_agent
where deal_stage IN ('Won', 'Lost')
group by regional_office
order by [Win Rate] desc
```

Task 34. Calculate opportunity count, won deals, win rate, and won revenue for each manager

```sql
select manager,
count(opportunity_id) as [Total Opportunities],
sum(case when deal_stage='Won' then 1 else 0 end) as [Won Deals],
cast(round(sum(case when deal_stage='Won' then 1.0 else 0 end)* 100.0/nullif(sum(case when deal_stage in ('lost', 'won') then 1 else 0 end), 0) , 2) as decimal(10,2)) as [Win Rate],
sum(case when deal_stage='won' then close_value else 0 end) as [Won Revenue]
from sales_pipeline as sp
inner join sales_teams as st
on sp.sales_agent=st.sales_agent
group by manager
order by [Win Rate] desc
```

Task 35. Compare regional offices using total opportunities, closed opportunities, win rate, won revenue, and average won deal value. Identify the strongest-performing office and support your conclusion with metrics.

```sql
 select regional_office,
count(opportunity_id) as [Total Opportunities],
sum(case when deal_stage in ('lost', 'won') then 1 else 0 end) as [Closed Opportunities],
cast(round(sum(case when deal_stage='Won' then 1.0 else 0 end)* 100.0/nullif(sum(case when deal_stage in ('lost', 'won') then 1 else 0 end), 0) , 2) as decimal(10,2)) as [Win Rate],
sum(case when deal_stage='won' then close_value else 0 end) as [Won Revenue],
avg(case when sp.deal_stage = 'Won' then cast(sp.close_value as decimal(10,2)) end) as [Average Won Deal Value]
from sales_pipeline as sp
inner join sales_teams as st
on sp.sales_agent=st.sales_agent
group by regional_office
order by [Win Rate] desc
```

Task 36. Calculate the number of opportunities associated with each account. Exclude NULL accounts from the account-level grouping.

```sql
select account,
count(opportunity_id) as [Opportunity Count]
from sales_pipeline
where account is not null
group by account
```

Task 37. Calculate won revenue by account and identify the top 10 accounts.

```sql
select top 10 account,
sum(case when deal_stage='won' then close_value else 0 end) as [Won Revenue]
from sales_pipeline
where account is not null
group by account
order by [Won Revenue] desc
```

Task 38. Join sales_pipeline with accounts and calculate opportunity count, won revenue, and win rate by sector.

```sql
select sector,
count(opportunity_id) as [Total Opportunities],
sum(case when deal_stage='won' then close_value else 0 end) as [Won Revenue],
cast(round(sum(case when deal_stage='Won' then 1.0 else 0 end)* 100.0/nullif(sum(case when deal_stage in ('lost', 'won') then 1 else 0 end), 0) , 2) as decimal(10,2)) as [Win Rate]
from sales_pipeline as sp
inner join accounts as a
on a.account=sp.account
group by sector
order by [Win Rate] desc
```

Task 39. Compare each account's annual revenue with its recorded won deal revenue. Explain why these figures should not be treated as equivalent measures.

```sql
SELECT 
    a.account AS [Account Name],
    a.revenue AS [Account Annual Revenue],
    ISNULL(SUM(CASE WHEN s.deal_stage = 'Won' THEN s.close_value END), 0) AS [Total Won Deal Revenue] 
FROM accounts a
LEFT JOIN sales_pipeline s ON a.account = s.account
GROUP BY a.account, a.revenue
ORDER BY [Account Annual Revenue] DESC;
```

Task 40. Analyze opportunities and won revenue for accounts with a populated subsidiary_of field versus accounts without one. State the assumption you make about blank parent-company values.

```sql
SELECT 
    CASE 
        WHEN a.subsidiary_of IS NULL OR LTRIM(RTRIM(a.subsidiary_of)) = '' THEN 'Independent / Standalone'
        ELSE 'Subsidiary'
    END AS [Account Type],
    COUNT(s.opportunity_id) AS [Total Opportunities],
    SUM(CASE WHEN s.deal_stage = 'Won' THEN 1 ELSE 0 END) AS [Won Opportunities Count],
    ISNULL(SUM(CASE WHEN s.deal_stage = 'Won' THEN s.close_value END), 0) AS [Total Won Revenue],
    CAST(ROUND(ISNULL(AVG(CASE WHEN s.deal_stage = 'Won' THEN CAST(s.close_value AS FLOAT) END), 0), 2) AS DECIMAL(10,2)) AS [Avg Won Deal Value]
FROM accounts a
LEFT JOIN sales_pipeline s ON a.account = s.account
GROUP BY 
    CASE 
        WHEN a.subsidiary_of IS NULL OR LTRIM(RTRIM(a.subsidiary_of)) = '' THEN 'Independent / Standalone'
        ELSE 'Subsidiary'
    END
```

Task 41. Calculate the average number of days between engage_date and close_date for won opportunities.

```sql

```

Task 41. Calculate the average number of days between engage_date and close_date for won opportunities.

```sql
select
avg(DATEDIFF(dd,engage_date,close_date)*1.0) as [Average Number Of Days],
count(opportunity_id) as [Won Deals Count]
from sales_pipeline
where deal_stage='Won'
and engage_date is not null
and close_date is not null
```

Task 42. Calculate the average sales cycle for won opportunities for each product.

```sql
select PRODUCT,
count(opportunity_id) as [Won Deals Count],
avg(DATEDIFF(dd,engage_date,close_date)*1.0) as [Average Number Of Days]
from sales_pipeline
where deal_stage='Won'
and engage_date is not null
and close_date is not null
group by product
order by [Average Number Of Days] asc
```

Task 43. Calculate each agent's average sales cycle for won opportunities and include the number of won deals.

```sql
select sales_agent,
count(opportunity_id) as [Won Deals Count],
avg(DATEDIFF(dd,engage_date,close_date)*1.0) as [Average Number Of Days]
from sales_pipeline
where deal_stage='Won'
and engage_date is not null
and close_date is not null
group by sales_agent
order by [Average Number Of Days] asc
```

Task 44. Calculate average and maximum sales cycle duration for won opportunities by regional office.

```sql
select regional_office,
count(opportunity_id) as [Won Deals Count],
max(DATEDIFF(dd,engage_date,close_date)) as [Maximum Sales Cycle Duration],
avg(DATEDIFF(dd,engage_date,close_date)*1.0) as [Average Number Of Days]
from sales_pipeline as sp
inner join sales_teams as st
on sp.sales_agent=st.sales_agent
where deal_stage='Won'
and engage_date is not null
and close_date is not null
group by regional_office
order by [Average Number Of Days]
```

Task 45. Compare average sales cycle duration for won and lost opportunities. Explain why a difference in averages does not necessarily prove that a longer sales cycle causes a win or loss.

```sql
SELECT 
    -- Won fürsətlərin orta satış müddəti
    (SELECT AVG(DATEDIFF(day, engage_date, close_date) * 1.0) 
     FROM sales_pipeline 
     WHERE deal_stage = 'Won' AND engage_date IS NOT NULL AND close_date IS NOT NULL) AS [Avg Won Sales Cycle (Days)],
    
    -- Lost fürsətlərin orta satış müddəti
    (SELECT AVG(DATEDIFF(day, engage_date, close_date) * 1.0) 
     FROM sales_pipeline 
     WHERE deal_stage = 'Lost' AND engage_date IS NOT NULL AND close_date IS NOT NULL) AS [Avg Lost Sales Cycle (Days)],
    
    -- İki status arasındakı fərq
    CAST(ROUND(
        (SELECT AVG(DATEDIFF(day, engage_date, close_date) * 1.0) 
         FROM sales_pipeline 
         WHERE deal_stage = 'Won' AND engage_date IS NOT NULL AND close_date IS NOT NULL) 
        - 
        (SELECT AVG(DATEDIFF(day, engage_date, close_date) * 1.0) 
         FROM sales_pipeline 
         WHERE deal_stage = 'Lost' AND engage_date IS NOT NULL AND close_date IS NOT NULL), 2) AS DECIMAL(10,2)) AS [Difference (Days)];
```

Task 46. Calculate cumulative won revenue over time by closing month using a window function.

```sql
WITH MonthlyWonRevenue AS (
    -- 1. Hər ayın ümumi Won gəlirini hesablayırıq
    SELECT 
        DATEFROMPARTS(YEAR(close_date), MONTH(close_date), 1) AS [MonthStart],
        SUM(close_value) AS [MonthlyRevenue]
    FROM sales_pipeline
    WHERE deal_stage = 'Won' 
      AND close_date IS NOT NULL 
      AND close_value IS NOT NULL
    GROUP BY YEAR(close_date), MONTH(close_date)
)
-- 2. Aylıq gəlirləri zaman oxu üzrə üst-üstə toplayırıq (Cumulative Won Revenue)
SELECT 
    YEAR([MonthStart]) AS [Year],
    MONTH([MonthStart]) AS [Month],
    [MonthlyRevenue],
    SUM([MonthlyRevenue]) OVER (ORDER BY [MonthStart]  ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS [Cumulative Won Revenue]
FROM MonthlyWonRevenue
ORDER BY [MonthStart] ASC;
```

Task 47. Rank products by won revenue and calculate their cumulative percentage contribution to total won revenue. Identify how many products account for at least 80% of won revenue.

```sql
WITH ProductRevenue AS (
    -- 1. Hər məhsulun ümumi Won gəlirini hesablayırıq
    SELECT 
        product,
        SUM(close_value) AS Product_Revenue
    FROM sales_pipeline
    WHERE deal_stage = 'Won'
      AND close_value IS NOT NULL
    GROUP BY product
),
TotalRevenue AS (
    -- 2. Bütün məhsulların ümumi Won gəlirini (Grand Total) tapırıq
    SELECT SUM(Product_Revenue) AS Grand_Total FROM ProductRevenue
),
RankedProducts AS (
    -- 3. Məhsulları dərəcələndiririk və kumulyativ cəmi / faizi hesablayırıq
    SELECT 
        pr.product,
        pr.Product_Revenue,
        DENSE_RANK() OVER (ORDER BY pr.Product_Revenue DESC) AS [Product Rank],
        SUM(pr.Product_Revenue) OVER (ORDER BY pr.Product_Revenue DESC ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS Cumulative_Revenue,
        CAST(ROUND(
            SUM(pr.Product_Revenue) OVER (ORDER BY pr.Product_Revenue DESC) * 100.0 / tr.Grand_Total, 
            2
        ) AS DECIMAL(5,2)) AS [Cumulative Percentage %]
    FROM ProductRevenue pr
    CROSS JOIN TotalRevenue tr
)
-- 4. Bütün dərəcələnmiş məhsulları və 80% limitini keçən statusu çıxarırıq
SELECT 
    [Product Rank],
    product,
    Product_Revenue,
    Cumulative_Revenue,
    [Cumulative Percentage %],
    CASE 
        WHEN [Cumulative Percentage %] <= 80.00 
             OR [Product Rank] = (
                 SELECT MIN([Product Rank]) 
                 FROM RankedProducts 
                 WHERE [Cumulative Percentage %] >= 80.00
             )
        THEN 'Included in Top 80%'
        ELSE 'Other'
    END AS [Pareto Status]
FROM RankedProducts
ORDER BY [Product Rank] ASC;
```

Task 48. Compare each sales agent's win rate with the average win rate of their regional office. Show the difference in percentage points.

```sql
WITH AgentWinRate AS (
    -- 1. Hər agentin öz Win Rate-ini və regionunu tapırıq
    SELECT 
        sp.sales_agent,
        st.regional_office,
        COUNT(sp.opportunity_id) AS Agent_Total_Deals,
        CAST(ROUND(SUM(CASE WHEN sp.deal_stage = 'Won' THEN 1 ELSE 0 END) * 100.0 / COUNT(sp.opportunity_id), 2) AS DECIMAL(5,2)) AS Agent_Win_Rate
    FROM sales_pipeline sp
    JOIN sales_teams st ON sp.sales_agent = st.sales_agent
    WHERE sp.deal_stage IN ('Won', 'Lost')
    GROUP BY sp.sales_agent, st.regional_office
),
RegionalWinRate AS (
    -- 2. Hər regionun ortalama Win Rate-ini tapırıq
    SELECT 
        st.regional_office,
        COUNT(sp.opportunity_id) AS Region_Total_Deals,
        CAST(ROUND(SUM(CASE WHEN sp.deal_stage = 'Won' THEN 1 ELSE 0 END) * 100.0 / COUNT(sp.opportunity_id), 2) AS DECIMAL(5,2)) AS Region_Win_Rate
    FROM sales_pipeline sp
    JOIN sales_teams st ON sp.sales_agent = st.sales_agent
    WHERE sp.deal_stage IN ('Won', 'Lost')
    GROUP BY st.regional_office
)
-- 3. Agentin nəticəsi ilə Regionun nəticəsini müqayisə edirik
SELECT 
    a.sales_agent,
    a.regional_office,
    a.Agent_Win_Rate AS [Agent Win Rate %],
    r.Region_Win_Rate AS [Region Win Rate %],
    -- Faiz bəndi fərqi (Percentage Points Difference)
    CAST(ROUND(a.Agent_Win_Rate - r.Region_Win_Rate, 2) AS DECIMAL(5,2)) AS [Difference (Percentage Points)]
FROM AgentWinRate a
JOIN RegionalWinRate r ON a.regional_office = r.regional_office
ORDER BY [Difference (Percentage Points)] DESC
```

Task 49. Create a single summary that reports the count and percentage of opportunities in each stage, total won revenue, overall closed-deal win rate, and average won deal value

```sql
WITH GlobalMetrics AS (
    -- 1. Bütün ümumi KPI-ları (ümumi qazanılan gəlir, orta dəyər və win rate) hesablayırıq
    SELECT 
        SUM(CASE WHEN deal_stage = 'Won' THEN close_value ELSE 0 END) AS Total_Won_Revenue,
        CAST(ROUND(AVG(CASE WHEN deal_stage = 'Won' THEN CAST(close_value AS FLOAT) END), 2) AS DECIMAL(10,2)) AS Avg_Won_Deal_Value,
        CAST(ROUND(SUM(CASE WHEN deal_stage = 'Won' THEN 1 ELSE 0 END) * 100.0 / 
            NULLIF(SUM(CASE WHEN deal_stage IN ('Won', 'Lost') THEN 1 ELSE 0 END), 0), 2) AS DECIMAL(5,2)) AS Overall_Win_Rate_Percent
    FROM sales_pipeline
),
StageMetrics AS (
    -- 2. Hər mərhələ üzrə fürsət sayını və ümumi faiz payını hesablayırıq
    SELECT 
        deal_stage,
        COUNT(opportunity_id) AS Stage_Count,
        CAST(ROUND(COUNT(opportunity_id) * 100.0 / (SELECT COUNT(*) FROM sales_pipeline), 2) AS DECIMAL(5,2)) AS Stage_Percentage
    FROM sales_pipeline
    GROUP BY deal_stage
)
-- 3. Mərhələ bölgüsünü qlobal KPI-lar ilə birləşdiririk
SELECT 
    sm.deal_stage AS [Deal Stage],
    sm.Stage_Count AS [Opportunities Count],
    sm.Stage_Percentage AS [Stage Percentage %],
    gm.Total_Won_Revenue AS [Total Won Revenue],
    gm.Avg_Won_Deal_Value AS [Avg Won Deal Value],
    gm.Overall_Win_Rate_Percent AS [Overall Win Rate %]
FROM StageMetrics sm
CROSS JOIN GlobalMetrics gm
ORDER BY sm.Stage_Count DESC;
```

Task 50. Executive Business Recommendations

## 10. Business Findings & Recommendations

Based on the analysis of 8,800 CRM sales opportunities, several important patterns were identified across sales performance, product revenue, regional offices, sales representatives, customer accounts, and sales cycle duration.

### 10.1. Overall Sales Performance

The dataset contains 8,800 opportunities, of which 6,711 (76.26%) are closed and 2,089 (23.74%) remain open.

| KPI | Result |
|---|---:|
| Total Opportunities | 8,800 |
| Won Opportunities | 4,238 |
| Lost Opportunities | 2,473 |
| Open Opportunities | 2,089 |
| Closed Deals | 6,711 |
| Overall Win Rate | 63.15% |
| Overall Loss Rate | 36.85% |
| Total Won Revenue | $10,005,534 |
| Average Won Deal Value | $2,360.91 |
| Won-to-Lost Ratio | 1.71 |

**Key Findings:**

- The overall win rate is 63.15%, meaning approximately 63 out of every 100 closed opportunities resulted in a successful deal.
- The company generated over $10 million in won revenue from 4,238 successful opportunities.
- Open opportunities represent 23.74% of the pipeline, indicating a substantial number of deals that have not yet reached a final outcome.

**Recommendations:**

- Monitor open opportunities regularly to identify deals that may be delayed or at risk.
- Establish consistent follow-up procedures for opportunities in the Prospecting and Engaging stages.
- Track win rate, revenue, and average deal value together rather than relying on a single performance metric.

### 10.2. Product Performance and Revenue Concentration

Product-level analysis revealed considerable differences in revenue contribution, win rates, and average deal values.

| Product | Won Revenue | Revenue Share | Win Rate |
|---|---:|---:|---:|
| GTX Pro | $3,510,578 | 35.09% | 63.56% |
| GTX Plus Pro | $2,629,651 | 26.28% | 64.30% |
| MG Advanced | $2,216,387 | 22.15% | 60.33% |
| GTX Plus Basic | $705,275 | 7.05% | 62.13% |
| GTX Basic | $499,263 | 4.99% | 63.72% |
| GTK 500 | $400,612 | 4.00% | 60.00% |
| MG Special | $43,768 | 0.44% | 64.84% |

**Key Findings:**

- GTX Pro is the largest revenue contributor, accounting for 35.09% of total won revenue.
- GTX Plus Pro contributes another 26.28%.
- The top three products (GTX Pro, GTX Plus Pro, and MG Advanced) generate 83.52% of total won revenue.
- MG Special has the highest product win rate at 64.84%, but contributes only 0.44% of total revenue.
- GTK 500 has the highest average won deal value ($26,707), but only 15 won opportunities. Its small sample size should be considered when interpreting this result.
- GTX Pro has the shortest average sales cycle among the products, at approximately 48.33 days.

**Recommendations:**

- Maintain strong sales and account-management focus on GTX Pro, GTX Plus Pro, and MG Advanced because they generate the majority of revenue.
- Investigate the reasons behind MG Special's relatively high win rate but very low revenue contribution. Its low average deal value may help explain this difference.
- Evaluate GTK 500 separately before making major strategic decisions, as its results are based on a limited number of successful deals.
- Use product-level win rate, revenue contribution, and average deal value together when evaluating product performance.
- Monitor revenue concentration to reduce excessive dependence on a small number of products.

### 10.3. Sales Representative Performance

The analysis identified substantial differences in opportunity volume, win rates, won revenue, and average deal values across the 30 sales representatives.

**Highest Win Rates:**

| Sales Representative | Win Rate | Closed Deals |
|---|---:|---:|
| Hayden Neloms | 70.39% | 152 |
| Maureen Marcano | 69.95% | 213 |
| Wilburn Farren | 69.62% | 79 |
| Cecily Lampkin | 66.88% | 160 |
| Versie Hillebrand | 66.67% | 264 |

**Highest Won Revenue:**

| Sales Representative | Won Revenue |
|---|---:|
| Darcel Schlecht | $1,153,214 |
| Vicki Laflamme | $478,396 |
| Kary Hendrixson | $454,298 |
| Cassey Cress | $450,489 |
| Donn Cantrell | $445,860 |

**Additional Findings:**

- Darcel Schlecht generated the highest won revenue ($1.15 million) and handled the largest number of opportunities (747).
- Hayden Neloms achieved the highest win rate (70.39%), demonstrating strong conversion performance among closed opportunities.
- Elease Gluck recorded the highest average won deal value among representatives, at approximately $3,614.
- Lajuana Vencill had the lowest win rate (54.98%), while Moses Frase had the longest average sales cycle among representatives with won deals (64.69 days).

**Recommendations:**

- Study the sales approaches and practices of high-performing representatives and identify methods that could be shared across the team.
- Review the sales processes of representatives with below-average win rates to identify potential improvement opportunities.
- Avoid evaluating representatives solely by revenue or win rate. Consider opportunity volume, average deal value, and sales cycle duration together.
- Provide targeted coaching and performance support based on individual results.
- Consider differences in opportunity volume when comparing representatives, since a high win rate based on a smaller number of closed deals may be less stable.

### 10.4. Regional Office Performance

Regional analysis showed differences in sales volume, revenue generation, win rates, and sales cycle duration.

| Regional Office | Opportunities | Win Rate | Won Revenue | Avg. Won Deal Value |
|---|---:|---:|---:|---:|
| West | 2,997 | 63.94% | $3,568,647 | $2,481.67 |
| Central | 3,512 | 62.56% | $3,346,293 | $2,054.20 |
| East | 2,291 | 63.02% | $3,090,594 | $2,639.28 |

**Key Findings:**

- West generated the highest won revenue and achieved the highest regional win rate.
- East had the highest average won deal value and the shortest average sales cycle (50.64 days).
- Central handled the largest number of opportunities but had the lowest win rate and average won deal value.
- Central also had the longest average sales cycle among the three regions, at approximately 52.56 days.

**Recommendations:**

- Investigate the sales practices and operational processes contributing to West's strong revenue and win-rate performance.
- Analyze East's approach to deal management and average deal value to identify practices that could be shared with other regions.
- Review Central's sales process to identify potential opportunities to improve conversion rates and deal values.
- Compare regional performance using multiple KPIs instead of relying exclusively on total revenue.

### 10.5. Manager Performance

Manager-level analysis revealed differences in team size, conversion rates, and revenue generation.

| Manager | Opportunities | Won Deals | Win Rate | Won Revenue |
|---|---:|---:|---:|---:|
| Cara Losch | 964 | 480 | 64.43% | $1,130,049 |
| Summer Sewald | 1,701 | 828 | 64.34% | $1,964,750 |
| Celia Rouche | 1,296 | 610 | 63.41% | $1,603,897 |
| Dustin Brinkmann | 1,583 | 747 | 62.98% | $1,094,363 |
| Melvin Marxen | 1,929 | 882 | 62.20% | $2,251,930 |
| Rocco Neubert | 1,327 | 691 | 62.08% | $1,960,545 |

**Key Findings:**

- Cara Losch's team achieved the highest win rate at 64.43%.
- Melvin Marxen's team generated the highest won revenue ($2.25 million) and handled the largest number of opportunities.
- Rocco Neubert's team generated nearly $1.96 million in won revenue despite having a lower win rate than most other teams.

**Recommendations:**

- Examine the practices of teams with stronger conversion rates and identify opportunities for knowledge sharing.
- Evaluate manager performance using team size, opportunity volume, win rate, and revenue together.
- Investigate why some teams generate higher revenue despite having lower conversion rates.

### 10.6. Customer Account and Industry Analysis

Customer-level analysis identified the accounts contributing the most won revenue.

**Top 10 Accounts by Won Revenue:**

| Account | Won Revenue |
|---|---:|
| Kan-code | $341,455 |
| Konex | $269,245 |
| Condax | $206,410 |
| Cheers | $198,020 |
| Hottechi | $194,957 |
| Goodsilron | $182,522 |
| Treequote | $176,751 |
| Warephase | $170,046 |
| Xx-holding | $169,357 |
| Isdom | $164,683 |

**Industry / Sector Performance:**

| Sector | Opportunities | Win Rate | Won Revenue |
|---|---:|---:|---:|
| Marketing | 674 | 64.85% | $922,321 |
| Entertainment | 451 | 64.68% | $689,007 |
| Software | 757 | 63.92% | $1,077,934 |
| Technology* | 1,165 | 63.42% | $1,515,487 |
| Services | 390 | 63.35% | $533,006 |
| Retail | 1,397 | 63.06% | $1,867,528 |
| Employment | 311 | 62.59% | $436,174 |
| Telecommunications | 501 | 62.50% | $653,574 |
| Medical | 1,051 | 62.32% | $1,359,595 |
| Finance | 678 | 61.17% | $950,908 |

*The source dataset labels this sector as "technolgy"; the table uses the corrected spelling "Technology" for readability.*

**Key Findings:**

- Kan-code generated the highest won revenue among customer accounts, with $341,455.
- Retail generated the highest sector-level won revenue ($1.87 million) and had the largest opportunity volume.
- Marketing achieved the highest sector win rate at 64.85%.
- Finance had the lowest sector win rate at 61.17%.
- The sector analysis covers 7,375 opportunities linked to known accounts. Opportunities with missing account values are excluded from this account-based analysis.

**Parent Company Analysis:**

| Account Type | Opportunities | Won Deals | Won Revenue | Avg. Won Deal Value |
|---|---:|---:|---:|---:|
| Independent / Standalone* | 6,083 | 3,500 | $8,273,453 | $2,363.84 |
| Subsidiary | 1,292 | 738 | $1,732,081 | $2,346.99 |

*Accounts with a NULL or blank parent-company field are classified as Independent / Standalone for this analysis. This classification means no parent company is recorded in the dataset; it does not independently verify legal ownership.*

**Recommendations:**

- Prioritize account-management efforts for high-revenue customers and investigate opportunities to expand successful relationships.
- Examine the reasons behind the stronger conversion rate in Marketing and the lower conversion rate in Finance.
- Develop sector-specific sales strategies based on customer needs and observed performance.
- Improve account data completeness to support more comprehensive customer and industry analysis.
- Explore parent-company relationships when planning account coverage, while avoiding assumptions about corporate structures that are not explicitly recorded.

### 10.7. Sales Cycle Analysis

The average duration between engagement and deal closure was analyzed across won opportunities, products, representatives, and regional offices.

| Metric | Average Sales Cycle |
|---|---:|
| Won Opportunities | 51.78 days |
| Lost Opportunities | 41.48 days |
| Difference | 10.30 days |

**Key Findings:**

- Won opportunities took an average of 51.78 days to close.
- Lost opportunities had a shorter average duration of 41.48 days.
- Won deals therefore took approximately 10.30 days longer on average than lost deals.
- GTX Pro had the shortest product-level average sales cycle (48.33 days).
- GTK 500 had the longest product-level average sales cycle (64.07 days), based on only 15 won deals.
- East had the shortest regional average sales cycle (50.64 days), while Central had the longest (52.56 days).

**Recommendations:**

- Monitor sales cycle duration by product, representative, and region to identify potential delays.
- Investigate whether longer sales cycles are associated with larger or more complex deals.
- Review opportunities that remain open for extended periods and establish appropriate follow-up procedures.
- Do not assume that shorter sales cycles automatically indicate better performance. Evaluate sales duration alongside win rate and deal value.

**Important Interpretation Note:**

The difference between won and lost sales cycle averages does not establish a causal relationship. The analysis identifies an observed association, not proof that a longer sales cycle causes a deal to be won or lost. Factors such as deal complexity, customer decision-making, product type, and sales approach may also influence the duration.

### 10.8. Revenue Trend and Monthly Performance

Monthly won revenue was analyzed across the available closing months in 2017.

**Key Findings:**

- Total won revenue for the analyzed period was $10,005,534.
- June generated the highest monthly won revenue at $1,338,466.
- March generated $1,134,672, while December closed the year with $1,131,573.
- Monthly revenue showed considerable variation, with notable decreases in April, July, and October.
- Monthly win rates also fluctuated, ranging from 48.63% in April to 82.84% in June.

**Recommendations:**

- Monitor monthly revenue and win-rate fluctuations to identify periods requiring further investigation.
- Compare monthly results with opportunity volume and deal size to understand the factors behind changes in revenue.
- Use cumulative revenue tracking to monitor progress toward sales targets throughout the year.
- Avoid interpreting monthly fluctuations as long-term trends without additional periods of data.

### 10.9. Executive-Level Recommendations

Based on the combined findings, the following actions are recommended:

1. **Focus on High-Revenue Products:** Maintain strong attention on GTX Pro, GTX Plus Pro, and MG Advanced, which collectively contribute 83.52% of won revenue.

2. **Improve Sales Conversion:** Review the practices of high-performing representatives and provide targeted coaching to representatives whose win rates are below their regional averages.

3. **Strengthen Regional Performance:** Study West's revenue and conversion performance, East's average deal value and sales cycle, and opportunities to improve Central's conversion efficiency.

4. **Develop Customer-Specific Strategies:** Prioritize high-value accounts and investigate sector-level differences, especially the strong revenue contribution of Retail and the relatively lower win rate in Finance.

5. **Monitor Sales Cycle Efficiency:** Track sales cycle duration together with win rate and average deal value rather than treating shorter sales cycles as an independent measure of success.

6. **Improve Data Quality:** Address missing account and engagement-date values to increase the completeness and reliability of future analysis.

7. **Build a Consistent KPI Monitoring Framework:** Track revenue, win rate, opportunity volume, average deal value, and sales cycle duration regularly to support evidence-based decisions.

### 10.10. Analysis Limitations

The findings should be interpreted in the context of the available dataset:

- The analysis covers 8,800 opportunities and closing activity from March through December 2017.
- A total of 500 opportunities have missing engagement dates and are excluded from sales cycle calculations.
- A total of 1,425 opportunities have missing account values and cannot be included in account-linked sector and customer analyses.
- GTK 500 has a relatively small number of closed and won opportunities, so its performance metrics should be interpreted cautiously.
- The dataset does not include marketing costs, lead sources, customer acquisition costs, or detailed stage-transition timestamps. Therefore, analyses of marketing ROI, acquisition efficiency, and exact stage conversion duration are outside the scope of this project.
- Observed relationships and differences do not necessarily establish causation.

**Overall Conclusion:**

The analysis shows that the company generated $10.0 million in won revenue with an overall closed-deal win rate of 63.15%. Revenue is concentrated in three major products, while meaningful performance differences exist across sales representatives, regional offices, managers, and customer sectors.

The findings provide a data-driven foundation for improving product focus, sales conversion, account management, and performance monitoring. By combining multiple KPIs rather than relying on individual metrics, the business can make more balanced and informed sales decisions.

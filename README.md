
# SQL Queries - Customer Lifecycle Analysis

This folder contains the SQL-based analysis for the customer lifecycle project. It is designed to explore customer revenue patterns, segment performance, lifetime value, and monthly trends using SQLite and Python.

## Objective

The analysis answers the following business questions:

- Which customer segments generate the most revenue?
- How concentrated is revenue across customer segments?
- Who are the highest-value customers?
- How does revenue change over time?
- Are there seasonal patterns in customer behavior?
- Can churn and retention risk be inferred from customer segmentation data?

## Project Scope

The SQL analysis covers:

- Segment revenue summary
- Revenue concentration by segment
- Top customers by lifetime value
- Monthly revenue trends
- Customer behavior by segment
- SQL cross-referencing between RFM outputs and transaction data

## Data Sources

This project uses the following files from the `Data` folder:

1. `rfm_segments.csv`
   - Customer-level segmentation output
   - Typical columns:
     - `CustomerID`
     - `Recency`
     - `Frequency`
     - `Monetary`
     - `segment`

2. `cleaned_online_retail_data.csv`
   - Cleaned transaction dataset
   - Typical columns:
     - `Customer ID`
     - `Invoice`
     - `InvoiceDate`
     - `InvoiceYearMonth`
     - `TotalRevenue`
     - `Quantity`
     - `UnitPrice`
     - `Country`

## Requirements

Before running the notebook, install the following:

- Python 3.9+
- pandas
- sqlite3 (included with Python)
- Jupyter Notebook or VS Code Python environment

Install dependencies:

pip install pandas notebook jupyter


## Setup

The notebook connects to a local SQLite database inside the project `Data` folder.

### Python setup

```python
import os
import sqlite3
import pandas as pd
```

### Database path

```python
BASE_DIR = os.path.expanduser('~/Downloads/Customer Life cycle Analysis')
DB_PATH = os.path.join(BASE_DIR, 'Data', 'customer_lifecycle.db')
conn = sqlite3.connect(DB_PATH)
```

## Data Loading

```python
rfm_df = pd.read_csv(
    os.path.join(BASE_DIR, 'Data', 'rfm_segments.csv'),
    dtype={'CustomerID': str}
)

df = pd.read_csv(
    os.path.join(BASE_DIR, 'Data', 'cleaned_online_retail_data.csv'),
    dtype={'Customer ID': str},
    parse_dates=['InvoiceDate']
)
```

Then write both datasets into SQLite:

```python
rfm_df.to_sql('rfm_segments', conn, if_exists='replace', index=False)
df.to_sql('transactions', conn, if_exists='replace', index=False)
```

## Query Helper Function

A reusable function is used to execute each SQL statement and print the results cleanly:

```python
def execute_query(sql, description=None):
    result = pd.read_sql_query(sql, conn)
    if description:
        print(f"\n{description}\n")
    print(result.to_string(index=False))
    print(f"\nQuery executed successfully. Result shape: {result.shape}\n")
    return result
```

## SQL Analyses Included

### 1. Sample RFM Data

```sql
SELECT *
FROM rfm_segments
LIMIT 5;
```

### 2. Segment Revenue Summary

```sql
SELECT
    segment,
    COUNT(*) AS customer_count,
    SUM(monetary) AS total_revenue,
    AVG(monetary) AS avg_revenue_per_customer
FROM rfm_segments
GROUP BY segment
ORDER BY total_revenue DESC;
```

### 3. Revenue Concentration by Segment

```sql
SELECT
    segment,
    SUM(monetary) AS total_revenue,
    SUM(monetary) * 100 / SUM(SUM(monetary)) OVER() AS revenue_percentage
FROM rfm_segments
GROUP BY segment
ORDER BY total_revenue DESC;
```

### 4. Top 10 Customers by Lifetime Value

```sql
SELECT
    "Customer ID",
    Frequency,
    Recency,
    SUM(Monetary) AS total_revenue
FROM rfm_segments
GROUP BY "Customer ID", Frequency, Recency
ORDER BY total_revenue DESC
LIMIT 10;
```

### 5. Monthly Revenue Trend

```sql
SELECT
    t."InvoiceYearMonth" AS invoice_yearmonth,
    COUNT(DISTINCT t."Invoice") AS total_invoices,
    COUNT(DISTINCT t."Customer ID") AS total_customers,
    SUM(t."TotalRevenue") AS total_revenue
FROM transactions t
INNER JOIN rfm_segments r
    ON t."Customer ID" = r."Customer ID"
GROUP BY t."InvoiceYearMonth"
ORDER BY t."InvoiceYearMonth";
```

## Notes

- `Customer ID` values are treated as strings to avoid mismatches in SQL joins.
- SQLite does not require a server installation.
- The notebook is designed to be run locally in VS Code or Jupyter.
- Some column names may vary slightly depending on the source data, so check header names before executing queries.

## How to Run

1. Open the project in VS Code.
2. Open the notebook:
   - `sql analysis/sql analysis.ipynb`
3. Run all cells in order.
4. Confirm that the database file is created in:
   - `~/Downloads/Customer Life cycle Analysis/Data/customer_lifecycle.db`

## Summary

This SQL workflow provides a practical and reusable way to analyze customer lifecycle performance using RFM segmentation and transaction data. It combines segmentation logic, SQL aggregation, and monthly trend analysis to help understand customer value and business performance.

## Related Files

- `sql analysis/sql analysis.ipynb`
- `Data/rfm_segments.csv`
- `Data/cleaned_online_retail_data.csv`
- `Data/customer_lifecycle.db`
````
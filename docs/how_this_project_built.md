# How We Built the Retail Enterprise BI Control Tower

**Author:** Arun Prabakar Vadaseri Rajendran  
**Project:** Retail Enterprise BI Control Tower  
**Tools Used:** Google Colab, Gemini (in Colab), Claude (Cowork), Power BI Service, Tableau Public, Looker Studio  
**Last Updated:** June 2026

---

## What This Document Is

This is the build narrative for the Retail Enterprise BI Control Tower project. It documents how the project was constructed, step by step: the decisions made, the reasoning behind each one, the prompts used to generate code, and what each step produced. It is written to be readable by anyone who wants to understand not just what was built, but how and why.

This document is updated at the end of each working session.

---

## The Dataset

Before any code was written, the business case was defined. The project needed a realistic retail dataset that could support analysis across sales, customers, products, stores, inventory, marketing, and returns. A synthetic dataset of 8 related tables was generated to simulate a multi-store retail chain operating across Canada.

**The 8 tables:**

| Table | Rows | What It Represents |
|---|---:|---|
| orders | 130,250 (raw) | One row per customer transaction |
| order_details | 303,124 | One row per line item within each order |
| customers | 50,100 (raw) | Customer profiles with segment, region, and demographics |
| products | 1,800 | Product catalogue with category, cost, and pricing |
| stores | 120 | Physical store locations with size and employee count |
| inventory | 720,000 | Monthly stock-level snapshots per store per product |
| returns | 22,734 | Return transactions linked back to specific order lines |
| marketing_campaigns | 900 | Campaign records with channel, spend, and dates |

The target structure was a star schema: orders and order_details as the central fact tables, with customers, products, stores, and marketing_campaigns as dimensions, and inventory and returns connected at the appropriate grain.

---

## Inspecting the Raw Data

The first notebook (`01_data_inspection.ipynb`) was built in Google Colab using Gemini for code generation. Its purpose was to establish the official starting condition of the data before any cleaning took place.

### Loading All 8 Tables

**Prompt used:**
```
I am building a data inspection notebook for a retail BI project in Google Colab. 
I have 8 CSV files uploaded directly to the Colab session at /content/:
customers.csv, inventory.csv, marketing_campaigns.csv, order_details.csv, 
orders.csv, products.csv, returns.csv, stores.csv

Write a Python code cell that:
1. Loads all 8 files into separate pandas DataFrames with clear variable names 
   (e.g. df_orders, df_customers, etc.)
2. Prints a summary for each table showing: table name, number of rows, 
   number of columns, and all column names
3. Uses a clear separator line between each table's output
4. Adds a comment above each section explaining what the table represents 
   in a retail business context

Keep the code clean, well-commented, and beginner-friendly.
```

**Result:**

| Table | Rows | Columns |
|---|---:|---:|
| Customers | 50,100 | 9 |
| Inventory | 720,000 | 7 |
| Marketing Campaigns | 900 | 8 |
| Order Details | 303,124 | 10 |
| Orders | 130,250 | 8 |
| Products | 1,800 | 9 |
| Returns | 22,734 | 7 |
| Stores | 120 | 9 |

**Total rows across all tables:** ~1,229,028

---

### Identifying Null Values

**Prompt used:**
```
I am doing a data inspection notebook for a retail BI project. I have 8 pandas 
DataFrames already loaded: df_customers, df_inventory, df_marketing, 
df_order_details, df_orders, df_products, df_returns, df_stores

Write a Python code cell that:
1. For each DataFrame, shows the count and percentage of missing/null values 
   per column -- only show columns that actually have missing values
2. If a table has no missing values at all, print a message saying 
   "No missing values found" for that table
3. Uses a clear header for each table showing the table name
4. At the end, prints a total summary showing which tables had issues 
   and how many columns were affected

Format the output to be easy to read. Add comments explaining why 
null values matter in a BI/analytics context.
```

**Result:**

| Table | Column | Null Count | % Null |
|---|---|---:|---:|
| Customers | preferred_contact | 5,484 | 10.95% |
| Orders | campaign_id | 87,860 | 67.45% |
| Order Details | discount_pct | 1,212 | 0.40% |
| All others | (none) | 0 | N/A |

The campaign_id null rate of 67.45% in Orders is not a data error. Most orders genuinely had no marketing attribution. Replacing nulls with 'UNKNOWN' keeps all orders participating in the data model without breaking relationships. The discount_pct nulls in Order Details mean no discount was applied, so the correct fill is the number 0, not a string.

---

### Checking for Duplicates

**Prompt used:**
```
I am doing a data inspection notebook for a retail BI project. I have 8 pandas 
DataFrames already loaded: df_customers, df_inventory, df_marketing, 
df_order_details, df_orders, df_products, df_returns, df_stores

Write a Python code cell that checks for duplicate records in each table:
1. For each DataFrame, check for fully duplicate rows AND check for 
   duplicate values in the primary key column (use these keys: 
   customers=customer_id, inventory=no single key, 
   marketing=campaign_id, order_details=order_line_id, 
   orders=order_id, products=product_id, returns=return_id, 
   stores=store_id)
2. Print the count of fully duplicate rows per table
3. Print the count of duplicate primary key values per table
4. Show 2 sample duplicate rows where duplicates exist
5. Add a summary at the end listing which tables need deduplication

Add comments explaining why duplicates cause problems in BI reporting.
```

A targeted follow-up verification prompt was also run using value_counts to confirm exact duplicate counts per primary key:

```
The earlier duplicate check showed 0 duplicates for Orders and Customers but 
I suspect the files may be wrong. Write a targeted verification cell that for 
df_orders checks: total rows vs unique order_id count (using value_counts), 
and for df_customers checks: total rows vs unique customer_id count. 
Print the difference so I can see exactly how many duplicates exist.
```

**Result:**

| Table | Duplicate Rows | Action |
|---|---:|---|
| Customers | 100 | Remove |
| Orders | 250 | Remove |
| All others | 0 | No action |

Sample confirmed: OR0000718 appeared at rows 717 and 130,140 as an exact copy. All duplicates were fully identical rows, safe to deduplicate by keeping the first occurrence.

---

### Checking Categorical Consistency

**Prompt used:**
```
I am doing a data inspection notebook for a retail BI project. I have 8 pandas 
DataFrames already loaded: df_customers, df_inventory, df_marketing, 
df_order_details, df_orders, df_products, df_returns, df_stores

Write a Python code cell that checks for inconsistent categorical values:
1. For each DataFrame, identify all TEXT/OBJECT columns
2. For each text column, show all unique values -- if a column has more than 
   20 unique values, just show the count instead
3. Flag any columns where values appear inconsistent due to capitalisation 
   differences (e.g. "Apparel" vs "APPAREL"), leading/trailing spaces, 
   or mixed formats
4. Print a clear summary at the end listing which tables and columns need 
   standardisation
```

**Result:**

| Table | Column | Issue |
|---|---|---|
| Products | category | Mixed case: 'Apparel' and 'APPAREL' as separate values |
| All others | (none) | No issues found |

---

### Data Quality Baseline Report

**Prompt used:**
```
Write a Python code cell that prints a consolidated Data Quality Report 
as a formatted summary table with these columns:
- Table, Rows, Issue Type, Column Affected, Count, Action Required

Use the findings already in memory from the previous cells -- do not re-scan 
the data. Print this as a clean formatted table with a title 
"=== DATA QUALITY REPORT -- PRE-CLEANING BASELINE ===".

This table is the official handoff document from inspection to cleaning.
```

**Output:**

| Table | Rows | Issue Type | Column Affected | Count | Action Required |
|---|---:|---|---|---:|---|
| Customers | 50,100 | Null Values | preferred_contact | 5,484 | Fill with 'UNKNOWN' |
| Order Details | 303,124 | Null Values | discount_pct | 1,212 | Fill with 0 |
| Orders | 130,250 | Null Values | campaign_id | 87,860 | Fill with 'UNKNOWN' |
| Customers | 50,100 | Duplicates | All (row level) | 100 | Remove |
| Orders | 130,250 | Duplicates | All (row level) | 250 | Remove |
| Products | 1,800 | Category Mismatch | category | N/A | Standardise to title case |
| Inventory, Marketing, Returns, Stores | various | None | (none) | 0 | No action needed |

---

## Cleaning the Data

The second notebook (`02_data_cleaning.ipynb`) worked through every issue from the baseline report in order. All code was generated via Gemini in Colab.

### Removing Duplicates

**Prompt used:**
```
I am cleaning retail data in Google Colab. I have 8 pandas DataFrames loaded.
df_orders has 250 exact duplicate rows. df_customers has 100 exact duplicate rows.

Write a Python code cell that:
1. Removes duplicate rows from df_orders using drop_duplicates(), keeping the first occurrence
2. Removes duplicate rows from df_customers using drop_duplicates(), keeping the first occurrence
3. After each dedup, prints the before and after row count so I can verify
4. Prints a confirmation summary at the end
```

**Result:** Orders: 130,250 to 130,000. Customers: 50,100 to 50,000.

---

### Filling Null Values

**Prompt used:**
```
Three columns have null values that need to be filled:
- df_orders['campaign_id']: 87,860 nulls -> fill with the string 'UNKNOWN'
- df_customers['preferred_contact']: 5,484 nulls -> fill with the string 'UNKNOWN'
- df_order_details['discount_pct']: 1,212 nulls -> fill with the number 0 (not the string)
```

**Result:** All 3 columns confirmed 0 nulls after fill.

---

### Fixing Category Capitalisation

**Prompt used:**
```
df_products has a 'category' column with mixed capitalisation -- values like 
'Apparel' and 'APPAREL' exist as separate categories. 
Standardise by applying .str.strip().str.title() to df_products['category'].
Print unique values before and after to confirm.
```

**Result:** 12 raw values (6 proper case + 6 uppercase) collapsed to 6 clean title-case categories: Electronics, Apparel, Grocery, Furniture, Home, Office Supplies.

---

### Exporting the Clean Files

**Prompt used:**
```
Export each DataFrame as a CSV to /content/ with _clean suffix filenames.
Use index=False. Print filename and row count after each export.
```

**Clean files exported to `data/processed/`:**

| File | Clean Rows |
|---|---:|
| customers_clean.csv | 50,000 |
| inventory_clean.csv | 720,000 |
| marketing_campaigns_clean.csv | 900 |
| order_details_clean.csv | 303,124 |
| orders_clean.csv | 130,000 |
| products_clean.csv | 1,800 |
| returns_clean.csv | 22,734 |
| stores_clean.csv | 120 |

**Total clean rows:** 1,228,678

All 8 CSVs were also combined into a single `all_in_one_dataset.xlsx` (8 sheets, one per table) for import into Power BI Service.

---

## Loading into Power BI

The clean dataset was uploaded to Power BI Service via the workspace Import flow: **Import > From this computer**, selecting `all_in_one_dataset.xlsx`. Power Query Online loaded all 8 sheets in one shot, with column types auto-detected correctly.

**Dataset created:** `Retail Enterprise BI - Star Schema`  
**Workspace:** Retail Enterprise BI Control Tower  
**Tables loaded:** customers_clean, inventory_clean, marketing_campaigns_clean, order_details_clean, orders_clean, products_clean, returns_clean, stores_clean  
**Dataset ID:** aef86651-63f2-47d9-83aa-cd6b49f00cec

The report canvas is open and ready for the next session.

---

## Setting Up GitHub

The full project folder was initialised as a git repository and pushed to GitHub as a public portfolio repo.

**Repository:** [PrabaAP/retail-enterprise-bi-control-tower](https://github.com/PrabaAP/retail-enterprise-bi-control-tower)  
**Branch:** main  
**Commits:** Initial commit (all data, notebooks, docs) + README + this narrative

What is in the repo:
- `data/raw/` : 8 original CSV files
- `data/processed/` : 8 clean CSVs + all_in_one_dataset.xlsx
- `notebooks/` : 01_data_inspection.ipynb, 02_data_cleaning.ipynb
- `docs/` : data dictionary, dataset README, this narrative
- `README.md` : project overview, star schema, tech stack, author bio

---

## Next Session

Power BI work continues from where the dataset was left:

1. Open semantic model in Power BI and set up star schema relationships in Model view
2. Add a Date dimension table for time intelligence DAX
3. Write DAX measures: Total Revenue, Return Rate, MoM Growth, YTD Revenue, Customer Lifetime Value, Inventory Turnover, Marketing ROI, and others
4. Build 9 report pages: Executive Summary, Sales Performance, Product Analytics, Customer Insights, Store Operations, Inventory Management, Marketing ROI, Returns Analysis, Financial Summary

---
<!-- NEXT SESSION ENTRIES ADDED BELOW -->

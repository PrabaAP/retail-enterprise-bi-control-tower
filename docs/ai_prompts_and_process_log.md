# AI Prompts and Process Log
## Retail Enterprise BI Control Tower

**Author:** Arun Prabakar  
**Project:** Retail Enterprise BI Control Tower  
**Tools Used:** Google Colab, Gemini (in Colab), Power BI Service, Tableau Public, Looker Studio  
**Last Updated:** June 2026

---

## Purpose of This Document

This log documents every AI prompt used to generate code in this project, the reasoning behind each step, and the findings produced. It serves as a transparent record of the AI-assisted analytics workflow and demonstrates how AI tools were used responsibly — to accelerate execution while maintaining full understanding and control of the process.

---

## Project Workflow Overview

```
Raw CSV Files (8 tables)
    ↓
Phase 1: Data Inspection (Google Colab + Gemini)
    ↓
Phase 2: Data Cleaning (Google Colab + Gemini)
    ↓
Phase 3: Power BI — Data Model + DAX + 9 Report Pages (Power BI Service)
    ↓
Phase 4: Tableau — Data Model + Calculated Fields + 8 Dashboards (Tableau Public)
    ↓
Phase 5: Looker Studio — Full Dashboard (Looker Studio)
    ↓
Phase 6: Portfolio Packaging (GitHub, LinkedIn, Demo Video)
```

---

## Notebook: 01_data_inspection.ipynb

**Purpose:** Profile all 8 raw tables before any cleaning. Establishes the official starting condition of the data — row counts, columns, data types, null values, duplicates, and categorical value issues.

**Why this matters:** In any real BI project, raw data is never assumed to be clean. Documenting the starting condition proves the cleaning work was necessary and deliberate, not accidental.

---

### Cell 1 — Notebook Title (Text Cell)

**Type:** Markdown/Text  
**Action:** Manually written — no AI prompt needed.  
**Content:** Project title, purpose, author, date.

---

### Cell 2 — Import Libraries

**Type:** Code  
**Action:** Manually written.  
**Content:** Import pandas, os. Set display options.

---

### Cell 3 — Load All 8 Tables

**Type:** Code  
**AI Tool:** Gemini (in Colab)

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

**Findings:**

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

> **Note:** First run showed Orders=130,000 and Customers=50,000 because clean_reference files were uploaded by mistake. Counts above reflect the correct raw files.

---

### Cell 4 — Null Value Analysis

**Type:** Code  
**AI Tool:** Gemini (in Colab)

**Prompt used:**
```
I am doing a data inspection notebook for a retail BI project. I have 8 pandas 
DataFrames already loaded: df_customers, df_inventory, df_marketing, 
df_order_details, df_orders, df_products, df_returns, df_stores

Write a Python code cell that:
1. For each DataFrame, shows the count and percentage of missing/null values 
   per column — only show columns that actually have missing values
2. If a table has no missing values at all, print a message saying 
   "No missing values found" for that table
3. Uses a clear header for each table showing the table name
4. At the end, prints a total summary showing which tables had issues 
   and how many columns were affected

Format the output to be easy to read. Add comments explaining why 
null values matter in a BI/analytics context.
```

**Findings:**

| Table | Column | Null Count | % Null |
|---|---|---:|---:|
| Customers | preferred_contact | 5,484 | 10.95% |
| Orders | campaign_id | 87,860 | 67.45% |
| Order Details | discount_pct | 1,212 | 0.40% |
| Inventory | — | 0 | — |
| Marketing Campaigns | — | 0 | — |
| Products | — | 0 | — |
| Returns | — | 0 | — |
| Stores | — | 0 | — |

**BI Impact:** Null campaign_ids in Orders (67.45%) means most orders had no marketing attribution — replace with 'UNKNOWN' so all orders can be linked in the data model. Null preferred_contact in Customers is non-critical — replace with 'UNKNOWN'. Null discount_pct in Order Details should be filled with **0** (meaning no discount was applied, not missing data).

---

### Cell 5 — Duplicate Check

**Type:** Code  
**AI Tool:** Gemini (in Colab)

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

> **Process note — File upload error caught and corrected:** First Colab upload used clean_reference files by mistake (orders showed 130,000 rows, customers showed 50,000 rows). Correct raw files re-uploaded from `retail-enterprise-bi-control-tower/data/raw/`. Confirmed correct counts after re-upload: Orders 130,250 rows, Customers 50,100 rows. All cells re-run with correct data. This is an example of why data verification is done before proceeding.

**Findings:**

| Table | Duplicate Rows | Duplicate Primary Keys | Action |
|---|---:|---:|---|
| Customers | 100 | 100 | Remove duplicates |
| Orders | 250 | 250 | Remove duplicates |
| Order Details | 0 | 0 | No action |
| Products | 0 | 0 | No action |
| Returns | 0 | 0 | No action |
| Inventory | 0 | N/A | No action |
| Marketing Campaigns | 0 | 0 | No action |
| Stores | 0 | 0 | No action |

**Sample confirmed:** OR0000718 appeared at rows 717 and 130140 (exact copy). CU000280 (Noah Martin) appeared at rows 279 and 50090 (exact copy). All duplicates are fully identical rows — safe to deduplicate by keeping first occurrence.

---

### Cell 10 — Targeted ID Verification

**Type:** Code  
**AI Tool:** Gemini (in Colab)

**Prompt used:**
```
The earlier duplicate check showed 0 duplicates for Orders and Customers but 
I suspect the files may be wrong. Write a targeted verification cell that for 
df_orders checks: total rows vs unique order_id count (using value_counts), 
and for df_customers checks: total rows vs unique customer_id count. 
Print the difference so I can see exactly how many duplicates exist.
```

**Findings:**
- Orders: 130,250 total / 130,000 unique / **250 duplicates confirmed**
- Customers: 50,100 total / 50,000 unique / **100 duplicates confirmed**

> This cell was needed because the first duplicate check used `df.duplicated()` which can miss soft duplicates — the targeted value_counts approach is more reliable.

---

### Cell 11 — Categorical Values Check

**Type:** Code  
**AI Tool:** Gemini (in Colab)

**Prompt used:**
```
I am doing a data inspection notebook for a retail BI project. I have 8 pandas 
DataFrames already loaded: df_customers, df_inventory, df_marketing, 
df_order_details, df_orders, df_products, df_returns, df_stores

Write a Python code cell that checks for inconsistent categorical values:
1. For each DataFrame, identify all TEXT/OBJECT columns
2. For each text column, show all unique values — if a column has more than 
   20 unique values, just show the count instead
3. Flag any columns where values appear inconsistent due to capitalisation 
   differences (e.g. "Apparel" vs "APPAREL"), leading/trailing spaces, 
   or mixed formats
4. Print a clear summary at the end listing which tables and columns need 
   standardisation

Add comments explaining why inconsistent categories cause problems in 
BI tools like Power BI and Tableau.
```

**Findings:**

| Table | Column | Issue |
|---|---|---|
| Products | category | Mixed case: 'Apparel' and 'APPAREL' treated as separate values |
| All others | — | No categorical inconsistencies found |

**BI Impact:** In Power BI and Tableau, 'Apparel' and 'APPAREL' appear as two separate category values and produce split totals in charts. Must standardise to title case before loading.

---

### Cell 12 — Data Quality Summary Report

**Type:** Code  
**AI Tool:** Gemini (in Colab)

**Prompt used:**
```
I am finishing my data inspection notebook for a retail BI project. I have 
8 pandas DataFrames loaded and have already identified: null values, 
duplicate rows, duplicate primary keys, and categorical inconsistencies.

Write a Python code cell that prints a consolidated Data Quality Report 
as a formatted summary table with these columns:
- Table: table name
- Rows: total row count
- Issue Type: what kind of issue (Nulls / Duplicates / Category Mismatch / None)
- Column Affected: which column has the issue
- Count: how many records are affected
- Action Required: what needs to be done to fix it (e.g. "Remove 250 duplicate 
  rows", "Fill 87,860 nulls with UNKNOWN", "Standardise to title case")

Use the findings already in memory from the previous cells — do not re-scan 
the data. Print this as a clean formatted table at the end with a title 
"=== DATA QUALITY REPORT — PRE-CLEANING BASELINE ===".

This table is the official handoff document from inspection to cleaning.
```

**Output — DATA QUALITY REPORT — PRE-CLEANING BASELINE:**

| Table | Rows | Issue Type | Column Affected | Count | Action Required |
|---|---:|---|---|---:|---|
| Customers | 50,100 | Null Values | preferred_contact | 5,484 | Fill nulls with 'UNKNOWN' |
| Order Details | 303,124 | Null Values | discount_pct | 1,212 | Fill nulls with **0** (no discount) |
| Orders | 130,250 | Null Values | campaign_id | 87,860 | Fill nulls with 'UNKNOWN' |
| Inventory | 720,000 | None | — | 0 | No action needed |
| Marketing Campaigns | 900 | None | — | 0 | No action needed |
| Returns | 22,734 | None | — | 0 | No action needed |
| Stores | 120 | None | — | 0 | No action needed |
| Customers | 50,100 | Duplicates | All (Row level) | 100 | Remove 100 duplicate rows |
| Orders | 130,250 | Duplicates | All (Row level) | 250 | Remove 250 duplicate rows |
| Products | 1,800 | Category Mismatch | category | N/A | Standardise to title case and trim spaces |

> **Important note on discount_pct:** Gemini's generated report said "Fill with UNKNOWN" but discount_pct is a NUMERIC field. The correct action is to fill nulls with **0**, meaning no discount was applied. This will be handled correctly in the cleaning notebook.

---

## Phase 1 Complete ✓

**Date completed:** June 2026  
**Notebook:** `01_data_inspection.ipynb`  
**Status:** All 8 raw tables profiled. Data Quality Report produced as official handoff to Phase 2.

**Summary of issues found:**
- 3 tables have null values (Customers, Orders, Order Details) — 94,556 affected cells total
- 2 tables have duplicate rows (Customers: 100, Orders: 250) — 350 rows to remove
- 1 table has categorical inconsistency (Products: category column mixed case)
- 5 tables are clean with no issues (Inventory, Marketing Campaigns, Returns, Stores, Order Details PK)

**Next:** Phase 2 — Data Cleaning notebook (`02_data_cleaning.ipynb`)

---

## Notebook: 02_data_cleaning.ipynb

**Purpose:** Fix all issues identified in Phase 1 and export 8 clean CSVs to `data/processed/`.

---

### Cell 4 — Remove Duplicates

**Type:** Code | **AI Tool:** Gemini (in Colab)

**Prompt used:**
```
I am cleaning retail data in Google Colab. I have 8 pandas DataFrames loaded.
df_orders has 250 exact duplicate rows. df_customers has 100 exact duplicate rows.

Write a Python code cell that:
1. Removes duplicate rows from df_orders using drop_duplicates(), keeping the first occurrence
2. Removes duplicate rows from df_customers using drop_duplicates(), keeping the first occurrence
3. After each dedup, prints the before and after row count so I can verify
4. Prints a confirmation summary at the end showing rows removed from each table
```

**Result:** Orders: 130,250 → 130,000 (250 removed). Customers: 50,100 → 50,000 (100 removed).

---

### Cell 5 — Fill Null Values

**Type:** Code | **AI Tool:** Gemini (in Colab)

**Prompt used:**
```
I am cleaning retail data in Google Colab. I have 8 pandas DataFrames loaded.
Three columns have null values that need to be filled:
- df_orders['campaign_id']: 87,860 nulls → fill with the string 'UNKNOWN'
- df_customers['preferred_contact']: 5,484 nulls → fill with the string 'UNKNOWN'
- df_order_details['discount_pct']: 1,212 nulls → fill with the number 0 (not the string)
```

**Result:** All 3 columns confirmed 0 nulls after fill. discount_pct correctly filled with 0 (numeric).

---

### Cell 6 — Fix Category Capitalisation

**Type:** Code | **AI Tool:** Gemini (in Colab)

**Prompt used:**
```
df_products has a 'category' column with mixed capitalisation — values like 
'Apparel' and 'APPAREL' exist as separate categories when they should be the same.
Standardise the column by applying .str.strip().str.title() to df_products['category'].
Print unique values before and after to confirm.
```

**Result:** 12 raw values (6 proper case + 6 uppercase) collapsed to 6 clean title-case categories: Electronics, Apparel, Grocery, Furniture, Home, Office Supplies.

---

### Cell 7 — Verification

**Type:** Code | **AI Tool:** Gemini (in Colab)

**Result:** All tables passed. 0 remaining nulls in all checked columns. "CLEANING COMPLETE — Ready for export" confirmed.

---

### Cell 8 — Export Clean CSVs

**Type:** Code | **AI Tool:** Gemini (in Colab)

**Prompt used:**
```
Export each DataFrame as a CSV to /content/ with _clean suffix filenames.
Use index=False. Print filename and row count after each export.
```

**Files exported to `data/processed/`:**

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

---

## Phase 2 Complete ✓

**Date completed:** June 2026  
**Notebook:** `02_data_cleaning.ipynb`  
**Status:** All 8 tables cleaned and exported to `data/processed/`. Ready for Power BI.

**Next:** Phase 3 — Power BI data model (upload clean CSVs to Power BI Service, build star schema)

---
<!-- NEW PROMPTS WILL BE ADDED BELOW AS THE PROJECT PROGRESSES -->

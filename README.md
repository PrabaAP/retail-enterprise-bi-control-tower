# Retail Enterprise BI Control Tower

A corporate-level Business Intelligence portfolio project built across **Power BI**, **Tableau**, and **Looker Studio** — three tools, one retail business case, full stack.

## Project Overview

This project simulates a real-world enterprise BI environment for a multi-store retail chain. The same dataset (8 tables, 1.2M+ rows) powers dashboards across all three platforms, demonstrating tool-agnostic BI skills: data modelling, DAX, LOD expressions, calculated fields, and executive reporting.

**Business Questions Answered:**
- How is revenue trending month-over-month and year-to-date?
- Which products, categories, and stores drive the most profit?
- What is the customer lifetime value distribution by segment?
- Where are inventory stockout risks highest?
- Which marketing campaigns have the best ROI?
- What is the return rate and what drives returns?

---

## Dataset

Synthetic retail dataset with 8 related tables forming a star schema.

| Table | Rows | Description |
|---|---|---|
| orders | 130,000 | Transaction headers — customer, store, channel, date |
| order_details | 303,124 | Line items — product, quantity, revenue, cost, profit |
| customers | 50,000 | Customer profile — segment, region, age, income band |
| products | 1,800 | Product catalogue — category, cost, price, tier |
| stores | 120 | Store locations — region, size, employees, open date |
| inventory | 720,000 | Monthly stock snapshots — stock on hand, reorder point, stockout risk |
| returns | 22,734 | Return records — reason, refund amount, resolution |
| marketing_campaigns | 900 | Campaign metadata — channel, spend, objective, dates |

**Total: ~1.23M rows across all tables**

### Star Schema Relationships

```
marketing_campaigns ─┐
customers ────────────┤
stores ───────────────┼─── orders ──── order_details ──── products
                      │                     │
                  returns ─────────────────┘
inventory ──── stores
inventory ──── products
```

---

## Tools & Phases

### Phase 1 — Data Inspection (`notebooks/01_data_inspection.ipynb`)
- Profiled all 8 raw CSV files
- Identified nulls, data types, key distributions, and outliers
- Documented schema and quality issues

### Phase 2 — Data Cleaning (`notebooks/02_data_cleaning.ipynb`)
- Standardised column names and formats
- Handled missing values and type conversions
- Validated referential integrity across all relationship keys
- Output: 8 clean CSVs + combined `all_in_one_dataset.xlsx`

### Phase 3 — Power BI *(in progress)*
- Star schema data model with all relationships
- 15+ DAX measures: Revenue, Return Rate, MoM Growth, CLV, Inventory Turnover, Marketing ROI
- 9 report pages: Executive Summary, Sales Performance, Product Analytics, Customer Insights, Store Operations, Inventory Management, Marketing ROI, Returns Analysis, Financial Summary

### Phase 4 — Tableau *(planned)*
- Same 9-page report structure rebuilt in Tableau
- LOD expressions, table calculations, parameter actions

### Phase 5 — Looker Studio *(planned)*
- Web-native dashboard using Google Looker Studio
- Blended data sources, community visualisations

---

## Repository Structure

```
retail-enterprise-bi-control-tower/
├── data/
│   ├── raw/              # Original source CSV files (8 tables)
│   └── processed/        # Cleaned CSVs + all_in_one_dataset.xlsx
├── notebooks/
│   ├── 01_data_inspection.ipynb
│   └── 02_data_cleaning.ipynb
├── docs/
│   ├── data_dictionary.csv
│   ├── README_DATASET.md
│   └── ai_prompts_and_process_log.md
├── powerbi/              # Power BI assets (reports, screenshots)
├── tableau/              # Tableau assets
└── looker_studio/        # Looker Studio assets
```

---

## Tech Stack

| Layer | Tools |
|---|---|
| Data cleaning | Python, pandas, numpy |
| BI — Layer 1 | Power BI Service (DAX, Power Query) |
| BI — Layer 2 | Tableau |
| BI — Layer 3 | Looker Studio |
| Version control | Git, GitHub |
| Documentation | Markdown, Jupyter Notebooks |

---

## Key Skills Demonstrated

- End-to-end data pipeline: raw CSV → cleaned dataset → star schema → dashboards
- Data modelling: star schema design, relationship management, cardinality
- DAX: time intelligence, CALCULATE, iterators, context transition
- Power Query: M language, type inference, applied steps
- Cross-platform BI: same business case implemented in 3 different tools
- Data storytelling: executive-level report design across 9 report pages

---

## Author

**Prabakar** — Data Analytics Portfolio  
[GitHub](https://github.com/PrabaAP/retail-enterprise-bi-control-tower)

# PySpark E-commerce Analytics Project

## Goal
Build a practical batch ETL pipeline that reads raw CSVs, profiles and cleans data,
joins orders/items/customers/products, calculates revenue metrics, validates output,
and writes curated Parquet datasets.

## Files
- data/raw/orders.csv
- data/raw/order_items.csv
- data/raw/customers.csv
- data/raw/products.csv

The sample data intentionally contains quality issues:
- duplicate order and customer rows
- inconsistent order status casing
- inconsistent date formats and an invalid date
- missing and unknown customer IDs
- invalid item quantity and unit price
- unknown product ID

Do not silently discard every problem. Profile it, define explicit rules, and retain
rejected rows in a quality/rejects output where appropriate.

## Suggested project structure
pyspark_ecommerce_project/
  data/raw/
  data/processed/
  src/
    main.py
  README.md

## Install locally
Python 3.10 or 3.11 is a sensible learning setup. Install Java (a compatible JDK
for your chosen Spark version), then:
    python -m venv .venv
    # Windows:
    .venv\Scripts\activate
    # macOS/Linux:
    source .venv/bin/activate
    python -m pip install pyspark

Verify:
    python -c "import pyspark; print(pyspark.__version__)"

## First task
Do not build the full pipeline yet. Read each CSV with an explicit schema where
practical, print schemas, count rows, and inspect a few rows. Record which data
quality issues you can detect before cleaning.

## Business rules to decide and document
1. What qualifies as revenue? Usually completed orders only, but define how invalid
   dates, quantities, prices, missing customers, and unknown products are handled.
2. Should duplicate orders be removed, quarantined, or reconciled? Choose a stable
   business key and explain it.
3. Keep raw data unchanged; write cleaned data to a separate location.
4. Do not count an order as valid revenue merely because its status is completed.
   Validate its line items too.

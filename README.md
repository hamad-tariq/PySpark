# PySpark Complete Guide & E-commerce Analytics Project

![Python](https://img.shields.io/badge/Python-3.10%20|%203.11-3776AB?logo=python&logoColor=white)
![PySpark](https://img.shields.io/badge/PySpark-3.5%2B-E25A1C?logo=apachespark&logoColor=white)
![Notebook](https://img.shields.io/badge/Jupyter-Colab%20ready-F9AB00?logo=googlecolab&logoColor=white)

A hands-on PySpark learning resource. The notebook [`PySpark.ipynb`](./PySpark.ipynb) starts with DataFrame basics and builds up to a batch ETL pipeline, data-quality checks, sales reporting, and interactive visualizations on a small e-commerce dataset.

**Contents:** [Highlights](#highlights) · [Quick start](#quick-start) · [Repository layout](#repository-layout) · [Learning path](#learning-path) · [Dataset](#dataset) · [ETL pipeline](#etl-pipeline) · [Outputs](#generated-outputs) · [Exercises](#suggested-exercises) · [Troubleshooting](#troubleshooting)

## Highlights

- **DataFrame fundamentals:** `SparkSession`, filtering, selecting, sorting, derived columns, `groupBy`/`agg`, and Spark SQL functions.
- **DataFrame comparison:** `distinct` vs. `exceptAll` (ignore vs. preserve duplicate counts).
- **Data quality:** profile schemas, nulls, duplicate rows and keys, invalid values, bad dates, and broken references.
- **ETL:** parse multiple date formats, normalize statuses, split valid from rejected records, join four datasets, compute line-level revenue.
- **Output and visualization:** Parquet reports, Plotly charts, an `ipywidgets` dashboard, and a small Streamlit example.

## Quick start

### Option A: Google Colab (easiest)

1. Open `PySpark.ipynb` in Colab.
2. Run the package-install and Spark-session cells.
3. When prompted, upload `pyspark_ecommerce_project.zip`. It is extracted to `/content/ecommerce_project`.
4. Run the remaining cells in order.

### Option B: Local

**Requirements:** Python 3.10 or 3.11, a JDK supported by your PySpark version (on `PATH`), and PySpark 3.5+ (the notebook uses `try_to_timestamp`).

```bash
python -m venv .venv
source .venv/bin/activate          # Windows: .\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
python -m pip install "pyspark>=3.5" jupyterlab pandas plotly ipywidgets streamlit
python -c "import pyspark; print(pyspark.__version__)"   # sanity check
jupyter lab PySpark.ipynb
```

In VS Code, open the notebook and select the virtual environment as the kernel.

The notebook's paths and a few commands are Colab-specific (`google.colab`, `/content`, `files.download`, `!` shell commands, `%%writefile`). Skip or replace those cells locally and read the CSVs directly:

```python
from pathlib import Path

data_dir = Path("pyspark_ecommerce_project") / "data" / "raw"
orders = (
    spark.read
    .option("header", True)
    .option("inferSchema", True)
    .csv(str(data_dir / "orders.csv"))
)
```

Write outputs to a local directory such as `pyspark_ecommerce_project/output`. Parquet writes use `mode("overwrite")`, so rerunning replaces the previous output.

## Repository layout

```text
.
├── PySpark.ipynb                     # main tutorial notebook
├── README.md
├── pyspark_ecommerce_project.zip     # archive used by the Colab flow
└── pyspark_ecommerce_project/
    ├── README.md                     # data and business-rule notes
    └── data/raw/
        ├── customers.csv
        ├── order_items.csv
        ├── orders.csv
        └── products.csv
```

## Learning path

Work through the notebook in order. Later sections reuse variables from earlier ones, so rerun downstream cells after changing an upstream DataFrame or rule.

| # | Section | What you do |
| - | --- | --- |
| 1 | DataFrame fundamentals | Filter, select, sort, derive columns, aggregate |
| 2 | Compare DataFrames | `distinct` vs. `exceptAll` |
| 3 | Load data | Create a Spark session, load the four CSVs, inspect schemas |
| 4 | Profile quality | Nulls, duplicates, statuses, invalid quantities/prices/dates |
| 5 | Clean and validate | Normalize statuses, parse dates, label issues, split valid/rejected |
| 6 | Join data | Check references, build the line-level analytics DataFrame |
| 7 | Calculate metrics | Revenue by category and month, sales KPIs, customer spend |
| 8 | Persist outputs | Write and reload Parquet, quality and reference reports |
| 9 | Visualize and deploy | Plotly charts, widget dashboard, Streamlit example |

## Dataset

| File | Key fields | Purpose |
| --- | --- | --- |
| `orders.csv` | `order_id`, `customer_id`, `order_date`, `status` | Order header and status |
| `order_items.csv` | `item_id`, `order_id`, `product_id`, `quantity`, `unit_price` | Line items per order |
| `customers.csv` | `customer_id`, `customer_name`, `country` | Customer attributes |
| `products.csv` | `product_id`, `product_name`, `category` | Product catalog |

The data is small and synthetic, and it intentionally contains problems to find and handle: duplicate rows or keys, inconsistent status casing, mixed date formats and invalid dates, unknown references, and invalid quantities or prices. See the [project README](./pyspark_ecommerce_project/README.md) for more detail.

## ETL pipeline

`run_etl_pipeline(raw_path, output_path)` runs these stages:

1. **Extract** the four CSVs with headers and inferred schemas.
2. **Validate orders:** drop exact duplicates, parse supported date formats, normalize statuses, check required IDs, dates, and recognized statuses.
3. **Validate order items:** check IDs and require positive quantities and unit prices.
4. **Check references:** orders with unknown customers, and items with unknown orders or products, are kept out of the final join.
5. **Transform** accepted rows into a line-level dataset where `line_revenue = quantity * unit_price`.
6. **Calculate** KPIs and category metrics for completed orders.
7. **Load** curated datasets, rejects, and reports as Parquet.

> **Educational scope, not production governance**
>
> - `dropDuplicates()` removes exact copies only. Define a deterministic rule for conflicting duplicate `order_id`, `customer_id`, or `product_id` values.
> - Unknown order-item references are filtered out of the join, not written to a dedicated reject dataset.
> - Revenue is per order item, and KPIs use completed orders only. Decide how refunds, discounts, taxes, currency, and duplicate lines work before treating results as financial reporting.
> - `inferSchema` is a tutorial convenience. Use explicit schemas and data contracts in production.
> - `toPandas()` collects data on the driver. It is fine for this sample, not for large datasets.
> - The Streamlit section is a demo and lacks production access control, observability, and deployment configuration.

## Generated outputs

Parquet datasets written under the output path:

| Directory | Contents |
| --- | --- |
| `analytics/` | Joined order, item, customer, and product data with line revenue |
| `category_sales/` | Completed-order revenue, units, and order counts by category |
| `sales_kpis/` | Overall revenue, order count, and units sold (completed orders) |
| `rejected_orders/` | Orders failing validation |
| `rejected_items/` | Order items failing field validation |
| `unknown_customer_orders/` | Otherwise-valid orders with no matching customer |
| `quality_report/` | Accepted and rejected counts for orders and items |
| `reference_report/` | Count of orders with unknown customers |

`monthly_sales/` and `top_customers/` are produced by earlier reporting cells but not by the ETL function. Extend the function to include them.

## Suggested exercises

- Replace `inferSchema` with explicit `StructType` schemas and compare results.
- Add a documented policy for conflicting duplicate business keys.
- Write unknown order-item references to dedicated reject outputs and add them to the reference report.
- Add assertions for unique keys, valid statuses, non-null join keys, and accepted/rejected/quarantined reconciliation.
- Add metrics (average order value, product performance, date-range reports) and test them against hand-calculated values.
- Refactor the pipeline into Python modules with tests.

## Troubleshooting

| Symptom | Fix |
| --- | --- |
| Java gateway / JVM startup error | Install a JDK compatible with your PySpark version and put it on `PATH`. |
| File not found | You are likely mixing Colab and local paths. Point `data_dir` at `pyspark_ecommerce_project/data/raw`. |
| `google.colab` import error | Those cells are Colab-only. Skip them and read the local CSVs. |
| Plotly or widgets not displaying | Install the dependencies in the active kernel and restart it. |
| Missing downstream variable or column | Rerun from the relevant upstream cell. |

## License and data

The dataset is a small synthetic sample for learning. No license file is included yet; add one (for example MIT) before inviting reuse or contributions, and review terms before redistributing the data.

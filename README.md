# E-Commerce Analytics Pipeline

An end-to-end data engineering project simulating a production-grade e-commerce analytics platform, built using the **Medallion Architecture** (Bronze → Silver → Gold) with Python and Apache Spark (PySpark).

---

## Project Overview

This project demonstrates a full data pipeline lifecycle — from raw data ingestion through transformation and business-level aggregation — mirroring the architecture used in real-world data platforms at fintech and e-commerce companies.

**Key engineering decisions:**

- Medallion architecture separates raw, cleaned, and aggregated data into distinct layers, improving data quality and query performance
- PySpark handles transformations at scale — the same approach used in production Databricks environments
- Modular `src/` structure separates data generation, transformation logic, and analytics queries for maintainability and testability

---

## Architecture

```
Raw Data (API / Generated)
        │
        ▼
┌──────────────────┐
│   Bronze Layer   │  Raw ingestion — unmodified source data stored as-is
│  (data/bronze/)  │  Format: CSV / JSON
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│   Silver Layer   │  Cleaned & normalised — nulls removed, types cast,
│  (data/silver/)  │  schema enforced, deduplication applied
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│    Gold Layer    │  Business-ready aggregations — revenue metrics,
│  (data/gold/)    │  customer behaviour, product performance
└──────────────────┘
```

---

## Tech Stack

| Tool                   | Purpose                                       |
| ---------------------- | --------------------------------------------- |
| Python 3.10+           | Pipeline orchestration and data generation    |
| Apache Spark (PySpark) | Distributed data transformation               |
| Pandas                 | Lightweight data manipulation and exploration |
| SQL                    | Analytics queries across aggregated datasets  |
| Jupyter Notebooks      | Exploratory analysis and visualisation        |
| Git                    | Version control                               |

---

## Project Structure

```
ecommerce-analytics/
├── data/
│   ├── bronze/        # Raw generated e-commerce data
│   ├── silver/        # Cleaned and normalised datasets
│   └── gold/          # Aggregated business metrics
├── notebooks/
│   ├── 01_data_exploration.ipynb     # Bronze layer EDA
│   ├── 02_silver_transformation.ipynb # Cleaning and normalisation
│   └── 03_gold_analytics.ipynb       # Revenue and customer metrics
├── src/
│   ├── generate_data.py    # Synthetic data generation
│   ├── bronze_to_silver.py # Cleaning transformations (PySpark)
│   └── silver_to_gold.py   # Aggregation logic (PySpark)
├── .gitignore
└── README.md
```

---

## Data Pipeline

### Bronze → Silver (Cleaning)

- Cast data types (dates, numerics, booleans)
- Remove null and duplicate records
- Normalise column names to `snake_case`
- Validate schema consistency across batches

### Silver → Gold (Aggregation)

- **Revenue metrics:** total revenue by product, category, and time period
- **Customer behaviour:** purchase frequency, average order value, repeat rate
- **Product performance:** top sellers, return rates, revenue contribution

---

## Key Analytics (Gold Layer)

-- Monthly revenue by product category
SELECT
DATE_TRUNC('month', order_date) AS month,
product_category,
SUM(revenue) AS total_revenue,
COUNT(DISTINCT customer_id) AS unique_customers,
ROUND(SUM(revenue) /
COUNT(DISTINCT customer_id), 2) AS revenue_per_customer
FROM gold.order_metrics
GROUP BY 1, 2
ORDER BY 1, total_revenue DESC;

````

```sql
-- Customer retention: repeat purchasers
WITH purchase_counts AS (
    SELECT
        customer_id,
        COUNT(order_id) AS total_orders
    FROM silver.orders
    GROUP BY customer_id
)
SELECT
    ROUND(100.0 * SUM(CASE WHEN total_orders > 1 THEN 1 END)
          / COUNT(*), 1) AS repeat_customer_rate_pct
FROM purchase_counts;
````

---

## Sample Data Schema

**Orders (Bronze)**
| Column | Type | Description |
|---|---|---|
| order_id | STRING | Unique order identifier |
| customer_id | STRING | Customer reference |
| product_id | STRING | Product reference |
| order_date | TIMESTAMP | Order placement time |
| quantity | INTEGER | Units ordered |
| unit_price | DECIMAL | Price per unit |
| status | STRING | Order status (completed/returned/pending) |

---

## What I Learned / Engineering Decisions

- **Why Medallion?** Separating raw, cleaned, and aggregated data means upstream issues (bad source data) never corrupt downstream business metrics — a critical requirement in financial and e-commerce systems.
- **Why PySpark over pandas?** The transformation logic is written to scale — the same code runs on a single machine today and on a Databricks cluster against millions of rows tomorrow.
- **Why synthetic data?** Allows full control over schema, volume, and edge cases (nulls, duplicates, outliers) without privacy concerns — mirrors real data quality challenges.

---

## Next Steps

- [ ] Add Prefect orchestration to schedule pipeline runs
- [ ] Connect to Databricks for cloud-native execution
- [ ] Add data quality checks with Great Expectations
- [ ] Build Streamlit dashboard for Gold layer metrics
- [ ] Add unit tests with pytest for transformation functions

---

## Author

**Maryam Mohsenpour** — Data Engineer  
[LinkedIn](https://linkedin.com/in/maryammohsenpour-07632a16b) · [GitHub](https://github.com/bita2019)

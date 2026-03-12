---
name: databricks
description: |
  Databricks platform expert for the Lakehouse architecture.
  Generates notebooks, Delta Lake pipelines, Unity Catalog configs, SQL warehouses,
  MLflow experiments, job definitions, and cluster policies.
  Works across Databricks on AWS, Azure, and GCP.
allowed-tools: Read, Grep, Write, Bash, Glob
argument-hint: "[task: pipeline|sql|unity-catalog|jobs|clusters|mlflow|review] [description]"
---

## Role

You are a Databricks Solutions Architect. You design lakehouse architectures that unify data engineering, analytics, and ML on a single platform.

## Task Router

| Task | What You Do |
|------|------------|
| `pipeline` | Generate Delta Live Tables or Spark ETL pipeline code |
| `sql` | Write optimized Databricks SQL queries and dashboard SQL |
| `unity-catalog` | Design catalog/schema/table structure, grants, data governance |
| `jobs` | Create Databricks Workflows/Jobs with tasks, dependencies, alerts |
| `clusters` | Generate cluster policies, configs, pools for different workloads |
| `mlflow` | MLflow experiment tracking, model registry, serving configs |
| `review` | Review existing notebooks/jobs for performance, cost, best practices |

---

## Delta Live Tables (DLT)

### Medallion Architecture

```
Bronze (Raw)          Silver (Cleaned)         Gold (Business)
┌──────────┐         ┌──────────────┐         ┌─────────────┐
│ Raw JSON  │────────▶│ Parsed,      │────────▶│ Aggregated, │
│ Raw CSV   │  DLT    │ Deduplicated,│  DLT    │ Joined,     │
│ Raw Parquet│────────▶│ Typed,       │────────▶│ Business    │
│ CDC Feeds │         │ Validated    │         │ Ready       │
└──────────┘         └──────────────┘         └─────────────┘
```

### DLT Pipeline Template (Python)

```python
import dlt
from pyspark.sql.functions import *

# ── Bronze ──────────────────────────────────────
@dlt.table(
    name="orders_bronze",
    comment="Raw orders from source system",
    table_properties={"quality": "bronze"}
)
def orders_bronze():
    return (
        spark.readStream
        .format("cloudFiles")        # Auto Loader
        .option("cloudFiles.format", "json")
        .option("cloudFiles.schemaLocation", "/mnt/schema/orders")
        .load("/mnt/landing/orders/")
    )

# ── Silver ──────────────────────────────────────
@dlt.table(name="orders_silver", comment="Cleaned and validated orders")
@dlt.expect_or_drop("valid_amount", "amount > 0")
@dlt.expect_or_fail("valid_id", "order_id IS NOT NULL")
@dlt.expect("valid_date", "order_date >= '2020-01-01'")
def orders_silver():
    return (
        dlt.read_stream("orders_bronze")
        .select(
            col("order_id").cast("long"),
            col("customer_id").cast("long"),
            col("amount").cast("decimal(18,2)"),
            to_timestamp("order_date").alias("order_date"),
            current_timestamp().alias("_processed_at")
        )
        .dropDuplicates(["order_id"])
    )

# ── Gold ────────────────────────────────────────
@dlt.table(name="daily_revenue", comment="Daily revenue by customer segment")
def daily_revenue():
    orders = dlt.read("orders_silver")
    customers = dlt.read("customers_silver")
    return (
        orders.join(customers, "customer_id")
        .groupBy(
            date_trunc("day", "order_date").alias("date"),
            "segment"
        )
        .agg(
            sum("amount").alias("revenue"),
            countDistinct("customer_id").alias("unique_customers"),
            count("order_id").alias("order_count")
        )
    )
```

### DLT Data Quality

| Expectation | Behavior | Use When |
|-------------|----------|----------|
| `@dlt.expect` | Record metric, keep row | Soft quality check, monitoring |
| `@dlt.expect_or_drop` | Drop failing rows | Filter bad data, keep pipeline running |
| `@dlt.expect_or_fail` | Fail pipeline | Critical data integrity (PKs, required fields) |

---

## Unity Catalog

### Namespace Structure

```
catalog (one per environment or domain)
└── schema (one per data product / team)
    ├── tables
    ├── views
    ├── volumes (unstructured data)
    └── functions
```

### Recommended Layout

```sql
-- Environment-based catalogs
CREATE CATALOG IF NOT EXISTS dev;
CREATE CATALOG IF NOT EXISTS staging;
CREATE CATALOG IF NOT EXISTS prod;

-- Domain schemas within each catalog
CREATE SCHEMA IF NOT EXISTS prod.sales;
CREATE SCHEMA IF NOT EXISTS prod.marketing;
CREATE SCHEMA IF NOT EXISTS prod.finance;

-- Shared/common data
CREATE SCHEMA IF NOT EXISTS prod.common;
```

### Grants

```sql
-- Data engineering team: full control on their domain
GRANT USE CATALOG ON CATALOG prod TO `data-engineering`;
GRANT USE SCHEMA ON SCHEMA prod.sales TO `data-engineering`;
GRANT ALL PRIVILEGES ON SCHEMA prod.sales TO `data-engineering`;

-- Data analysts: read-only on gold tables
GRANT USE CATALOG ON CATALOG prod TO `data-analysts`;
GRANT USE SCHEMA ON SCHEMA prod.sales TO `data-analysts`;
GRANT SELECT ON SCHEMA prod.sales TO `data-analysts`;

-- ML engineers: read data + write to ML schema
GRANT SELECT ON SCHEMA prod.sales TO `ml-engineers`;
GRANT ALL PRIVILEGES ON SCHEMA prod.ml_models TO `ml-engineers`;
```

### Unity Catalog Best Practices
- **One catalog per environment** (dev/staging/prod) for clear isolation
- **Schemas = data products**, not source systems
- **External locations** registered centrally, managed by platform team
- **Table ownership** assigned to service principals for automated pipelines
- **Column-level masking** and **row-level security** for PII

---

## Databricks SQL

### Optimization Patterns

```sql
-- ✅ Use Delta-native features
OPTIMIZE catalog.schema.table ZORDER BY (partition_col, frequent_filter_col);

-- ✅ Liquid clustering (replaces partitioning + ZORDER in newer DBR)
ALTER TABLE catalog.schema.table
CLUSTER BY (region, order_date);

-- ✅ Predictive I/O (auto-optimizes reads)
-- Enabled at table level: delta.tuneFileSizesForRewrites

-- ✅ Materialized views for dashboards
CREATE OR REPLACE MATERIALIZED VIEW prod.sales.daily_summary AS
SELECT date, region, SUM(revenue) as total_revenue
FROM prod.sales.orders_gold
GROUP BY date, region;
```

### SQL Warehouse Sizing

| Workload | Size | Auto-Stop | Scaling |
|----------|------|-----------|---------|
| Ad-hoc queries (analysts) | Small–Medium | 10 min | 1–2 clusters |
| Dashboards | Medium | Never (during business hours) | 2–4 clusters |
| ETL queries | Large | 5 min | 1 cluster |
| Dev/test | X-Small | 5 min | 1 cluster |

---

## Jobs / Workflows

### Job Definition Template (JSON)

```json
{
  "name": "daily-sales-pipeline",
  "schedule": {
    "quartz_cron_expression": "0 0 6 * * ?",
    "timezone_id": "UTC"
  },
  "tasks": [
    {
      "task_key": "ingest_raw",
      "notebook_task": {
        "notebook_path": "/Repos/prod/pipelines/ingest_orders"
      },
      "job_cluster_key": "etl_cluster"
    },
    {
      "task_key": "transform_silver",
      "depends_on": [{"task_key": "ingest_raw"}],
      "notebook_task": {
        "notebook_path": "/Repos/prod/pipelines/transform_orders"
      },
      "job_cluster_key": "etl_cluster"
    },
    {
      "task_key": "build_gold",
      "depends_on": [{"task_key": "transform_silver"}],
      "notebook_task": {
        "notebook_path": "/Repos/prod/pipelines/build_daily_revenue"
      },
      "job_cluster_key": "etl_cluster"
    },
    {
      "task_key": "run_quality_checks",
      "depends_on": [{"task_key": "build_gold"}],
      "notebook_task": {
        "notebook_path": "/Repos/prod/pipelines/quality_checks"
      },
      "job_cluster_key": "etl_cluster"
    }
  ],
  "job_clusters": [
    {
      "job_cluster_key": "etl_cluster",
      "new_cluster": {
        "spark_version": "14.3.x-scala2.12",
        "node_type_id": "Standard_D4ds_v5",
        "autoscale": {"min_workers": 1, "max_workers": 4},
        "spark_conf": {
          "spark.sql.adaptive.enabled": "true",
          "spark.databricks.delta.optimizeWrite.enabled": "true",
          "spark.databricks.delta.autoCompact.enabled": "true"
        }
      }
    }
  ],
  "email_notifications": {
    "on_failure": ["data-team@company.com"]
  },
  "max_retries": 1,
  "timeout_seconds": 7200
}
```

---

## Cluster Configuration

### Cluster Policies

| Workload | Policy |
|----------|--------|
| **Interactive (dev)** | Auto-terminate 60 min, max 4 workers, single user |
| **Interactive (analysts)** | Auto-terminate 30 min, max 2 workers, shared |
| **ETL jobs** | Auto-scale 1–8 workers, job clusters only, spot/preemptible |
| **ML training** | GPU nodes allowed, max 8 workers, single user |

### Spark Config Recommendations

```json
{
  "spark.sql.adaptive.enabled": "true",
  "spark.sql.adaptive.coalescePartitions.enabled": "true",
  "spark.databricks.delta.optimizeWrite.enabled": "true",
  "spark.databricks.delta.autoCompact.enabled": "true",
  "spark.sql.shuffle.partitions": "auto",
  "spark.databricks.io.cache.enabled": "true"
}
```

---

## Review Checklist

When task is `review`:

| Area | Check |
|------|-------|
| **Delta format** | All tables are Delta? No Parquet/CSV output? |
| **Data quality** | DLT expectations or explicit checks on every pipeline? |
| **Medallion layers** | Clear bronze/silver/gold separation? |
| **Unity Catalog** | Tables registered in UC? Proper grants? |
| **Cluster sizing** | Right-sized? Using job clusters for jobs? Auto-terminate? |
| **Costs** | Spot instances for fault-tolerant workloads? Photon enabled? |
| **Idempotency** | MERGE INTO for upserts? Rerunnable without duplicates? |
| **Secrets** | Using Databricks secret scopes, not hardcoded? |
| **Monitoring** | Job alerts configured? Pipeline event logs checked? |

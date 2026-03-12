---
name: data-quality-check
description: |
  Generates data quality validation checks for tables, pipelines, or datasets.
  Use when you need to add assertions, Great Expectations suites, dbt tests,
  or custom validation scripts to catch data issues before they reach downstream consumers.
allowed-tools: Read, Grep, Write, Bash
argument-hint: "[table name or file path]"
---

## Role

You are a data quality engineer. You write checks that catch real problems without generating false alarm fatigue.

## Process

Given $ARGUMENTS (a table, file, or pipeline):

### 1. Profile the Data

Before writing checks, understand the data:
- Row count and growth pattern (append-only, full refresh, SCD)
- Column types and cardinality
- NULL rates per column
- Key uniqueness
- Value distributions for important columns
- Freshness (when was the last row added/updated?)

### 2. Generate Checks by Category

#### Schema Checks
- Column exists and has expected type
- No unexpected new columns (schema drift)
- Column order hasn't changed (for position-dependent consumers)

#### Completeness
- NOT NULL on required fields
- NULL rates within acceptable thresholds
- No empty strings masquerading as NULLs

#### Uniqueness
- Primary key uniqueness
- Business key uniqueness
- No full-row duplicates

#### Validity
- Values within expected ranges (dates, amounts, percentages)
- Enum/status columns contain only known values
- Email/phone/URL format validation where applicable
- Referential integrity (FK relationships hold)

#### Freshness
- Data arrived within expected SLA window
- No unexpected gaps in time-series data
- Partition freshness check

#### Volume
- Row count within expected range (± N% of rolling average)
- No sudden drops to zero
- Reasonable growth rate

#### Distribution
- Statistical distribution hasn't shifted dramatically
- No single value dominating where it shouldn't
- Outlier detection on numeric columns

### 3. Output Format

Detect the project's testing framework and generate accordingly:

- **dbt project** → schema tests in YAML + custom generic tests
- **Great Expectations** → expectation suite JSON
- **Python project** → pytest functions using pandas/polars assertions
- **SQL project** → standalone assertion queries that fail on bad data
- **No framework** → SQL checks with PASS/FAIL output

### 4. Alert Severity

Tag each check:
- 🔴 **BLOCK** — pipeline should halt (PK violation, zero rows, NULL in required field)
- 🟡 **WARN** — investigate but don't block (unusual volume, drift in distributions)
- ⚪ **INFO** — log for observability (row counts, freshness timestamps)

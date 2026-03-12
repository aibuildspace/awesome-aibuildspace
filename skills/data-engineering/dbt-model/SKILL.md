---
name: dbt-model
description: |
  Creates dbt models from requirements. Generates SQL models, schema YAML, and tests.
  Use when building new dbt models, staging layers, marts, or intermediate transformations.
allowed-tools: Read, Grep, Write, Bash, Glob
argument-hint: "[model description or source table]"
---

## Role

You are a dbt developer following the dbt best practices guide. You write clean, tested, documented models.

## Process

Given $ARGUMENTS describing the model:

### 1. Discover Project Context
- Read `dbt_project.yml` for project name, model paths, and materialization defaults
- Check `models/` directory structure for existing naming conventions
- Read `packages.yml` for available macros (dbt_utils, dbt_expectations, etc.)
- Identify the warehouse (Snowflake, BigQuery, Redshift, Postgres, Databricks)

### 2. Follow Layering Conventions

```
models/
├── staging/          -- 1:1 with source tables, renamed + retyped
│   └── stg_<source>__<table>.sql
├── intermediate/     -- business logic building blocks
│   └── int_<entity>__<verb>.sql
└── marts/            -- final business entities
    └── <entity>.sql
```

### 3. Generate Model Files

For each model, create:

**SQL Model** — using CTEs:
```sql
with source as (
    select * from {{ ref('...') }}
),

renamed as (
    select
        -- primary key first
        -- dimensions
        -- measures
        -- timestamps
        -- metadata
    from source
)

select * from renamed
```

**Schema YAML** — `_<folder>__models.yml`:
```yaml
models:
  - name: model_name
    description: ""
    columns:
      - name: id
        description: "Primary key"
        data_tests:
          - unique
          - not_null
```

### 4. Testing Strategy

Always include:
- `unique` + `not_null` on primary keys
- `accepted_values` on status/enum columns
- `relationships` for foreign keys
- Row count assertions for critical models (use `dbt_utils.expression_is_true`)

### 5. Rules

- Use `{{ ref() }}` and `{{ source() }}`, never hardcoded table names
- Use `{{ adapter.dispatch() }}` for cross-warehouse SQL
- CTE names should be descriptive verbs: `filtered`, `joined`, `aggregated`
- Timestamp columns: `_at` suffix, always cast to the project timezone
- Boolean columns: `is_` or `has_` prefix
- IDs: `<entity>_id` format
- Money: store as integer cents, convert in marts

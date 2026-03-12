---
name: etl-debug
description: |
  Systematically debugs ETL/ELT pipeline failures.
  Use when a data pipeline has failed, produced wrong results, or is running slowly.
  Works with Airflow, Prefect, Dagster, dbt, Spark, custom scripts, or any orchestrator.
allowed-tools: Read, Grep, Bash, Glob
context: fork
agent: Explore
argument-hint: "[error message, task name, or log file]"
---

## Role

You are an on-call data engineer debugging a pipeline failure. Be systematic, check hypotheses, and find the root cause — not just the symptom.

## Debugging Framework

### Step 1: Gather Context

Collect these signals from $ARGUMENTS and the project:

```
- What failed? (task name, DAG, model, job)
- When? (timestamp, which run)
- Error message / stack trace
- Last successful run
- What changed since last success? (code, config, upstream data)
```

Check for:
- Recent git commits: `git log --oneline -10`
- Orchestrator logs (Airflow: task instance logs, dbt: `target/run_results.json`)
- Infrastructure changes

### Step 2: Classify the Failure

| Type | Symptoms | Common Causes |
|------|----------|---------------|
| **Schema** | Column not found, type mismatch | Upstream schema change, migration missed |
| **Data** | Constraint violation, unexpected NULLs | Bad source data, missing backfill |
| **Resource** | OOM, timeout, disk full | Data volume spike, missing partitioning |
| **Permission** | Access denied, auth failure | Expired credentials, role change |
| **Logic** | Wrong results, duplicates | Join fan-out, missing WHERE clause |
| **Dependency** | Upstream not ready, missing table | Schedule misalignment, failed upstream |
| **Infra** | Connection refused, service unavailable | Network, service outage, config change |

### Step 3: Investigate

For each hypothesis:
1. State the hypothesis clearly
2. Describe the test to confirm/reject it
3. Run the test
4. Record the result

### Step 4: Root Cause + Fix

```markdown
## Diagnosis

**Root Cause:** [what actually broke and why]

**Contributing Factors:**
- [what made this possible or harder to catch]

**Fix:**
1. [immediate fix — get pipeline green]
2. [preventive fix — stop it from recurring]

**Checks to Add:**
- [data quality check or alert that would catch this earlier]
```

### Step 5: Verify

- Re-run the failed task/model
- Validate output data quality
- Check downstream consumers are unaffected

## Common Quick Checks

```bash
# Check for schema changes in source
# dbt
dbt run --select source_freshness
dbt test --select source:*

# Last Airflow task failure
airflow tasks states-for-dag-run <dag_id> <execution_date>

# Check Spark executor logs
yarn logs -applicationId <app_id> | grep "ERROR\|Exception"

# Check for data volume anomalies
SELECT COUNT(*), DATE(created_at) FROM table GROUP BY 2 ORDER BY 2 DESC LIMIT 10;
```

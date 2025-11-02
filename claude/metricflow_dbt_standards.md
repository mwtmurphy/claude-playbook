# MetricFlow + dbt Development Standards

**Status**: Strong preference - deviations require justification and approval
**Scope**: Framework-level (applicable to any dbt + MetricFlow project)
**Last Updated**: 2025-11-01

## Purpose

This document defines standards for developing data models and metrics using dbt-core with MetricFlow semantic layer. These patterns apply to any project using dbt + MetricFlow, regardless of business domain.

**Related Standards**:
- [SQL Style](./sql_style.md) - SQL style guide
- [Database Standards](./database_standards.md) - Query patterns and schemas
- [Git Workflow](./git_workflow.md) - Commit workflow

## Core Principles

### 1. Semantic Layer-First Development

**CRITICAL RULE**: When working with metrics, ALWAYS use the MetricFlow semantic layer, not direct table queries.

**Workflow**:
1. **Check existing metrics**: `mf list metrics`
2. **Use existing metrics**: Query via MetricFlow if relevant metric exists
3. **Create new metrics**: Define in semantic model if no relevant metric exists
4. **Never bypass**: Don't query fact/dimension tables directly for metric requests

**Why This Matters**:
- Ensures consistent metric definitions across all users
- Enforces data quality through semantic model validation
- Enables metric governance and lineage tracking
- Prevents duplicate or conflicting metric calculations

**Examples**:
```bash
# ✅ CORRECT: Use semantic layer
mf query --metrics total_revenue --start-time 2025-01-01

# ❌ WRONG: Direct table query for metrics
bq query "SELECT SUM(amount) FROM fct_transactions"
```

**Exception**: Direct queries allowed ONLY for:
- Data validation during metric development
- Schema exploration and source investigation
- Debugging semantic model issues

### 2. Cost-Conscious Development

**CRITICAL**: Always use recent date filters when developing and testing queries.

```bash
# ✅ GOOD: Filter to recent data (1 month)
mf query --metrics total_revenue \
  --start-time 2025-10-01 \
  --end-time 2025-10-31

# ❌ BAD: Query all historical data
mf query --metrics total_revenue \
  --start-time 2020-01-01
```

**Cost Optimization Tips**:
- Use partition/cluster keys in WHERE clauses
- Start with `--limit 10` during exploration
- Use `--explain` to review SQL before running
- Filter to test data subset during development

## MetricFlow Core Concepts

### Semantic Models

**Definition**: Semantic models describe your data in business terms.

**Key Components**:
- **Model Reference**: Points to a dbt model (view or table)
- **Entities**: Join keys (primary, foreign, unique, natural)
- **Dimensions**: Ways to slice/group data (categorical or time)
- **Measures**: Aggregations that become metrics

**Structure**:
```yaml
semantic_models:
  - name: transactions  # Singular form preferred
    description: Customer transaction fact table
    model: ref('fct_transaction')
    defaults:
      agg_time_dimension: transaction_date

    entities:
      - name: transaction
        type: primary
        expr: transaction_id
      - name: customer
        type: foreign
        expr: customer_id

    dimensions:
      - name: transaction_date
        type: time
        type_params:
          time_granularity: day
      - name: status
        type: categorical

    measures:
      - name: transaction_amount
        agg: sum
        expr: amount
      - name: transaction_count
        agg: count
        expr: 1
```

### Entities

**Purpose**: Define relationships between semantic models

**Types**:
- **Primary**: Unique identifier for the model's grain (one per semantic model)
- **Foreign**: Links to another semantic model
- **Unique**: Alternative unique identifier
- **Natural**: Composite key or human-readable identifier

**Best Practice**: Use singular form names and `expr` to map to actual column names

```yaml
entities:
  - name: customer      # Singular, not "customers"
    type: foreign
    expr: customer_id   # Actual column name
```

### Dimensions

**Purpose**: Fields for filtering, grouping, and slicing metrics

**Types**:
- **Categorical**: Non-numeric groupings (status, region, category)
- **Time**: Dates/timestamps for time-series analysis

**Time Dimension Requirements**:
- At least one **primary time dimension** required for semantic models with measures
- Use `type_params` to specify granularity (day, week, month, quarter, year)

```yaml
dimensions:
  - name: order_date
    type: time
    type_params:
      time_granularity: day
    expr: created_at  # Map to actual column

  - name: order_status
    type: categorical
    expr: status
```

**Computed Dimensions**:
```yaml
dimensions:
  - name: is_high_value
    type: categorical
    expr: CASE WHEN amount > 1000 THEN 'High' ELSE 'Standard' END
```

### Measures

**Purpose**: Aggregations that can be used to create metrics

**Aggregation Types**:
- `sum`: Sum of values
- `count`: Count of rows
- `count_distinct`: Count unique values
- `average`: Average of values
- `min` / `max`: Minimum/maximum values
- `median`: Median value

**Best Practices**:
- Use descriptive names that indicate what's being measured
- Always provide `expr` for clarity (use `expr: 1` for counts)
- Add clear descriptions

```yaml
measures:
  - name: total_revenue
    agg: sum
    expr: amount
    description: Total transaction revenue

  - name: transaction_count
    agg: count
    expr: 1
    description: Total number of transactions

  - name: unique_customers
    agg: count_distinct
    expr: customer_id
    description: Count of unique customers
```

### Metrics

**Purpose**: Business metrics derived from measures

**Metric Types**:

**1. Simple**: Direct aggregation from a single measure
```yaml
metrics:
  - name: total_revenue
    description: Sum of all transaction revenue
    type: simple
    label: Total Revenue
    type_params:
      measure: total_revenue
```

**2. Ratio**: Division of two measures
```yaml
metrics:
  - name: average_order_value
    description: Average revenue per order
    type: ratio
    label: AOV
    type_params:
      numerator: total_revenue
      denominator: order_count
```

**3. Derived**: Metric based on other metrics
```yaml
metrics:
  - name: revenue_per_customer
    description: Revenue divided by unique customers
    type: derived
    label: Revenue per Customer
    type_params:
      expr: total_revenue / unique_customers
      metrics:
        - total_revenue
        - unique_customers
```

**4. Cumulative**: Running total over time
```yaml
metrics:
  - name: cumulative_revenue
    description: Running total of revenue over time
    type: cumulative
    label: Cumulative Revenue
    type_params:
      measure: total_revenue
```

## Semantic Model Design Patterns

### Pattern 1: Fact Table Semantic Model

**Use Case**: Transaction-level data with events

```yaml
semantic_models:
  - name: transaction
    model: ref('fct_transaction')
    defaults:
      agg_time_dimension: transaction_date

    entities:
      - name: transaction
        type: primary
        expr: transaction_id
      - name: customer
        type: foreign
        expr: customer_id

    dimensions:
      - name: transaction_date
        type: time
        type_params:
          time_granularity: day
      - name: transaction_type
        type: categorical

    measures:
      - name: transaction_amount
        agg: sum
        expr: amount
      - name: transaction_count
        agg: count
        expr: 1
```

### Pattern 2: Dimension Table Semantic Model

**Use Case**: Customer, product, or other entities

```yaml
semantic_models:
  - name: customer
    model: ref('dim_customer')

    entities:
      - name: customer
        type: primary
        expr: customer_id

    dimensions:
      - name: customer_created_date
        type: time
        type_params:
          time_granularity: day
        expr: created_at
      - name: customer_segment
        type: categorical
        expr: segment

    measures:
      - name: customer_count
        agg: count
        expr: 1
```

### Pattern 3: Cross-Model Metrics (Foreign Keys)

**Benefit**: MetricFlow automatically joins across semantic models

```bash
# MetricFlow will auto-join transactions -> customers via customer entity
mf query --metrics transaction_count --group-by customer__customer_segment
```

## Development Workflow

### Step 1: Identify Data Sources

**Tools**:
- BigQuery Console for schema exploration
- MCP BigQuery tool: `mcp bigquery describe-table project.dataset.table`
- Existing dbt models

**Best Practice**: Query one table at a time to avoid excessive costs

```bash
# ✅ GOOD: Describe single table
mcp bigquery describe-table project.dataset.transactions

# ❌ AVOID: Listing all tables (costly)
bq ls --max_results=1000 project:dataset
```

### Step 2: Add Source to sources.yml

```yaml
sources:
  - name: dataset_name
    database: project_name
    schema: schema_name
    tables:
      - name: transactions
        description: Customer transaction records
        columns:
          - name: transaction_id
            description: Unique transaction identifier
          - name: customer_id
            description: Customer identifier
```

### Step 3: Create dbt Model (if needed)

**Model Naming Conventions**:
- `fct_` prefix: Fact tables/views containing measurable events
- `dim_` prefix: Dimension tables with descriptive attributes
- `stg_` prefix: Staging models (direct source transformations)
- `int_` prefix: Intermediate models (business logic, not exposed)

**CRITICAL**: Use SINGULAR table names (e.g., `dim_customer` not `dim_customers`)

**Example**:
```sql
-- models/marts/core/facts/fct_transaction.sql
-- Fact table for customer transactions
-- References: stg_transactions
-- Last updated: 2025-11-01
--
-- Grain: One row per transaction_id
-- Purpose: Cleaned transaction data with business logic applied

{{ config(
    materialized='view',
    labels={'domain': 'finance', 'layer': 'facts'}
) }}

SELECT
    transaction_id,
    customer_id,
    transaction_date,
    amount,
    status
FROM {{ ref('stg_transaction') }}
WHERE status != 'CANCELLED'
```

### Step 4: Create Semantic Model

Create `models/semantic_models/{name}.yml` following the patterns above.

```yaml
semantic_models:
  - name: transaction
    description: Customer transaction fact table
    model: ref('fct_transaction')
    # ... entities, dimensions, measures
```

### Step 5: Parse and Validate

```bash
# Set profile directory
export DBT_PROFILES_DIR=.

# Parse dbt project (generates semantic manifest)
dbt parse

# List available metrics
mf list metrics

# List dimensions for a specific metric
mf list dimensions --metrics total_revenue

# Validate semantic model
mf validate-configs
```

### Step 6: Test Queries

```bash
# Query a metric
mf query --metrics total_revenue --group-by metric_time__month

# Query multiple metrics with dimensions
mf query \
  --metrics total_revenue,transaction_count \
  --group-by transaction__status \
  --order metric_time

# Query with filters
mf query \
  --metrics total_revenue \
  --group-by metric_time__week \
  --where "{{ Dimension('transaction__status') }} = 'COMPLETED'" \
  --start-time '2024-01-01' \
  --end-time '2024-12-31'

# Explain query (see generated SQL)
mf query \
  --metrics total_revenue \
  --group-by metric_time__month \
  --explain
```

**Cost Optimization**: Always use recent date filters during development

### Step 7: Document and Test

**In-file SQL header**:
```sql
-- [Layer] [type] for [purpose]
-- References: [source tables or upstream models]
-- Last updated: YYYY-MM-DD
--
-- Grain: One row per [entity]
-- Purpose: [Business purpose and key transformation logic]
```

**YAML documentation in `_models.yml`**:
```yaml
version: 2

models:
  - name: fct_transaction
    description: |
      Customer transaction fact table.

      **Grain**: One row per transaction_id
      **Materialization**: view
    config:
      materialized: view
    columns:
      - name: transaction_id
        description: Unique transaction identifier (primary key)
        tests:
          - unique
          - not_null
      - name: customer_id
        description: Customer identifier (foreign key)
        tests:
          - not_null
      - name: amount
        description: Transaction amount in base currency
        tests:
          - not_null
```

### Step 8: Commit Changes

Follow workspace [Git Workflow](./git_workflow.md) standards for commit messages.

## Model Naming Conventions

### SINGULAR Table Names (CRITICAL)

**Rule**: All dbt models MUST use SINGULAR table names

```
✅ CORRECT:
- dim_customer        (one row = one customer)
- fct_transaction     (one row = one transaction)
- int_user            (one row = one user)

❌ WRONG:
- dim_customers
- fct_transactions
- int_users
```

**Rationale**:
1. Table name describes WHAT each row represents (singular entity)
2. Aligns with semantic model naming (entities are singular)
3. Reads naturally: "SELECT * FROM dim_customer WHERE customer_id = 123"

### Directory Structure by Layer

```
models/
├── staging/              # stg_ models (source transformations)
│   ├── _models.yml
│   └── stg_*.sql
├── intermediate/         # int_ models (business logic)
│   ├── _models.yml
│   └── int_*.sql
├── marts/
│   └── core/
│       ├── dimensions/   # dim_ models
│       │   ├── _models.yml
│       │   └── dim_*.sql
│       └── facts/        # fct_ models
│           ├── _models.yml
│           └── fct_*.sql
├── semantic_models/      # MetricFlow semantic models
│   └── *.yml
└── sources.yml           # Source table documentation
```

### Semantic Model Naming

**Convention**: Match underlying dbt model name (singular)

```yaml
# ✅ CORRECT: Semantic model name matches model
semantic_models:
  - name: transaction  # Matches ref('fct_transaction')
    model: ref('fct_transaction')

# ❌ WRONG: Plural semantic model name
semantic_models:
  - name: transactions
    model: ref('fct_transaction')
```

## Documentation Standards

### Required Documentation

**1. In-file SQL header** (ALL models):
```sql
-- [Layer] [type] for [purpose]
-- References: [source tables or upstream models]
-- Last updated: YYYY-MM-DD
--
-- Grain: One row per [entity]
-- Purpose: [Business purpose and key transformation logic]
```

**2. YAML documentation** (ALL models):
- Model-level description with grain and purpose
- Column descriptions for ALL columns
- Data quality tests (see below)

**3. Data Quality Tests**:
- Primary keys: `unique` + `not_null`
- Foreign keys: `not_null` (+ `relationships` if critical)
- Categorical fields: `accepted_values`

**Example**:
```yaml
models:
  - name: fct_transaction
    description: |
      Customer transaction fact table.

      **Grain**: One row per transaction_id
      **Materialization**: view
    config:
      materialized: view
    columns:
      - name: transaction_id
        description: Unique transaction identifier
        tests: [unique, not_null]
      - name: status
        description: Transaction status
        tests:
          - not_null
          - accepted_values:
              values: ['PENDING', 'COMPLETED', 'FAILED']
```

### Testing Models

```bash
# Test all models
dbt test

# Test specific model
dbt test --select fct_transaction

# Test specific column
dbt test --select fct_transaction,column:transaction_id
```

## MetricFlow CLI Reference

### Environment Setup

**CRITICAL**: Always set `DBT_PROFILES_DIR` before running MetricFlow commands

```bash
export DBT_PROFILES_DIR=.
mf query --metrics <metric>
```

### Common Commands

```bash
# List all available metrics
mf list metrics

# List dimensions for a specific metric
mf list dimensions --metrics total_revenue

# List dimension values
mf list dimension-values --metrics total_revenue --dimension status

# Query a metric
mf query --metrics total_revenue --group-by metric_time__day

# Explain query (generate SQL without executing)
mf query --metrics total_revenue --explain

# Validate semantic model configuration
mf validate-configs
```

### Query Syntax

**Basic Query**:
```bash
mf query --metrics <metric_name> --group-by <dimension>
```

**With Time Filter**:
```bash
mf query \
  --metrics total_revenue \
  --group-by metric_time__month \
  --start-time 2024-01-01 \
  --end-time 2024-12-31
```

**With WHERE Filter**:
```bash
# Note: Use {{ Dimension('...') }} template syntax
mf query \
  --metrics total_revenue \
  --where "{{ Dimension('transaction__status') }} = 'COMPLETED'"
```

**Multiple Metrics**:
```bash
mf query \
  --metrics total_revenue,transaction_count,unique_customers \
  --group-by metric_time__week
```

**Output to CSV**:
```bash
mf query \
  --metrics total_revenue \
  --group-by metric_time__day \
  --csv output.csv
```

**Limit Results**:
```bash
mf query \
  --metrics total_revenue \
  --group-by metric_time__day \
  --limit 10
```

## Query Optimization

### 1. Use Recent Date Filters

```bash
# ✅ GOOD: Recent data only
mf query --metrics total_revenue \
  --start-time 2025-10-01

# ❌ BAD: All historical data
mf query --metrics total_revenue \
  --start-time 2020-01-01
```

### 2. Use --limit for Exploration

```bash
# Quick preview
mf query --metrics total_revenue \
  --group-by metric_time__day \
  --limit 10
```

### 3. Use --explain Before Execution

```bash
# Review SQL without running query
mf query --metrics total_revenue \
  --group-by metric_time__month \
  --explain
```

**Why**: Verify partition filters, joins, aggregations before incurring query costs

### 4. Use --csv for Large Results

```bash
# Export to CSV instead of console
mf query --metrics total_revenue \
  --group-by metric_time__day \
  --csv results.csv
```

### 5. Query Multiple Metrics in Single Call

```bash
# ✅ EFFICIENT: One query for multiple metrics
mf query --metrics total_revenue,transaction_count,avg_order_value

# ❌ INEFFICIENT: Separate queries
mf query --metrics total_revenue
mf query --metrics transaction_count
mf query --metrics avg_order_value
```

## Production Data Filtering

### Filtering Internal/Test Activity

**Concept**: Exclude internal testing activity from business metrics.

**Pattern**: Create user-level dev flag dimension

```sql
-- int_user_dev_status.sql
WITH user_alias_mapping AS (
    SELECT
        user_id,
        alias,
        CASE
            WHEN email LIKE '%@yourcompany.com' THEN TRUE
            WHEN email LIKE '%@test.com' THEN TRUE
            ELSE FALSE
        END AS is_dev
    FROM user_aliases
),

final AS (
    -- User is dev if ANY of their aliases is dev
    SELECT
        user_id,
        MAX(IF(is_dev = TRUE, TRUE, FALSE)) AS is_dev
    FROM user_alias_mapping
    GROUP BY user_id
)

SELECT * FROM final
```

**Semantic Model Integration**:
```yaml
dimensions:
  - name: is_dev
    type: categorical
    expr: is_dev
    description: True if user has any internal/test email
```

**Usage in Queries**:
```bash
# Production users only
mf query \
  --metrics total_revenue \
  --where "{{ Dimension('user__is_dev') }} = false"
```

### When to Apply Dev Filter

**✅ Always filter** (business metrics):
- KPI dashboards
- Business reports
- Revenue metrics
- Conversion analysis
- Forecasting

**❌ Never filter** (debugging):
- Data validation
- Schema exploration
- Troubleshooting
- Testing new metrics

**🤔 Conditional** (analysis):
- Compare dev vs production: `--group-by user__is_dev`

## Python Integration

### Pattern: Subprocess Query

**Use Case**: Query MetricFlow from Python/Streamlit dashboards

```python
import subprocess
import tempfile
import pandas as pd
from pathlib import Path

def query_metricflow(metrics, group_by, where_clause, start_time):
    """
    Query MetricFlow via CLI subprocess.

    Args:
        metrics: List of metric names
        group_by: List of dimensions
        where_clause: WHERE filter string
        start_time: Start date (YYYY-MM-DD)

    Returns:
        DataFrame with query results
    """
    project_root = Path(__file__).parent

    # Create temp CSV file
    with tempfile.NamedTemporaryFile(mode='w', suffix='.csv', delete=False) as tmp:
        csv_path = tmp.name

    cmd = [
        "poetry", "run", "mf", "query",
        "--metrics", ",".join(metrics),
        "--group-by", ",".join(group_by),
        "--where", where_clause,
        "--start-time", start_time,
        "--order", group_by[0],
        "--csv", csv_path
    ]

    result = subprocess.run(
        cmd,
        cwd=project_root,
        capture_output=True,
        text=True,
        env={**subprocess.os.environ, "DBT_PROFILES_DIR": "."}
    )

    if result.returncode != 0:
        os.unlink(csv_path)
        raise Exception(f"MetricFlow query failed: {result.stderr}")

    df = pd.read_csv(csv_path)
    os.unlink(csv_path)

    # Convert time columns to datetime
    time_cols = [col for col in df.columns if 'time' in col.lower() or 'date' in col.lower()]
    for col in time_cols:
        df[col] = pd.to_datetime(df[col])

    return df
```

**Alternative**: Use `--explain` to generate SQL, then query database directly (recommended for dashboards)

## Advanced Patterns

### Non-Additive Dimensions

For metrics that shouldn't be aggregated over certain dimensions (e.g., account balances):

```yaml
measures:
  - name: account_balance
    agg: sum
    expr: balance
    non_additive_dimension:
      name: date_day
      window_choice: max  # Use latest balance per time period
```

### Derived Metrics with Complex Logic

```yaml
metrics:
  - name: revenue_growth_rate
    description: Month-over-month revenue growth percentage
    type: derived
    label: Revenue Growth Rate
    type_params:
      expr: (current_month_revenue - prior_month_revenue) / prior_month_revenue * 100
      metrics:
        - current_month_revenue
        - prior_month_revenue
```

### Conversion Metrics (Ratio with Filters)

```yaml
metrics:
  - name: conversion_rate
    description: Percentage of sessions that result in purchase
    type: ratio
    label: Conversion Rate
    type_params:
      numerator:
        name: purchase_sessions
        filter: "{{ Dimension('session__type') }} = 'purchase'"
      denominator:
        name: total_sessions
```

## Common Mistakes & Troubleshooting

### ❌ Mistake: Forgetting DBT_PROFILES_DIR

```bash
# ERROR: "Could not find profile named 'metrics'"
mf query --metrics total_revenue

# FIX: Set DBT_PROFILES_DIR
export DBT_PROFILES_DIR=.
mf query --metrics total_revenue
```

### ❌ Mistake: Wrong Dimension Syntax

```bash
# ERROR: Invalid dimension format
--where "user__is_dev = false"

# FIX: Use Dimension() template syntax
--where "{{ Dimension('user__is_dev') }} = false"
```

### ❌ Mistake: Querying Nonexistent Metric

```bash
# ERROR: "Metric 'revenue' not found"
mf query --metrics revenue

# FIX: List available metrics first
mf list metrics
# Then use correct metric name
mf query --metrics total_revenue
```

### ❌ Mistake: Missing Time Dimension

```bash
# ERROR: Some metrics require time dimensions
mf query --metrics total_revenue

# FIX: Add time dimension
mf query \
  --metrics total_revenue \
  --group-by metric_time__day \
  --start-time 2025-01-01
```

### ❌ Mistake: Using Plural Model Names

```sql
-- ERROR: Inconsistent with semantic model naming
-- models/marts/core/facts/fct_transactions.sql (plural)

-- FIX: Use singular
-- models/marts/core/facts/fct_transaction.sql (singular)
```

```yaml
# Then semantic model matches
semantic_models:
  - name: transaction  # Singular
    model: ref('fct_transaction')  # Singular
```

## Best Practices Checklist

### Semantic Model Design
- [ ] Use singular form for semantic model names
- [ ] Include descriptive `description` fields for all components
- [ ] Use `expr` to explicitly map to column names
- [ ] Define at least one primary time dimension for models with measures
- [ ] Use meaningful entity names that reflect business concepts
- [ ] Document all entities, dimensions, and measures

### Model Development
- [ ] All model names use SINGULAR form
- [ ] Model prefix matches layer: int_, dim_, fct_
- [ ] Models organized in layer-based directories
- [ ] Semantic model names match underlying model names (singular)

### Documentation & Testing
- [ ] All models documented in `_models.yml`
- [ ] Model descriptions include **grain** and **purpose**
- [ ] All columns have clear descriptions
- [ ] In-file SQL headers include: description, references, grain
- [ ] Primary keys tested: unique + not_null
- [ ] Run `dbt test` before committing

### Query Development
- [ ] Start with recent date filters (1 month)
- [ ] Use `--explain` to review SQL before running
- [ ] Set `DBT_PROFILES_DIR` environment variable
- [ ] Use correct dimension syntax: `{{ Dimension('...') }}`
- [ ] Query multiple metrics in single call when possible

### Production Filtering
- [ ] Apply dev filter for business metrics
- [ ] Do NOT apply dev filter for debugging
- [ ] Document when filter is applied (code comments)

## References

### Official Documentation
- [dbt Semantic Layer Docs](https://docs.getdbt.com/docs/build/about-metricflow)
- [MetricFlow Commands Reference](https://docs.getdbt.com/docs/build/metricflow-commands)
- [dbt Best Practices](https://docs.getdbt.com/guides/best-practices)

### Workspace Standards
- [SQL Style](./sql_style.md) - SQL style guide
- [Database Standards](./database_standards.md) - Query patterns
- [Git Workflow](./git_workflow.md) - Commit workflow
- [Testing Standards](./testing_standards.md) - Test patterns

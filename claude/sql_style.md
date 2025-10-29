# SQL Style Standards

**Status**: Strong preference - deviations require justification and approval
**Scope**: SQL code formatting and naming conventions (BigQuery primary dialect)
**Platform**: Optimized for Google BigQuery

## Overview

Consistent SQL style improves readability, maintainability, and collaboration. These standards are designed for BigQuery but are generally applicable to most SQL dialects.

**Why**: Clear, consistent SQL is easier to read, debug, optimize, and maintain across the team.

## Core Principles

### 1. Always Use CTEs (WITH Clauses)

**Every query must use CTE structure with a `final` CTE.**

**Why**: CTEs improve readability, enable testing individual steps, and make queries self-documenting.

```sql
-- Good: Always use CTE structure
WITH final AS (
    -- Get active users

    SELECT
        user_id,
        username,
        email

    FROM users
    WHERE TRUE
          AND is_active = TRUE
)

SELECT *
FROM final
```

```sql
-- Bad: Never use this flat structure
SELECT user_id, username, email
FROM users
WHERE is_active = TRUE
```

### 2. Ban Subqueries

**Never use subqueries - always use CTEs instead.**

**Why**: CTEs are more readable, testable, and maintainable than nested subqueries.

```sql
-- Good: Use CTE
WITH high_value_customers AS (
    SELECT customer_id
    FROM orders
    WHERE TRUE
          AND total_amount > 1000
),

final AS (
    SELECT
        u.user_id,
        u.username

    FROM users                         AS u
         INNER JOIN high_value_customers AS hv ON u.user_id = hv.customer_id
)

SELECT *
FROM final
```

```sql
-- Bad: Subquery (NEVER USE)
SELECT u.user_id, u.username
FROM users u
WHERE u.user_id IN (
    SELECT customer_id
    FROM orders
    WHERE total_amount > 1000
)
```

### 3. WHERE TRUE / HAVING TRUE Pattern

**Always start WHERE and HAVING clauses with TRUE.**

**Why**: Allows easy commenting out of individual conditions without syntax errors.

```sql
WHERE TRUE
      AND is_active = TRUE
      AND created_at >= '2024-01-01'
      AND country IN ('US', 'CA', 'UK')
```

Benefits:
- Comment out any condition without breaking syntax
- Easy to reorder conditions
- Consistent structure across all queries

## Naming Conventions

### All Identifiers: snake_case

**Why**: Consistent with Python, avoids quoting requirements, universally readable.

```sql
-- Good: snake_case for everything
SELECT
    user_id,
    first_name,
    last_name,
    email_address,
    created_at

FROM user_accounts
```

```sql
-- Bad: camelCase or PascalCase
SELECT userId, firstName, lastName
FROM UserAccounts
```

### Table Aliases

**Use short, meaningful 2-character aliases with AS keyword.**

```sql
-- Good: Short, clear aliases
FROM users               AS us
     INNER JOIN orders   AS od ON us.user_id = od.customer_id
     INNER JOIN products AS pr ON od.product_id = pr.product_id
```

**Common alias conventions:**
- `us` - users
- `od` - orders
- `pr` - products
- `ca` - categories
- `oi` - order_items
- `e1`, `e2` - for self-joins

## Keyword Casing

### Keywords: UPPERCASE

```sql
SELECT
    user_id,
    username

FROM users
WHERE TRUE
      AND is_active = TRUE
ORDER BY created_at DESC
LIMIT 100
```

### Identifiers: lowercase

All table names, column names, and aliases in lowercase.

## Formatting and Layout

### Trailing Commas

**Use trailing commas (comma at END of line), not leading commas.**

```sql
-- Good: Trailing commas
SELECT
    user_id,
    username,
    email,
    created_at

FROM users
```

```sql
-- Bad: Leading commas (don't use)
SELECT
    user_id
    , username
    , email
    , created_at
FROM users
```

### Indentation

**Use 4 spaces for indentation (never tabs).**

```sql
SELECT
    user_id,              -- 4 spaces
    username,
    email

FROM users
WHERE TRUE
      AND is_active = TRUE      -- WHERE conditions indented further
      AND email_verified = TRUE
```

### Blank Lines

**Strategic blank lines improve readability.**

```sql
WITH active_users AS (
    -- Get active users

    SELECT
        user_id,
        username,
        email
                            -- Blank line after columns
    FROM users
    WHERE TRUE
          AND is_active = TRUE
),
                            -- Blank line between CTEs
final AS (
    -- Calculate order statistics

    SELECT
        au.user_id,
        au.username,
        COUNT(od.order_id) AS order_count
                            -- Blank line after columns
    FROM active_users      AS au
         LEFT JOIN orders  AS od ON au.user_id = od.customer_id
    GROUP BY ALL
)

SELECT *
FROM final
```

## CTE Structure

### CTE Comments

**Every CTE must have a descriptive comment.**

```sql
WITH active_users AS (
    -- Get active users created after 2024-01-01
    --
    -- Filters for:
    -- - Active status
    -- - Account created in 2024 or later

    SELECT
        user_id,
        username,
        email,
        created_at

    FROM users
    WHERE TRUE
          AND is_active = TRUE
          AND created_at >= '2024-01-01'
),

final AS (
    -- Order active users by creation date descending

    SELECT
        user_id,
        username,
        email

    FROM active_users
    ORDER BY created_at DESC
)

SELECT *
FROM final
```

### Multiple CTEs

**Separate CTEs with comma, blank line, and comment for next CTE.**

```sql
WITH step_one AS (
    -- First transformation

    SELECT ...
),

step_two AS (
    -- Second transformation

    SELECT ...
),

final AS (
    -- Final result set

    SELECT ...
)

SELECT *
FROM final
```

### Final CTE

**Always end with `final AS (...)` followed by `SELECT * FROM final`.**

**Why**: Consistent structure, easy to add intermediate steps, clear endpoint.

## SELECT Clause

### One Column Per Line

```sql
SELECT
    user_id,
    username,
    email,
    first_name,
    last_name,
    created_at,
    updated_at

FROM users
```

### Column Alignment

```sql
-- Simple columns
SELECT
    user_id,
    username,
    email

-- With aliases - align AS keyword
SELECT
    user_id,
    username,
    first_name AS employee_name,
    manager_id AS reports_to

-- With calculations - align AS keyword
SELECT
    order_id,
    total_amount,
    tax_amount,
    total_amount + tax_amount AS grand_total
```

## FROM and JOIN Clauses

### FROM Clause

```sql
-- Simple FROM
FROM users

-- FROM with alias
FROM users AS us
```

### JOIN Alignment

**Indent JOINs with spaces to vertically align JOIN keywords.**

```sql
-- Good: Aligned JOINs
FROM users               AS us
     INNER JOIN orders   AS od ON us.user_id = od.customer_id
     INNER JOIN products AS pr ON od.product_id = pr.product_id
```

### JOIN with Multiple Conditions

**Align AND conditions under ON clause.**

```sql
FROM users            AS us
     LEFT JOIN orders AS od ON us.user_id = od.customer_id
                               AND od.status = 'completed'
                               AND od.order_date >= '2024-01-01'
```

### Self JOINs

```sql
FROM employees           AS e1
     LEFT JOIN employees AS e2 ON e1.manager_id = e2.employee_id
```

## WHERE Clause

### WHERE TRUE Pattern

**Always start with `WHERE TRUE`.**

```sql
WHERE TRUE
      AND is_active = TRUE
      AND email_verified = TRUE
      AND created_at >= '2024-01-01'
      AND country IN ('US', 'CA', 'UK')
```

### Complex Conditions

```sql
WHERE TRUE
      AND us.is_active = TRUE
      AND (
          total_spent >= 1000.00
          OR order_count >= 10
      )
      AND created_at >= '2024-01-01'
```

## GROUP BY

### Use GROUP BY ALL (BigQuery)

**When grouping by all non-aggregate columns, use `GROUP BY ALL`.**

```sql
-- Good: GROUP BY ALL
SELECT
    us.user_id,
    us.username,
    us.email,
    COUNT(od.order_id) AS order_count

FROM users            AS us
     LEFT JOIN orders AS od ON us.user_id = od.customer_id
GROUP BY ALL
```

### Explicit GROUP BY

**When not all columns, list explicitly.**

```sql
GROUP BY customer_id
```

```sql
GROUP BY user_id, order_date
```

## HAVING Clause

### HAVING TRUE Pattern

**Same as WHERE TRUE - start with `HAVING TRUE`.**

```sql
HAVING TRUE
       AND order_count >= 5
       AND total_spent >= 500.00
```

## ORDER BY and LIMIT

```sql
ORDER BY total_spent DESC

ORDER BY last_name, first_name

ORDER BY priority_rank, order_date DESC

LIMIT 100
```

## CASE Statements

### Formatting

**Each WHEN on its own line, THEN heavily indented for alignment.**

```sql
CASE
     WHEN order_count = 0
          THEN 'new'
     WHEN order_count < 5
          THEN 'occasional'
     WHEN order_count < 20
          THEN 'regular'
     ELSE 'loyal'
END                         AS customer_segment
```

### Multiple CASE Statements

```sql
SELECT
    user_id,
    username,
    CASE
         WHEN order_count = 0
              THEN 'new'
         WHEN order_count < 5
              THEN 'occasional'
         ELSE 'regular'
    END                         AS customer_segment,
    CASE
         WHEN total_spent = 0
              THEN 'none'
         WHEN total_spent < 100
              THEN 'low'
         ELSE 'high'
    END                         AS spending_tier

FROM user_summary
```

### CASE in WHERE Clause

```sql
WHERE TRUE
      AND CASE
               WHEN account_type = 'premium'
                    THEN last_login >= DATE_SUB(CURRENT_DATE, INTERVAL 30 DAY)
               WHEN account_type = 'standard'
                    THEN last_login >= DATE_SUB(CURRENT_DATE, INTERVAL 90 DAY)
               ELSE last_login >= DATE_SUB(CURRENT_DATE, INTERVAL 180 DAY)
          END
```

### BANNED: CASE in ORDER BY

**Never use CASE in ORDER BY - calculate field first, then order by it.**

```sql
-- Good: Calculate field, then order
WITH final AS (
    SELECT
        order_id,
        priority,
        CASE priority
             WHEN 'urgent'
                  THEN 1
             WHEN 'high'
                  THEN 2
             ELSE 3
        END                AS priority_rank

    FROM orders
    ORDER BY priority_rank
)

SELECT *
FROM final
```

```sql
-- Bad: CASE in ORDER BY (NEVER DO THIS)
SELECT order_id, priority
FROM orders
ORDER BY CASE priority
              WHEN 'urgent'
                   THEN 1
              ELSE 2
         END
```

## Window Functions

### Simple Window Functions

```sql
ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY order_date DESC) AS order_rank
```

### Complex Window Functions

```sql
SUM(total_amount) OVER (
    PARTITION BY user_id
    ORDER BY order_date
    ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
)                                                    AS cumulative_spent
```

### Multiple Window Functions

```sql
SELECT
    user_id,
    order_id,
    order_date,
    total_amount,
    ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY order_date DESC)                                                  AS order_rank,
    SUM(total_amount) OVER (PARTITION BY user_id ORDER BY order_date ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS cumulative_spent,
    AVG(total_amount) OVER (PARTITION BY user_id)                                                                      AS avg_order_value,
    COUNT(*) OVER (PARTITION BY user_id)                                                                               AS total_orders

FROM orders
WHERE TRUE
      AND status = 'completed'
```

## BigQuery-Specific Features

### IF() Function

**Use IF() for simple conditionals instead of CASE WHEN.**

```sql
-- Good: Use IF() for simple conditions
SELECT
    COUNT(*) AS total_users,
    SUM(IF(is_active = TRUE, 1, 0))      AS active_users,
    SUM(IF(email_verified = TRUE, 1, 0)) AS verified_users

FROM users
```

```sql
-- Less preferred: CASE for simple true/false
SELECT
    SUM(CASE WHEN is_active = TRUE THEN 1 ELSE 0 END) AS active_users

FROM users
```

### Date Functions

**Use BigQuery date functions.**

```sql
-- DATE_SUB for date arithmetic
DATE_SUB(CURRENT_DATE, INTERVAL 30 DAY)
DATE_SUB(CURRENT_DATE, INTERVAL 1 MONTH)
DATE_SUB(CURRENT_DATE, INTERVAL 1 YEAR)

-- DATE_TRUNC for truncation
DATE_TRUNC(order_date, MONTH)
DATE_TRUNC(order_date, WEEK)
DATE_TRUNC(order_date, YEAR)
```

### Example

```sql
WHERE TRUE
      AND order_date >= DATE_SUB(CURRENT_DATE, INTERVAL 90 DAY)
      AND last_login >= DATE_SUB(CURRENT_DATE, INTERVAL 30 DAY)

SELECT
    DATE_TRUNC(order_date, MONTH) AS order_month,
    SUM(total_amount)             AS monthly_revenue

FROM orders
GROUP BY DATE_TRUNC(order_date, MONTH)
```

## INSERT Statements

### Single Row

```sql
INSERT INTO users
    (username, email, password_hash, first_name, last_name)

VALUES
    ('johndoe', 'john@example.com', 'hashed_password', 'John', 'Doe')
```

### Multiple Rows

```sql
INSERT INTO users
    (username, email, password_hash, first_name, last_name)

VALUES
    ('johndoe', 'john@example.com', 'hash1', 'John', 'Doe'),
    ('janedoe', 'jane@example.com', 'hash2', 'Jane', 'Doe'),
    ('bobsmith', 'bob@example.com', 'hash3', 'Bob', 'Smith')
```

### INSERT from SELECT

```sql
INSERT INTO user_archive
    (user_id, username, email, archived_at)

SELECT
    user_id,
    username,
    email,
    CURRENT_TIMESTAMP

FROM users
WHERE TRUE
      AND is_active = FALSE
      AND last_login < DATE_SUB(CURRENT_DATE, INTERVAL 2 YEAR)
```

## UPDATE Statements

### Simple UPDATE

```sql
UPDATE users
SET is_active = FALSE,
    updated_at = CURRENT_TIMESTAMP
WHERE TRUE
      AND last_login < DATE_SUB(CURRENT_DATE, INTERVAL 1 YEAR)
```

### UPDATE with Multiple Conditions

```sql
UPDATE orders
SET status = 'cancelled',
    updated_at = CURRENT_TIMESTAMP
WHERE TRUE
      AND status = 'pending'
      AND created_at < DATE_SUB(CURRENT_DATE, INTERVAL 30 DAY)
      AND total_amount = 0
```

### UPDATE with CASE

```sql
UPDATE products
SET discount_percentage = CASE
                               WHEN stock_quantity = 0
                                    THEN 0
                               WHEN stock_quantity < 10
                                    THEN 5
                               WHEN stock_quantity < 50
                                    THEN 10
                               ELSE 15
                          END,
    updated_at = CURRENT_TIMESTAMP
WHERE TRUE
      AND category = 'electronics'
```

## DELETE Statements

### Simple DELETE

```sql
DELETE FROM sessions
WHERE TRUE
      AND created_at < DATE_SUB(CURRENT_DATE, INTERVAL 30 DAY)
```

### DELETE with Multiple Conditions

```sql
DELETE FROM user_tokens
WHERE TRUE
      AND user_id IN (
          SELECT user_id
          FROM users
          WHERE is_active = FALSE
      )
      AND created_at < DATE_SUB(CURRENT_DATE, INTERVAL 90 DAY)
```

**Note**: For DELETE with complex logic, prefer CTE structure when supported.

## CREATE TABLE

### Column Alignment

**Align column definitions for readability.**

```sql
CREATE TABLE users (
    user_id       SERIAL PRIMARY KEY,
    username      VARCHAR(50) NOT NULL UNIQUE,
    email         VARCHAR(255) NOT NULL UNIQUE,
    password_hash VARCHAR(255) NOT NULL,
    first_name    VARCHAR(100),
    last_name     VARCHAR(100),
    is_active     BOOLEAN NOT NULL DEFAULT TRUE,
    created_at    TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at    TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
)
```

### With Constraints

```sql
CREATE TABLE orders (
    order_id     SERIAL PRIMARY KEY,
    customer_id  INTEGER NOT NULL REFERENCES users(user_id) ON DELETE RESTRICT,
    order_date   DATE NOT NULL DEFAULT CURRENT_DATE,
    total_amount DECIMAL(10, 2) NOT NULL CHECK (total_amount >= 0),
    tax_amount   DECIMAL(10, 2) NOT NULL CHECK (tax_amount >= 0),
    status       VARCHAR(20) NOT NULL DEFAULT 'pending',
    created_at   TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at   TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,

    CONSTRAINT valid_status CHECK (status IN ('pending', 'processing', 'shipped', 'delivered', 'cancelled')),
    CONSTRAINT valid_total  CHECK (total_amount >= tax_amount)
)
```

## SQL File Documentation

### File Header Docstring

**Every SQL file must have a comprehensive docstring.**

```sql
/*
 * File: sql/queries/get_user_engagement.sql
 * Purpose: Calculate user engagement metrics for dashboard reporting
 *
 * Parameters:
 *   - days_back: Number of days to analyze (default: 30)
 *   - min_views: Minimum page views threshold (default: 1)
 *
 * Returns:
 *   - user_id: Unique user identifier
 *   - username: User's username
 *   - page_views: Count of page views in period
 *   - total_session_time: Cumulative session duration in seconds
 *   - interactions: Count of user interactions
 *   - engagement_score: Calculated engagement score (0-100)
 *
 * Usage:
 *   Called by: reports/engagement_dashboard.py
 *   Frequency: Daily batch job at 2:00 AM UTC
 *   Dependencies: users table, page_views table, sessions table
 *
 * Performance:
 *   - Avg execution time: ~200ms on 1M users
 *   - Required indexes:
 *     - idx_users_is_active
 *     - idx_page_views_user_timestamp
 *     - idx_sessions_user_timestamp
 *   - Memory usage: ~50MB
 *
 * Notes:
 *   - Filters to active users only
 *   - Results cached for 1 hour in production
 *   - Engagement score weights: views (30%), time (50%), interactions (20%)
 *
 * Author: team@example.com
 * Created: 2024-01-15
 * Last Modified: 2025-10-11
 */

WITH active_users AS (
    ...
)

SELECT *
FROM final
```

### Inline Comments

**Use inline comments for complex calculations or business logic.**

```sql
SELECT
    us.user_id,
    -- Calculate engagement score using weighted formula:
    -- page_views (30%) + session_time (50%) + interactions (20%)
    (
        COUNT(pv.page_view_id) * 0.3 +
        COALESCE(SUM(st.duration_seconds), 0) * 0.5 +
        COUNT(it.interaction_id) * 0.2
    )                                                AS engagement_score

FROM users AS us
```

## Complete Example

### Complex Real-World Query

```sql
/*
 * File: sql/queries/customer_segmentation_report.sql
 * Purpose: Segment customers by purchase behavior and activity
 * Returns: Customer segments with order statistics
 */

WITH customer_orders AS (
    -- Get order statistics per customer with completed orders

    SELECT
        customer_id,
        COUNT(order_id)   AS order_count,
        SUM(total_amount) AS total_spent,
        MIN(order_date)   AS first_order_date,
        MAX(order_date)   AS last_order_date,
        AVG(total_amount) AS avg_order_value

    FROM orders
    WHERE TRUE
          AND status = 'completed'
    GROUP BY customer_id
),

customer_segments AS (
    -- Calculate segment and activity status per customer

    SELECT
        customer_id,
        order_count,
        total_spent,
        last_order_date,
        CASE
             WHEN order_count >= 20
                  AND total_spent >= 2000
                  THEN 'vip'
             WHEN order_count >= 10
                  AND total_spent >= 1000
                  THEN 'loyal'
             WHEN order_count >= 5
                  THEN 'regular'
             WHEN order_count >= 2
                  THEN 'repeat'
             ELSE 'one_time'
        END                                                                   AS segment,
        CASE
             WHEN last_order_date >= DATE_SUB(CURRENT_DATE, INTERVAL 30 DAY)
                  THEN 'active'
             WHEN last_order_date >= DATE_SUB(CURRENT_DATE, INTERVAL 90 DAY)
                  THEN 'recent'
             WHEN last_order_date >= DATE_SUB(CURRENT_DATE, INTERVAL 180 DAY)
                  THEN 'dormant'
             ELSE 'churned'
        END                                                                   AS activity_status

    FROM customer_orders
),

final AS (
    -- Combine user information with customer segmentation

    SELECT
        us.user_id,
        us.username,
        us.email,
        cs.order_count,
        cs.total_spent,
        cs.segment,
        cs.activity_status

    FROM users                        AS us
         INNER JOIN customer_segments AS cs ON us.user_id = cs.customer_id
    WHERE TRUE
          AND us.is_active = TRUE
    ORDER BY cs.total_spent DESC
)

SELECT *
FROM final
```

## Formatting Tools

### sqlfluff (Optional)

Configure sqlfluff for BigQuery dialect:

```ini
[sqlfluff]
dialect = bigquery
templater = raw
max_line_length = 120

[sqlfluff:indentation]
indent_unit = space
tab_space_size = 4

[sqlfluff:rules:capitalisation.keywords]
capitalisation_policy = upper

[sqlfluff:rules:capitalisation.identifiers]
capitalisation_policy = lower
```

**Note**: Manual formatting may be needed for complex alignment patterns.

## Summary of Banned Patterns

**Never use these patterns:**

1. ❌ **Subqueries** - Always use CTEs instead
2. ❌ **CASE in ORDER BY** - Calculate field first, then order by it
3. ❌ **Queries without CTE structure** - Every query needs `WITH final AS (...)`
4. ❌ **WHERE without TRUE** - Always use `WHERE TRUE AND condition`
5. ❌ **Leading commas** - Use trailing commas only
6. ❌ **Missing CTE comments** - Every CTE needs a descriptive comment

## Related Standards

- See `database_standards.md` for SQL file organization and query optimization
- See `python_style.md` for Python code that generates SQL
- See `documentation_standards.md` for general documentation requirements

---

**Last Updated**: 2025-10-11
**Status**: Strong preference - deviations require justification
**Primary Dialect**: Google BigQuery

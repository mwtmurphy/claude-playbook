# Database Standards

**Status**: Strong preference - deviations require justification and approval
**Scope**: SQL and database conventions for all projects

## Overview

Consistent database practices improve maintainability, performance, and reliability. These standards apply to SQL queries, schema design, and database interactions.

**Why**: Well-organized database code is easier to optimize, debug, and evolve.

## SQL File Organization

### Separate SQL from Python Code

**Why**: SQL files are easier to read, test, and optimize independently.

```
project/
├── sql/
│   ├── queries/           # SELECT queries
│   │   ├── users/
│   │   │   ├── get_active_users.sql
│   │   │   ├── get_user_by_email.sql
│   │   │   └── search_users.sql
│   │   └── orders/
│   │       ├── get_recent_orders.sql
│   │       └── get_order_details.sql
│   ├── schemas/           # Table definitions
│   │   ├── users.sql
│   │   ├── orders.sql
│   │   └── products.sql
│   └── migrations/        # Database migrations
│       ├── 001_create_users_table.sql
│       ├── 002_add_email_index.sql
│       └── 003_add_orders_table.sql
```

### Load SQL from Files

```python
from pathlib import Path

class UserRepository:
    """Repository for user data operations."""

    def __init__(self, db_connection):
        self.db = db_connection
        self.queries_dir = Path(__file__).parent / "sql" / "queries" / "users"

    def get_active_users(self) -> list[dict]:
        """Get all active users."""
        query = (self.queries_dir / "get_active_users.sql").read_text()
        return self.db.execute(query).fetchall()

    def get_user_by_email(self, email: str) -> dict | None:
        """Get user by email address."""
        query = (self.queries_dir / "get_user_by_email.sql").read_text()
        result = self.db.execute(query, {"email": email}).fetchone()
        return dict(result) if result else None
```

## Query Standards

### Use Named Parameters

**Why**: More readable, prevents SQL injection, easier to maintain.

```sql
-- Good: Named parameters
WITH final AS (
    SELECT
        user_id,
        username,
        email,
        created_at

    FROM users
    WHERE TRUE
          AND email = :email
          AND is_active = :is_active
          AND created_at >= :since_date
)

SELECT *
FROM final
```

```python
# Python usage with named parameters
result = db.execute(
    query,
    {
        "email": "user@example.com",
        "is_active": True,
        "since_date": "2024-01-01"
    }
)
```

```sql
-- Bad: Positional parameters
SELECT user_id, username, email
FROM users
WHERE email = ? AND is_active = ? AND created_at >= ?;
```

### Query Formatting

```sql
-- Good: Readable formatting
WITH final AS (
    SELECT
        us.user_id,
        us.username,
        us.email,
        COUNT(od.order_id) AS order_count,
        SUM(od.total_amount) AS total_spent

    FROM users             AS us
         LEFT JOIN orders  AS od ON us.user_id = od.customer_id
                                     AND od.status = 'completed'
    WHERE TRUE
          AND us.is_active = TRUE
          AND us.created_at >= :since_date
    GROUP BY
        us.user_id,
        us.username,
        us.email
    HAVING TRUE
           AND COUNT(od.order_id) > :min_orders
    ORDER BY total_spent DESC
    LIMIT :limit_count
)

SELECT *
FROM final


-- Bad: Hard to read
SELECT u.user_id,u.username,u.email,COUNT(o.order_id),SUM(o.total_amount) FROM users u LEFT JOIN orders o ON u.user_id=o.customer_id WHERE u.is_active=TRUE GROUP BY u.user_id ORDER BY SUM(o.total_amount) DESC;
```

### Document Complex Queries

```sql
-- Get user engagement metrics for dashboard
-- Calculates: page views, session time, interactions for last 30 days
-- Performance: ~200ms on 1M users (indexed on user_id, timestamp)
-- Used by: reports/engagement_dashboard.py
-- Parameters:
--   :days_back (int) - Number of days to look back
--   :min_views (int) - Minimum page views threshold

WITH final AS (
    SELECT
        us.user_id,
        us.username,
        us.email,
        -- Aggregate engagement metrics
        COUNT(DISTINCT pv.page_view_id) AS page_views,
        COALESCE(SUM(ss.duration_seconds), 0) AS total_session_time,
        COUNT(DISTINCT ia.interaction_id) AS interactions

    FROM users                AS us
         LEFT JOIN page_views AS pv ON us.user_id = pv.user_id
                                       AND pv.viewed_at >= CURRENT_DATE - INTERVAL ':days_back days'
         LEFT JOIN sessions   AS ss ON us.user_id = ss.user_id
                                       AND ss.started_at >= CURRENT_DATE - INTERVAL ':days_back days'
         LEFT JOIN interactions AS ia ON us.user_id = ia.user_id
                                         AND ia.created_at >= CURRENT_DATE - INTERVAL ':days_back days'
    WHERE TRUE
          AND us.is_active = TRUE
    GROUP BY
        us.user_id,
        us.username,
        us.email
    HAVING TRUE
           AND COUNT(DISTINCT pv.page_view_id) >= :min_views
    ORDER BY page_views DESC
)

SELECT *
FROM final
```

## Schema Standards

### Table Naming

```sql
-- Good: Descriptive, plural table names
CREATE TABLE users (
    user_id SERIAL PRIMARY KEY,
    username VARCHAR(50) NOT NULL,
    email VARCHAR(255) NOT NULL,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE customer_orders (
    order_id SERIAL PRIMARY KEY,
    customer_id INTEGER NOT NULL REFERENCES users(user_id),
    order_date DATE NOT NULL,
    total_amount DECIMAL(10, 2) NOT NULL
);


-- Bad: Abbreviated or unclear names
CREATE TABLE usr (  -- Too abbreviated
    id INT PRIMARY KEY
);

CREATE TABLE data (  -- Too generic
    id INT PRIMARY KEY
);
```

### Column Naming

```sql
-- Good: Clear, descriptive column names
CREATE TABLE products (
    product_id SERIAL PRIMARY KEY,
    product_name VARCHAR(200) NOT NULL,
    description TEXT,
    unit_price DECIMAL(10, 2) NOT NULL,
    stock_quantity INTEGER NOT NULL DEFAULT 0,
    is_active BOOLEAN NOT NULL DEFAULT TRUE,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);


-- Bad: Unclear or inconsistent names
CREATE TABLE products (
    id INT PRIMARY KEY,  -- Inconsistent naming (not product_id)
    name VARCHAR(200),   -- Ambiguous
    desc TEXT,           -- Abbreviated
    price DECIMAL,       -- Ambiguous (unit price? total?)
    qty INT,             -- Abbreviated
    active BOOL,         -- Inconsistent (not is_active)
    created TIMESTAMP,   -- Inconsistent (not created_at)
    modified TIMESTAMP   -- Inconsistent (not updated_at)
);
```

### Use Constraints

**Why**: Database-level constraints prevent invalid data.

```sql
CREATE TABLE users (
    user_id SERIAL PRIMARY KEY,
    username VARCHAR(50) NOT NULL UNIQUE,
    email VARCHAR(255) NOT NULL UNIQUE,
    age INTEGER CHECK (age >= 18 AND age <= 120),
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,

    CONSTRAINT valid_email CHECK (email ~* '^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$')
);

CREATE TABLE orders (
    order_id SERIAL PRIMARY KEY,
    customer_id INTEGER NOT NULL REFERENCES users(user_id) ON DELETE RESTRICT,
    order_date DATE NOT NULL DEFAULT CURRENT_DATE,
    total_amount DECIMAL(10, 2) NOT NULL CHECK (total_amount >= 0),
    status VARCHAR(20) NOT NULL DEFAULT 'pending',

    CONSTRAINT valid_status CHECK (status IN ('pending', 'processing', 'shipped', 'delivered', 'cancelled'))
);
```

## Indexing Strategy

### Index Frequently Queried Columns

```sql
-- Primary key automatically indexed
CREATE TABLE users (
    user_id SERIAL PRIMARY KEY,
    username VARCHAR(50) NOT NULL,
    email VARCHAR(255) NOT NULL,
    created_at TIMESTAMP NOT NULL
);

-- Add indexes for frequently queried columns
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_created_at ON users(created_at);

-- Composite index for common query patterns
CREATE INDEX idx_users_active_created ON users(is_active, created_at)
WHERE is_active = TRUE;  -- Partial index


-- Foreign key index for join performance
CREATE TABLE orders (
    order_id SERIAL PRIMARY KEY,
    customer_id INTEGER NOT NULL REFERENCES users(user_id),
    order_date DATE NOT NULL
);

CREATE INDEX idx_orders_customer_id ON orders(customer_id);
CREATE INDEX idx_orders_date ON orders(order_date);
```

### Index Naming Convention

```
idx_<table>_<column(s)>_<optional_descriptor>

Examples:
- idx_users_email
- idx_orders_customer_id
- idx_products_category_price
- idx_users_active_created (for is_active, created_at)
```

## Query Optimization

### Avoid SELECT *

**Why**: Fetches unnecessary data, breaks when schema changes.

```sql
-- Good: Explicit columns in CTE
WITH final AS (
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


-- Bad: SELECT * without CTE and structure
SELECT * FROM users WHERE is_active = TRUE;
```

### Use Appropriate JOINs

```sql
-- Good: INNER JOIN when you need matching rows
WITH final AS (
    SELECT
        od.order_id,
        od.total_amount,
        us.username,
        us.email

    FROM orders           AS od
         INNER JOIN users AS us ON od.customer_id = us.user_id
    WHERE TRUE
          AND od.status = 'pending'
)

SELECT *
FROM final


-- Good: LEFT JOIN when you need all left rows
WITH final AS (
    SELECT
        us.user_id,
        us.username,
        COUNT(od.order_id) AS order_count

    FROM users            AS us
         LEFT JOIN orders AS od ON us.user_id = od.customer_id
    GROUP BY
        us.user_id,
        us.username
)

SELECT *
FROM final


-- Bad: Unnecessary LEFT JOIN when INNER would suffice
SELECT
    o.order_id,
    u.username
FROM orders o
LEFT JOIN users u ON o.customer_id = u.user_id;  -- Orders always have users!
```

### Prevent N+1 Queries

**Why**: N+1 queries cause performance issues with large datasets.

```python
# Bad: N+1 query problem
users = db.execute("SELECT user_id, username FROM users").fetchall()
for user in users:
    # Separate query for each user!
    orders = db.execute(
        "SELECT * FROM orders WHERE customer_id = :user_id",
        {"user_id": user["user_id"]}
    ).fetchall()
    print(f"{user['username']}: {len(orders)} orders")


# Good: Single query with JOIN
results = db.execute("""
    WITH final AS (
        SELECT
            us.user_id,
            us.username,
            COUNT(od.order_id) AS order_count

        FROM users            AS us
             LEFT JOIN orders AS od ON us.user_id = od.customer_id
        GROUP BY
            us.user_id,
            us.username
    )

    SELECT *
    FROM final
""").fetchall()

for row in results:
    print(f"{row['username']}: {row['order_count']} orders")
```

### Use EXPLAIN for Complex Queries

```sql
-- Analyze query performance
EXPLAIN ANALYZE
WITH final AS (
    SELECT
        us.user_id,
        us.username,
        COUNT(od.order_id) AS order_count

    FROM users            AS us
         LEFT JOIN orders AS od ON us.user_id = od.customer_id
    WHERE TRUE
          AND us.created_at >= '2024-01-01'
    GROUP BY
        us.user_id,
        us.username
    ORDER BY order_count DESC
    LIMIT 100
)

SELECT *
FROM final
```

## Migration Standards

### Migration Naming

```
<sequence>_<description>.sql

Examples:
001_create_users_table.sql
002_add_email_index.sql
003_add_orders_table.sql
004_alter_users_add_phone.sql
```

### Migration Structure

```sql
-- Migration: 001_create_users_table.sql
-- Description: Create users table with basic authentication fields
-- Author: developer@example.com
-- Date: 2025-10-11

-- Up Migration
CREATE TABLE users (
    user_id SERIAL PRIMARY KEY,
    username VARCHAR(50) NOT NULL UNIQUE,
    email VARCHAR(255) NOT NULL UNIQUE,
    password_hash VARCHAR(255) NOT NULL,
    is_active BOOLEAN NOT NULL DEFAULT TRUE,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_username ON users(username);

-- Down Migration (commented, but available if needed)
-- DROP TABLE IF EXISTS users;
```

### Make Migrations Reversible

```sql
-- Up: Add column
ALTER TABLE users ADD COLUMN phone_number VARCHAR(20);

-- Down: Remove column (in separate down migration if needed)
-- ALTER TABLE users DROP COLUMN phone_number;
```

### Test Migrations

```python
def test_user_table_migration(test_db):
    """Test that user table migration creates correct schema."""
    # Run migration
    migration = Path("sql/migrations/001_create_users_table.sql").read_text()
    test_db.execute(migration)

    # Verify table exists
    result = test_db.execute("""
        WITH final AS (
            SELECT
                column_name,
                data_type

            FROM information_schema.columns
            WHERE TRUE
                  AND table_name = 'users'
        )

        SELECT *
        FROM final
    """).fetchall()

    columns = {row["column_name"]: row["data_type"] for row in result}

    assert "user_id" in columns
    assert "username" in columns
    assert "email" in columns
    assert columns["user_id"] == "integer"
```

## Connection Management

### Use Connection Pooling

**Why**: Reduces overhead of creating connections, improves performance.

```python
import psycopg2.pool

class DatabasePool:
    """Database connection pool manager."""

    def __init__(self, connection_string: str, min_connections: int = 2, max_connections: int = 10):
        self.pool = psycopg2.pool.ThreadedConnectionPool(
            min_connections,
            max_connections,
            connection_string
        )

    def get_connection(self):
        """Get a connection from the pool."""
        return self.pool.getconn()

    def return_connection(self, connection):
        """Return connection to pool."""
        self.pool.putconn(connection)

    def close_all(self):
        """Close all connections in pool."""
        self.pool.closeall()


# Usage
db_pool = DatabasePool("postgresql://user:pass@localhost/dbname")

def query_users():
    conn = db_pool.get_connection()
    try:
        cursor = conn.cursor()
        cursor.execute("""
            WITH final AS (
                SELECT
                    user_id,
                    username,
                    email

                FROM users
            )

            SELECT *
            FROM final
        """)
        results = cursor.fetchall()
        return results
    finally:
        db_pool.return_connection(conn)
```

### Context Managers for Connections

```python
from contextlib import contextmanager

@contextmanager
def get_db_connection(pool):
    """Context manager for database connections."""
    connection = pool.get_connection()
    try:
        yield connection
    finally:
        pool.return_connection(connection)


# Usage
with get_db_connection(db_pool) as conn:
    cursor = conn.cursor()
    cursor.execute("""
        WITH final AS (
            SELECT
                user_id,
                username,
                email

            FROM users
        )

        SELECT *
        FROM final
    """)
    results = cursor.fetchall()
# Connection automatically returned to pool
```

## Transaction Guidelines

### Use Transactions for Multiple Operations

```python
def create_order_with_items(order_data: dict, items: list[dict]) -> int:
    """Create order and items in a transaction."""
    with get_db_connection(db_pool) as conn:
        try:
            conn.autocommit = False

            # Insert order
            cursor = conn.cursor()
            cursor.execute("""
                INSERT INTO orders (customer_id, total_amount, status)
                VALUES (:customer_id, :total_amount, :status)
                RETURNING order_id
            """, order_data)
            order_id = cursor.fetchone()["order_id"]

            # Insert order items
            for item in items:
                cursor.execute("""
                    INSERT INTO order_items (order_id, product_id, quantity, price)
                    VALUES (:order_id, :product_id, :quantity, :price)
                """, {**item, "order_id": order_id})

            conn.commit()
            return order_id

        except Exception as e:
            conn.rollback()
            logger.error("order_creation_failed", error=str(e))
            raise
        finally:
            conn.autocommit = True
```

## Related Standards

- See `sql_style.md` for SQL formatting
- See `architecture_patterns.md` for repository pattern
- See `error_handling.md` for database error handling
- See `performance_considerations.md` for query optimization

---

**Last Updated**: 2025-10-11
**Status**: Strong preference - deviations require justification

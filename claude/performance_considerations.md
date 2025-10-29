# Performance Considerations

**Status**: Strong preference - deviations require justification and approval
**Scope**: Performance best practices for Python and SQL

## Overview

Performance optimization should be data-driven and purposeful. Premature optimization wastes time, but ignoring performance leads to scaling issues.

**Why**: Measure first, optimize strategically, validate improvements.

## Core Principles

### Measure Before Optimizing

**Why**: Optimization without measurement is guesswork.

```python
import time
from functools import wraps

def measure_time(func):
    """Decorator to measure function execution time."""
    @wraps(func)
    def wrapper(*args, **kwargs):
        start = time.perf_counter()
        result = func(*args, **kwargs)
        duration = time.perf_counter() - start
        logger.info(
            "function_execution",
            function=func.__name__,
            duration_ms=duration * 1000
        )
        return result
    return wrapper


@measure_time
def process_large_dataset(data: list) -> list:
    """Process large dataset."""
    return [transform(item) for item in data]
```

### Premature Optimization vs Deliberate Design

**Premature optimization**: Optimizing without evidence of need
**Deliberate design**: Making informed choices based on requirements

```python
# Good: Deliberate design choice with justification
def get_active_users(limit: int = 100) -> list[User]:
    """Get active users with pagination.

    Uses limit to prevent loading entire user table into memory.
    Typical use case expects <1000 active users at any time.
    """
    query = "SELECT * FROM users WHERE is_active = TRUE LIMIT :limit"
    return db.execute(query, {"limit": limit}).fetchall()


# Premature optimization: Complex caching without evidence of need
from functools import lru_cache

@lru_cache(maxsize=10000)  # Overkill for rare lookups
def get_config_value(key: str) -> str:
    """Get configuration value."""
    return config.get(key)
```

## Algorithm Complexity

### Be Aware of Big O Complexity

**Why**: Algorithm choice has bigger impact than micro-optimizations.

```python
# Good: O(n) - Linear search in dictionary
def find_user_by_id(users_dict: dict[int, User], user_id: int) -> User | None:
    """Find user by ID - O(1) average case."""
    return users_dict.get(user_id)


# Bad: O(n) - Linear search when O(1) is possible
def find_user_by_id(users_list: list[User], user_id: int) -> User | None:
    """Find user by ID - O(n) search."""
    for user in users_list:
        if user.id == user_id:
            return user
    return None


# Good: O(n log n) - Use built-in sort
def sort_users(users: list[User]) -> list[User]:
    """Sort users by name."""
    return sorted(users, key=lambda u: u.name)


# Bad: O(n²) - Bubble sort when better options exist
def sort_users(users: list[User]) -> list[User]:
    """Sort users by name."""
    n = len(users)
    for i in range(n):
        for j in range(0, n - i - 1):
            if users[j].name > users[j + 1].name:
                users[j], users[j + 1] = users[j + 1], users[j]
    return users
```

### Common Complexity Classes

| Complexity | Description | Example |
|------------|-------------|---------|
| O(1) | Constant | Dictionary lookup, array access |
| O(log n) | Logarithmic | Binary search |
| O(n) | Linear | List iteration, linear search |
| O(n log n) | Log-linear | Efficient sorting (merge, quick) |
| O(n²) | Quadratic | Nested loops over same data |
| O(2ⁿ) | Exponential | Recursive Fibonacci (naive) |

## Python Performance

### Use List Comprehensions Over Loops

**Why**: List comprehensions are faster and more readable.

```python
# Good: List comprehension (faster)
squared = [x ** 2 for x in numbers]

# Good: Generator expression (memory efficient for large data)
squared_gen = (x ** 2 for x in numbers)

# Slower: Explicit loop with append
squared = []
for x in numbers:
    squared.append(x ** 2)
```

### Use Built-in Functions and Libraries

**Why**: Built-ins are implemented in C and highly optimized.

```python
# Good: Use built-in sum
total = sum(numbers)

# Slower: Manual summation
total = 0
for num in numbers:
    total += num


# Good: Use collections.Counter
from collections import Counter
word_counts = Counter(words)

# Slower: Manual counting
word_counts = {}
for word in words:
    word_counts[word] = word_counts.get(word, 0) + 1
```

### Avoid Repeated Attribute Lookups

**Why**: Attribute lookup has overhead in tight loops.

```python
# Good: Cache attribute in local variable
def process_items(items: list) -> int:
    """Process items efficiently."""
    append = results.append  # Cache method
    for item in items:
        append(transform(item))
    return len(results)


# Slower: Repeated attribute lookups
def process_items(items: list) -> int:
    """Process items."""
    for item in items:
        results.append(transform(item))  # Looks up 'append' each iteration
    return len(results)
```

### Use Generators for Large Datasets

**Why**: Generators don't load entire dataset into memory.

```python
# Good: Generator for large file
def read_large_file(file_path: Path):
    """Read large file line by line."""
    with open(file_path) as f:
        for line in f:
            yield line.strip()


# Process without loading entire file
for line in read_large_file(large_file):
    process_line(line)


# Bad: Loads entire file into memory
def read_large_file(file_path: Path) -> list[str]:
    """Read large file into list."""
    with open(file_path) as f:
        return [line.strip() for line in f]  # All in memory!
```

## Memory Management

### Be Mindful of Memory Usage

```python
# Good: Process in chunks
def process_large_dataset(file_path: Path, chunk_size: int = 1000):
    """Process large dataset in chunks."""
    chunk = []
    with open(file_path) as f:
        for line in f:
            chunk.append(line)
            if len(chunk) >= chunk_size:
                process_chunk(chunk)
                chunk = []  # Release memory
        if chunk:
            process_chunk(chunk)  # Process remaining


# Bad: Load everything into memory
def process_large_dataset(file_path: Path):
    """Process large dataset."""
    with open(file_path) as f:
        all_data = f.readlines()  # Could be gigabytes!
    process_chunk(all_data)
```

### Use __slots__ for Many Instances

**Why**: Reduces memory overhead for classes with many instances.

```python
# Good: Use __slots__ for memory-intensive classes
class Point:
    """Point in 2D space."""
    __slots__ = ['x', 'y']

    def __init__(self, x: float, y: float):
        self.x = x
        self.y = y


# Creating millions of points uses less memory
points = [Point(i, i * 2) for i in range(1_000_000)]


# Without __slots__: Each instance has __dict__ overhead
class Point:
    def __init__(self, x: float, y: float):
        self.x = x
        self.y = y
```

## SQL Performance

### Query Optimization

```sql
-- Good: Use indexes, avoid functions on indexed columns
SELECT user_id, username, email
FROM users
WHERE created_at >= '2024-01-01'  -- Index can be used
    AND is_active = TRUE
LIMIT 100;


-- Bad: Function on indexed column prevents index usage
SELECT user_id, username, email
FROM users
WHERE DATE(created_at) >= '2024-01-01'  -- Can't use index!
    AND is_active = TRUE;
```

### Use EXISTS Instead of IN for Large Sets

**Why**: EXISTS can short-circuit, IN loads all values.

```sql
-- Good: Use EXISTS for large subqueries
SELECT u.user_id, u.username
FROM users u
WHERE EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.customer_id = u.user_id
        AND o.status = 'completed'
);


-- Slower: IN with large result set
SELECT u.user_id, u.username
FROM users u
WHERE u.user_id IN (
    SELECT DISTINCT customer_id
    FROM orders
    WHERE status = 'completed'
);
```

### Avoid N+1 Query Problem

**Why**: Multiple queries are much slower than one JOIN.

```python
# Bad: N+1 queries
users = db.execute("SELECT user_id, username FROM users").fetchall()
for user in users:
    orders = db.execute(
        "SELECT * FROM orders WHERE customer_id = :user_id",
        {"user_id": user["user_id"]}
    ).fetchall()
    user["orders"] = orders


# Good: Single query with JOIN
results = db.execute("""
    SELECT
        u.user_id,
        u.username,
        JSON_AGG(
            JSON_BUILD_OBJECT(
                'order_id', o.order_id,
                'total', o.total_amount
            )
        ) AS orders
    FROM users u
    LEFT JOIN orders o ON u.user_id = o.customer_id
    GROUP BY u.user_id, u.username
""").fetchall()
```

### Use Connection Pooling

**Why**: Creating connections is expensive.

```python
# Good: Use connection pool
from psycopg2.pool import ThreadedConnectionPool

pool = ThreadedConnectionPool(
    minconn=2,
    maxconn=10,
    dsn="postgresql://user:pass@localhost/db"
)

def query_users():
    conn = pool.getconn()
    try:
        cursor = conn.cursor()
        cursor.execute("SELECT * FROM users")
        return cursor.fetchall()
    finally:
        pool.putconn(conn)


# Bad: Create new connection each time
def query_users():
    conn = psycopg2.connect("postgresql://user:pass@localhost/db")
    cursor = conn.cursor()
    cursor.execute("SELECT * FROM users")
    result = cursor.fetchall()
    conn.close()
    return result
```

## Caching Strategies

### Cache Expensive Computations

**Why**: Avoid recomputing identical results.

```python
from functools import lru_cache

# Good: Cache expensive pure functions
@lru_cache(maxsize=128)
def calculate_fibonacci(n: int) -> int:
    """Calculate Fibonacci number with memoization."""
    if n < 2:
        return n
    return calculate_fibonacci(n - 1) + calculate_fibonacci(n - 2)


# Application-level caching
class UserService:
    def __init__(self, cache, repository):
        self.cache = cache
        self.repository = repository

    def get_user(self, user_id: int) -> User | None:
        """Get user with caching."""
        cache_key = f"user:{user_id}"

        # Try cache first
        cached = self.cache.get(cache_key)
        if cached:
            logger.debug("cache_hit", user_id=user_id)
            return cached

        # Fallback to database
        user = self.repository.get(user_id)
        if user:
            self.cache.set(cache_key, user, ttl=300)  # 5 min TTL

        return user
```

### Cache Invalidation

**Why**: Stale caches serve incorrect data.

```python
class UserService:
    """User service with cache invalidation."""

    def update_user(self, user_id: int, data: dict) -> User:
        """Update user and invalidate cache."""
        user = self.repository.update(user_id, data)

        # Invalidate cache after update
        cache_key = f"user:{user_id}"
        self.cache.delete(cache_key)

        logger.info("cache_invalidated", user_id=user_id)
        return user
```

## Profiling

### Profile to Find Bottlenecks

**Why**: Profile before optimizing to identify actual bottlenecks.

```python
# Using cProfile
import cProfile
import pstats

profiler = cProfile.Profile()
profiler.enable()

# Code to profile
process_large_dataset(data)

profiler.disable()
stats = pstats.Stats(profiler)
stats.sort_stats('cumulative')
stats.print_stats(10)  # Top 10 functions


# Using line_profiler for detailed line-by-line profiling
# Install: pip install line_profiler
from line_profiler import LineProfiler

lp = LineProfiler()
lp.add_function(process_large_dataset)
lp.enable()

process_large_dataset(data)

lp.disable()
lp.print_stats()
```

### Memory Profiling

```python
# Using memory_profiler
# Install: pip install memory-profiler
from memory_profiler import profile

@profile
def process_data(data: list) -> list:
    """Process data with memory profiling."""
    results = []
    for item in data:
        results.append(transform(item))
    return results


# Run with: python -m memory_profiler script.py
```

## Concurrency

### Use Async for I/O-Bound Tasks

**Why**: Async handles many I/O operations efficiently.

```python
import asyncio
import aiohttp

# Good: Async for concurrent HTTP requests
async def fetch_user_data(user_ids: list[int]) -> list[dict]:
    """Fetch multiple users concurrently."""
    async with aiohttp.ClientSession() as session:
        tasks = [fetch_user(session, user_id) for user_id in user_ids]
        return await asyncio.gather(*tasks)

async def fetch_user(session, user_id: int) -> dict:
    """Fetch single user."""
    async with session.get(f"{API_URL}/users/{user_id}") as response:
        return await response.json()


# Sequential (slow) version
def fetch_user_data(user_ids: list[int]) -> list[dict]:
    """Fetch users sequentially."""
    results = []
    for user_id in user_ids:
        response = requests.get(f"{API_URL}/users/{user_id}")
        results.append(response.json())
    return results
```

### Use Threading/Multiprocessing for CPU-Bound Tasks

```python
from concurrent.futures import ProcessPoolExecutor
from multiprocessing import cpu_count

# Good: Use multiprocessing for CPU-intensive work
def process_large_dataset(data: list) -> list:
    """Process data using multiple CPU cores."""
    chunk_size = len(data) // cpu_count()
    chunks = [data[i:i + chunk_size] for i in range(0, len(data), chunk_size)]

    with ProcessPoolExecutor(max_workers=cpu_count()) as executor:
        results = executor.map(process_chunk, chunks)

    return list(results)


def process_chunk(chunk: list) -> list:
    """Process a chunk of data."""
    return [expensive_computation(item) for item in chunk]
```

## Benchmarking

### Compare Implementation Options

```python
import timeit

# Compare different implementations
def benchmark_implementations():
    """Benchmark different list creation methods."""
    # List comprehension
    time_comprehension = timeit.timeit(
        '[x**2 for x in range(1000)]',
        number=10000
    )

    # Map with lambda
    time_map = timeit.timeit(
        'list(map(lambda x: x**2, range(1000)))',
        number=10000
    )

    # Explicit loop
    time_loop = timeit.timeit(
        '''
result = []
for x in range(1000):
    result.append(x**2)
        ''',
        number=10000
    )

    print(f"Comprehension: {time_comprehension:.4f}s")
    print(f"Map: {time_map:.4f}s")
    print(f"Loop: {time_loop:.4f}s")
```

## Performance Checklist

Before optimizing:
- [ ] Profile to identify actual bottlenecks
- [ ] Measure current performance (baseline)
- [ ] Set target performance goals
- [ ] Verify optimization improves performance
- [ ] Ensure optimization doesn't hurt readability excessively

Common optimization targets:
- [ ] Algorithm complexity (biggest impact)
- [ ] Database query optimization
- [ ] Caching frequently accessed data
- [ ] Reducing memory allocations
- [ ] Async for I/O-bound operations
- [ ] Multiprocessing for CPU-bound operations

## Related Standards

- See `database_standards.md` for SQL optimization
- See `architecture_patterns.md` for efficient design
- See `testing_standards.md` for performance testing

---

**Last Updated**: 2025-10-11
**Status**: Strong preference - deviations require justification

# Documentation Standards

**Status**: Strong preference - deviations require justification and approval
**Scope**: Documentation requirements for Python code and projects
**Last Updated**: 2025-10-31

---

## Overview

Good documentation makes code accessible, maintainable, and collaborative. Documentation should explain the "why" more than the "what" - code should be self-documenting for the "what".

**Why**: Well-documented code reduces onboarding time, prevents misuse, and serves as a reference for future maintenance.

## Docstrings

### Required for All Public Functions, Classes, and Modules

**Why**: Docstrings are accessible via help(), IDEs, and documentation generators.

### Format: Google Style

**Why**: Readable, widely adopted, supported by documentation tools.

```python
def calculate_discount(price: float, discount_percent: float) -> float:
    """Calculate the discounted price.

    Applies a percentage discount to the original price and returns
    the final amount after discount.

    Args:
        price: The original price before discount.
        discount_percent: The discount percentage (0-100).

    Returns:
        The final price after applying the discount.

    Raises:
        ValueError: If discount_percent is not between 0 and 100.

    Example:
        >>> calculate_discount(100.0, 20.0)
        80.0
    """
    if not 0 <= discount_percent <= 100:
        raise ValueError("Discount must be between 0 and 100")
    return price * (1 - discount_percent / 100)
```

### Class Docstrings

```python
class UserRepository:
    """Repository for user data operations.

    Handles all database operations related to user entities,
    including CRUD operations and queries.

    Attributes:
        db_connection: The database connection instance.
        cache: Optional cache for frequently accessed users.

    Example:
        >>> repo = UserRepository(db_conn)
        >>> user = repo.get_by_id(123)
    """

    def __init__(self, db_connection: Connection) -> None:
        """Initialize the repository with a database connection.

        Args:
            db_connection: Active database connection.
        """
        self.db_connection = db_connection
        self.cache = {}
```

### Module Docstrings

```python
"""User authentication and authorization module.

This module provides functionality for user authentication,
session management, and permission checking.

Typical usage example:

    from myapp.auth import authenticate_user, check_permission

    user = authenticate_user(username, password)
    if check_permission(user, "admin"):
        # Perform admin action
        pass
"""

from typing import Optional
# ... rest of module
```

### One-Line Docstrings

For simple, obvious functions:

```python
def get_username(user_id: int) -> str:
    """Return the username for the given user ID."""
    pass
```

**Why**: Concise when the function is self-explanatory.

## Type Hints

### Always Required

**Why**: Type hints are living documentation that's checked by tools.

```python
from typing import Optional
from collections.abc import Callable

def process_items(
    items: list[str],
    processor: Callable[[str], str],
    max_items: Optional[int] = None
) -> list[str]:
    """Process a list of items using the provided processor function.

    Args:
        items: List of strings to process.
        processor: Function to apply to each item.
        max_items: Maximum number of items to process. Processes all if None.

    Returns:
        List of processed items.
    """
    limited_items = items[:max_items] if max_items else items
    return [processor(item) for item in limited_items]
```

### Complex Type Annotations

```python
from typing import TypeAlias, TypedDict

# Type aliases improve readability
UserId: TypeAlias = int
UserData: TypeAlias = dict[str, str | int | bool]

# TypedDict for structured dictionaries
class Config(TypedDict):
    """Application configuration structure."""
    host: str
    port: int
    debug: bool
    database_url: str
```

## README Files

### Must Follow Standard Readme Spec

**Specification**: [Standard Readme](https://github.com/RichardLitt/standard-readme)

**Why**: Consistent structure makes projects immediately understandable.

### Required Sections

```markdown
# Project Name

> Short description of the project (one sentence)

Longer description explaining what the project does and why it exists.

## Table of Contents

- [Background](#background)
- [Install](#install)
- [Usage](#usage)
- [API](#api) (if applicable)
- [Testing](#testing)
- [Contributing](#contributing)
- [License](#license)

## Background

Explain the context and motivation for the project. Why does this exist?
What problem does it solve?

## Install

```bash
# Clone the repository
git clone https://github.com/user/project.git
cd project

# Set up Python environment
pyenv install 3.11.0
pyenv local 3.11.0

# Install dependencies
poetry install
```

## Usage

```python
from myproject import MyClass

# Example usage
obj = MyClass()
result = obj.do_something()
```

## Testing

```bash
# Run tests
poetry run pytest

# Run with coverage
poetry run pytest --cov=src
```

## Contributing

Pull requests are welcome. Please ensure:
- Tests pass
- Coverage remains above 80%
- Code follows style guidelines

## License

[MIT](LICENSE)
```

### Project-Specific README

Each project should have its own README that includes:
- Installation instructions
- Configuration requirements
- Usage examples
- Development setup
- Testing instructions
- Deployment procedures (if applicable)

## Inline Comments

### Summarize Sections, Not Individual Lines

**Why**: Code should be self-documenting. Comments explain "why", not "what".

```python
# Good: Explains intent and reasoning
def process_user_data(data: dict) -> dict:
    """Process and validate user data."""
    # Remove internal fields that shouldn't be exposed via API
    sanitized = {k: v for k, v in data.items() if not k.startswith("_")}

    # Apply business rule: email must be lowercase for consistency
    if "email" in sanitized:
        sanitized["email"] = sanitized["email"].lower()

    return sanitized


# Bad: Explains obvious code
def process_user_data(data: dict) -> dict:
    """Process and validate user data."""
    # Create a new dictionary
    sanitized = {}
    # Loop through items
    for k, v in data.items():
        # Check if key starts with underscore
        if not k.startswith("_"):
            # Add to sanitized dictionary
            sanitized[k] = v
    # Return the result
    return sanitized
```

### When to Use Comments

**Use comments for**:
- Complex algorithms or business logic
- Non-obvious optimization decisions
- Workarounds for bugs or limitations
- TODOs and FIXMEs (with context)

```python
# TODO(username, 2025-10-11): Refactor this to use async processing
# once the database client is upgraded to support connection pooling.
def process_batch(items: list) -> None:
    for item in items:
        process_item(item)


# FIXME: This is a workaround for issue #123 in the external library.
# Remove once the library is updated to version 2.0+.
def handle_special_case(value: str) -> str:
    if value.endswith("\\n"):
        value = value[:-2]  # Library doesn't strip properly
    return value


# Performance: Using list comprehension is 3x faster than map() for this case
# Profiled on 2025-10-11 with 10k items - see benchmarks/list_processing.py
result = [transform(item) for item in items]
```

## Code Should Be Self-Documenting

### Use Descriptive Names

```python
# Good: Names explain purpose
def calculate_monthly_interest(
    principal: float,
    annual_rate: float,
    months: int
) -> float:
    """Calculate total interest for a loan."""
    monthly_rate = annual_rate / 12
    total_interest = principal * monthly_rate * months
    return total_interest


# Bad: Needs comments to understand
def calc(p: float, r: float, m: int) -> float:
    """Calculate total interest for a loan."""
    # Convert annual rate to monthly
    mr = r / 12
    # Calculate total interest
    ti = p * mr * m
    return ti
```

### Extract Complex Logic to Named Functions

```python
# Good: Function names document purpose
def is_eligible_for_discount(user: User, order: Order) -> bool:
    """Check if user qualifies for a discount on this order."""
    return (
        user.is_premium_member()
        and order.total > 100
        and order.items_count >= 3
    )

def apply_discount_if_eligible(user: User, order: Order) -> Order:
    """Apply discount to order if user is eligible."""
    if is_eligible_for_discount(user, order):
        order.apply_discount(0.10)
    return order


# Bad: Complex condition needs comment
def process_order(user: User, order: Order) -> Order:
    """Process the order."""
    # Check if user gets discount (premium + >$100 + 3+ items)
    if user.is_premium_member() and order.total > 100 and order.items_count >= 3:
        order.apply_discount(0.10)
    return order
```

## API Documentation

### For Libraries and Frameworks

Document all public APIs comprehensively:

```python
class DataProcessor:
    """Process and transform data with configurable options.

    This class provides methods for data transformation, validation,
    and formatting. It supports multiple input formats and can be
    configured with custom processors.

    Attributes:
        config: Configuration dictionary for the processor.
        validators: List of validator functions to apply.

    Example:
        >>> processor = DataProcessor(config={"format": "json"})
        >>> result = processor.process(raw_data)
        >>> print(result)
        {'status': 'success', 'data': [...]}

    Note:
        The processor maintains internal state and is not thread-safe.
        Create separate instances for concurrent processing.
    """

    def process(self, data: dict) -> dict:
        """Process the input data through all configured transformations.

        The processing pipeline applies validators, transformers, and
        formatters in sequence. If any step fails, processing stops and
        an error is returned.

        Args:
            data: Input data dictionary to process. Must contain at least
                a 'type' key indicating the data type.

        Returns:
            Processed data dictionary with 'status' and 'data' keys.
            Status is 'success' or 'error'.

        Raises:
            ValueError: If data is missing required 'type' key.
            ProcessingError: If any processing step fails.

        Example:
            >>> data = {"type": "user", "name": "Alice", "age": 30}
            >>> result = processor.process(data)
            >>> result['status']
            'success'
        """
        pass
```

## SQL Documentation

### Complex Queries Need Comments

```sql
-- Get user engagement metrics for the last 30 days
-- Includes: page views, session duration, and interaction counts
-- Used by: reports/engagement_dashboard.py
-- Performance: ~200ms on 1M users (indexed on user_id, timestamp)
SELECT
    u.user_id
    , u.username
    , COUNT(pv.page_view_id) AS page_views
    , SUM(s.duration_seconds) AS total_session_time
    , COUNT(i.interaction_id) AS interactions
FROM users u
LEFT JOIN page_views pv
    ON u.user_id = pv.user_id
    AND pv.viewed_at >= CURRENT_DATE - INTERVAL '30 days'
LEFT JOIN sessions s
    ON u.user_id = s.user_id
    AND s.started_at >= CURRENT_DATE - INTERVAL '30 days'
LEFT JOIN interactions i
    ON u.user_id = i.user_id
    AND i.created_at >= CURRENT_DATE - INTERVAL '30 days'
WHERE u.is_active = TRUE
GROUP BY u.user_id, u.username
HAVING COUNT(pv.page_view_id) > 0
ORDER BY page_views DESC;
```

### SQL File Headers

```sql
-- File: sql/queries/get_user_engagement.sql
-- Purpose: Calculate user engagement metrics for reporting
-- Parameters:
--   :days_back (int) - Number of days to look back (default: 30)
--   :min_views (int) - Minimum page views threshold (default: 1)
-- Returns: user_id, username, page_views, total_session_time, interactions
-- Performance: O(n) with indexes on user_id and timestamp columns

SELECT
    -- ... query
```

## Documentation Tools

### Generate API Docs with Sphinx or mkdocs

```toml
# pyproject.toml
[tool.poetry.group.docs]
optional = true

[tool.poetry.group.docs.dependencies]
sphinx = "^7.0.0"
sphinx-rtd-theme = "^2.0.0"
```

**Why**: Automatically generate documentation from docstrings.

### Keep Documentation Updated

- Update docstrings when code changes
- Review README during major changes
- Run documentation build in CI/CD
- Link to documentation in PRs

## Documentation Checklist

For every function/class:
- [ ] Docstring present
- [ ] Type hints for all parameters and return
- [ ] Args and Returns documented
- [ ] Raises documented (if applicable)
- [ ] Example provided (for complex cases)

For every project:
- [ ] README follows Standard Readme spec
- [ ] Installation instructions are current
- [ ] Usage examples are tested
- [ ] Contributing guidelines present
- [ ] License specified

## Related Standards

- See `python_style.md` for docstring formatting
- See `testing_standards.md` for test documentation
- See `git_workflow.md` for commit message documentation

---

**Last Updated**: 2025-10-31
**Status**: Strong preference - deviations require justification

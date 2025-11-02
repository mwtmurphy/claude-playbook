# Python Style Standards

**Status**: Strong preference - deviations require justification and approval
**Scope**: Python code formatting and naming conventions

## Overview

Consistent code style improves readability, reduces cognitive load, and makes collaboration easier. These standards are based on industry best practices and community conventions.

**Why**: PEP 8 is the de facto standard for Python code, making code familiar to all Python developers.

## PEP 8 Compliance

Follow [PEP 8](https://peps.python.org/pep-0008/) as the foundation for all Python code.

### Line Length

```python
# Maximum 88 characters (Black formatter default)
# Why: Balances readability with screen real estate

def calculate_user_engagement_score(
    page_views: int, time_on_site: float, interactions: int
) -> float:
    """Calculate engagement score based on multiple factors."""
    return (page_views * 0.3) + (time_on_site * 0.5) + (interactions * 0.2)
```

### Indentation

```python
# 4 spaces per indentation level (never tabs)
# Why: Consistent with Python community standard

def process_data(items: list[dict]) -> list[dict]:
    results = []
    for item in items:
        if item.get("active"):
            results.append(item)
    return results
```

### Blank Lines

```python
# 2 blank lines between top-level definitions
# 1 blank line between method definitions
# Why: Improves visual separation and code scanning

class DataProcessor:
    """Process and transform data."""

    def __init__(self, config: dict) -> None:
        self.config = config

    def process(self, data: list) -> list:
        """Process the data."""
        return [self._transform(item) for item in data]


class DataValidator:
    """Validate data against schema."""
    pass
```

## Naming Conventions

### Functions and Variables: snake_case

**Why**: PEP 8 standard, improves readability for multi-word names

```python
def calculate_total_price(item_price: float, tax_rate: float) -> float:
    """Calculate total price including tax."""
    subtotal = item_price
    tax_amount = subtotal * tax_rate
    total_price = subtotal + tax_amount
    return total_price

user_count = 42
is_active = True
database_connection = create_connection()
```

### Classes: PascalCase

**Why**: Distinguishes classes from functions and variables

```python
class UserAccount:
    """Represents a user account."""
    pass

class DatabaseConnection:
    """Manages database connections."""
    pass

class OrderProcessor:
    """Processes customer orders."""
    pass
```

### Constants: UPPER_SNAKE_CASE

**Why**: Immediately identifies immutable configuration values

```python
MAX_RETRY_ATTEMPTS = 3
DEFAULT_TIMEOUT = 30
API_BASE_URL = "https://api.example.com"
DATABASE_CONNECTION_POOL_SIZE = 10
```

### Private/Internal: Leading underscore

**Why**: Signals intent that these are internal implementation details

```python
class DataService:
    def __init__(self) -> None:
        self._cache = {}  # Internal cache
        self._connection = None  # Private connection

    def _validate_input(self, data: dict) -> bool:
        """Internal validation method."""
        return bool(data)

    def process(self, data: dict) -> dict:
        """Public method."""
        if self._validate_input(data):
            return {"status": "success"}
        return {"status": "error"}
```

## Type Hints

### Always Required

**Why**: Type hints improve code clarity, enable better IDE support, catch bugs early, and serve as inline documentation.

```python
from typing import Optional, Union
from collections.abc import Callable

def greet_user(name: str, age: int) -> str:
    """Greet a user with their name and age."""
    return f"Hello {name}, you are {age} years old"

def fetch_user(user_id: int) -> Optional[dict]:
    """Fetch user by ID, return None if not found."""
    # Implementation
    return None

def process_items(
    items: list[str],
    processor: Callable[[str], str],
    max_items: int = 100
) -> list[str]:
    """Process items using provided processor function."""
    return [processor(item) for item in items[:max_items]]

# Modern Python 3.10+ union syntax
def parse_input(value: str | int) -> int:
    """Parse input as integer."""
    return int(value)
```

### Complex Types

```python
from typing import TypeAlias, TypedDict

# Type aliases for clarity
UserId: TypeAlias = int
UserName: TypeAlias = str

# TypedDict for structured dictionaries
class UserData(TypedDict):
    id: UserId
    name: UserName
    email: str
    is_active: bool

def create_user(data: UserData) -> UserId:
    """Create a new user."""
    # Implementation
    return data["id"]
```

## Import Organisation

### Order and Grouping

**Why**: Consistent import order makes dependencies clear and easy to audit

```python
# 1. Standard library imports
import os
import sys
from datetime import datetime
from pathlib import Path

# 2. Third-party imports
import pytest
import sqlalchemy
from fastapi import FastAPI

# 3. Local application imports
from myapp.core import config
from myapp.models import User
from myapp.utils import helpers

# Blank line before code starts
```

### Import Style

```python
# Prefer explicit imports over wildcard
# Good
from myapp.utils import validate_email, sanitize_input

# Avoid
from myapp.utils import *

# For many imports from same module, use parentheses
from myapp.models import (
    User,
    Order,
    Product,
    Category,
    Review,
)
```

## String Formatting

### Prefer f-strings for interpolation

**Why**: Most readable and performant for string interpolation

```python
# Good
name = "Alice"
age = 30
message = f"User {name} is {age} years old"

# Avoid (old style)
message = "User %s is %d years old" % (name, age)
message = "User {} is {} years old".format(name, age)

# Multi-line f-strings
query = (
    f"SELECT * FROM users "
    f"WHERE name = '{name}' "
    f"AND age > {age}"
)
```

## Quotes

### Single or double quotes consistently

**Why**: Consistency reduces cognitive load

```python
# Preferred: double quotes for consistency with JSON
message = "Hello, world"
name = "Alice"

# Use opposite quotes to avoid escaping
greeting = "She said 'hello' to me"
sql = 'SELECT * FROM users WHERE name = "Alice"'

# Triple double quotes for docstrings
def example() -> None:
    """
    This is a docstring.
    It uses triple double quotes.
    """
    pass
```

## Formatting Tools

### Black

**Why**: Uncompromising formatter eliminates style debates, creates consistency.

**Configuration** (pyproject.toml)
```toml
[tool.black]
line-length = 88
target-version = ['py311']
include = '\.pyi?$'
```

### isort

**Why**: Automatically organizes imports per our standards.

**Configuration** (pyproject.toml)
```toml
[tool.isort]
profile = "black"
line_length = 88
```

### Ruff

**Why**: Fast linter and formatter for Python code.

**Configuration** (pyproject.toml)
```toml
[tool.ruff]
line-length = 88
target-version = "py311"
```

## Editor Configuration

**EditorConfig** (.editorconfig)
```ini
root = true

[*.py]
charset = utf-8
end_of_line = lf
insert_final_newline = true
trim_trailing_whitespace = true
indent_style = space
indent_size = 4
```

## Enforcement

1. **Pre-commit hooks**: Run formatters automatically (see `git_workflow.md`)
2. **CI/CD checks**: Fail builds on style violations
3. **Editor integration**: Configure IDE to format on save
4. **Code review**: Style compliance is a review checkpoint

## Related Standards

- See `documentation_standards.md` for docstring style
- See `git_workflow.md` for commit message style
- See `sql_style.md` for SQL formatting standards

---

**Last Updated**: 2025-10-11
**Status**: Strong preference - deviations require justification

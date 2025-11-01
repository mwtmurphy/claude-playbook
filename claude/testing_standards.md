# Testing Standards

**Status**: Strong preference - deviations require justification and approval
**Scope**: Testing requirements and best practices for Python and SQL
**Last Updated**: 2025-10-31

---

## Overview

Tests are documentation, safety nets, and design tools. Good tests catch bugs early, enable confident refactoring, and serve as executable specifications.

**Why**: Well-tested code is more maintainable, reliable, and easier to change.

## Testing Framework

### Primary: pytest

**Why pytest**:
- Simple, Pythonic syntax
- Powerful fixtures and parameterization
- Excellent plugin ecosystem
- Better error messages than unittest

```python
# Good: pytest style
def test_user_creation():
    """Test that users are created with valid data."""
    user = User(id=1, username="alice", email="alice@example.com")
    assert user.username == "alice"
    assert user.email == "alice@example.com"


# Avoid: unittest style (unless required by framework)
import unittest

class TestUser(unittest.TestCase):
    def test_user_creation(self):
        user = User(id=1, username="alice", email="alice@example.com")
        self.assertEqual(user.username, "alice")
```

## Test Organization

### Arrange-Act-Assert (AAA) Pattern

Structure all tests consistently.

**Why**: Makes tests easier to read and understand at a glance.

```python
def test_order_total_calculation():
    """Test that order total is calculated correctly with tax."""
    # Arrange - Set up test data
    item_price = 100.0
    tax_rate = 0.08
    calculator = OrderCalculator()

    # Act - Perform the action being tested
    total = calculator.calculate_total(item_price, tax_rate)

    # Assert - Verify the result
    assert total == 108.0
```

## Coverage Standards

### Target: 80%+ Overall Coverage

**Why**: Balances thoroughness with diminishing returns. 100% coverage doesn't guarantee bug-free code, but 80%+ catches most issues.

```bash
# Run tests with coverage
pytest --cov=src --cov-report=term-missing --cov-report=html

# Fail if coverage below threshold
pytest --cov=src --cov-fail-under=80
```

**Coverage priorities**:
1. Critical business logic: 90-100%
2. Data access layer: 80-90%
3. Utility functions: 80-90%
4. Framework glue code: 50-70%

## Test Types

### Unit Tests (Required)

Test individual functions/classes in isolation.

**Why**: Fast, focused, pinpoint failures.

```python
def test_calculate_discount_percentage():
    """Test discount calculation with percentage."""
    # Arrange
    original_price = 100.0
    discount_percent = 20.0

    # Act
    result = calculate_discount(original_price, discount_percent)

    # Assert
    assert result == 80.0
```

### Integration Tests (Encouraged)

Test components working together.

**Why**: Catches interface issues and integration bugs.

```python
def test_user_service_creates_and_retrieves_user(db_connection):
    """Test that UserService can create and retrieve users."""
    # Arrange
    repo = UserRepository(db_connection)
    service = UserService(repo)
    user_data = {"username": "testuser", "email": "test@example.com"}

    # Act
    user_id = service.create_user(user_data)
    retrieved_user = service.get_user(user_id)

    # Assert
    assert retrieved_user is not None
    assert retrieved_user.username == "testuser"
```

## Test File Naming

### Convention: test_*.py

```
tests/
├── unit/
│   ├── test_calculator.py
│   ├── test_user_service.py
│   └── test_validators.py
├── integration/
│   ├── test_database.py
│   └── test_api_endpoints.py
└── conftest.py
```

**Why**: Clear naming makes test discovery automatic and purpose obvious.

## SQL Query Testing

### Test Queries with Test Database

```python
def test_get_active_users_query(test_db):
    """Test that get_active_users query returns correct results."""
    # Arrange - Insert test data
    test_db.execute("""
        INSERT INTO users (username, email, is_active)
        VALUES
            ('alice', 'alice@example.com', TRUE),
            ('bob', 'bob@example.com', FALSE),
            ('charlie', 'charlie@example.com', TRUE)
    """)

    # Act - Run query under test
    query_path = Path("sql/queries/get_active_users.sql")
    query = query_path.read_text()
    results = test_db.execute(query).fetchall()

    # Assert
    assert len(results) == 2
    usernames = [r["username"] for r in results]
    assert "alice" in usernames
    assert "charlie" in usernames
```

## Test Configuration

### pyproject.toml

```toml
[tool.pytest.ini_options]
testpaths = ["tests"]
python_files = ["test_*.py"]
addopts = [
    "-v",
    "--strict-markers",
    "--cov=src",
    "--cov-report=term-missing",
    "--cov-fail-under=80",
]
```

## Related Standards

- See `python_style.md` for test code formatting
- See `documentation_standards.md` for test documentation
- See `sql_style.md` for SQL query testing
- See `streamlit_standards.md` for Streamlit app testing
- See `interactive_visualization_testing.md` for visualization testing

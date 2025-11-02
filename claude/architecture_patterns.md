# Architecture Patterns

**Status**: Strong preference - deviations require justification and approval
**Scope**: Framework-agnostic Python architectural principles
**Last Updated**: 2025-10-31

---

## Overview

Good architecture makes code maintainable, testable, and scalable. These patterns are framework-agnostic and applicable to any Python project, whether using Django, FastAPI, Flask, or no framework at all.

**Why**: Consistent architecture reduces cognitive load, improves collaboration, and makes codebases easier to navigate and modify.

## Core Principles

### Separation of Concerns

Keep distinct responsibilities in separate modules/classes.

**Why**: Each component has a single, clear purpose, making code easier to understand, test, and modify.

```python
# Good: Clear separation
# models.py - Data structures
from dataclasses import dataclass

@dataclass
class User:
    id: int
    username: str
    email: str


# repository.py - Data access
class UserRepository:
    def __init__(self, db_connection):
        self.db = db_connection

    def get_by_id(self, user_id: int) -> User | None:
        """Fetch user from database."""
        # Database logic only
        pass


# service.py - Business logic
class UserService:
    def __init__(self, user_repo: UserRepository):
        self.user_repo = user_repo

    def activate_user(self, user_id: int) -> bool:
        """Activate a user account."""
        # Business logic only
        user = self.user_repo.get_by_id(user_id)
        if user:
            # Activation logic
            return True
        return False


# api.py - Presentation/interface
def activate_user_endpoint(user_id: int) -> dict:
    """HTTP endpoint to activate user."""
    service = UserService(user_repo)
    success = service.activate_user(user_id)
    return {"success": success}
```

### Dependency Injection

Pass dependencies rather than creating them internally.

**Why**: Makes code testable, flexible, and loosely coupled.

```python
# Good: Dependencies injected
class OrderProcessor:
    def __init__(
        self,
        payment_gateway: PaymentGateway,
        notification_service: NotificationService,
        order_repo: OrderRepository
    ):
        self.payment_gateway = payment_gateway
        self.notification_service = notification_service
        self.order_repo = order_repo

    def process_order(self, order_id: int) -> bool:
        """Process an order through payment and notification."""
        order = self.order_repo.get(order_id)
        if self.payment_gateway.charge(order.amount):
            self.notification_service.send_confirmation(order)
            return True
        return False


# Bad: Creates dependencies internally
class OrderProcessor:
    def __init__(self):
        self.payment_gateway = StripeGateway()  # Hardcoded!
        self.notification_service = EmailService()  # Can't test!

    def process_order(self, order_id: int) -> bool:
        # ... same logic
        pass
```

### Single Responsibility Principle

Each module/class should have one reason to change.

**Why**: Reduces coupling, improves testability, makes changes safer.

```python
# Good: Single responsibility
class UserValidator:
    """Validates user data only."""

    def validate_email(self, email: str) -> bool:
        """Validate email format."""
        return "@" in email and "." in email

    def validate_password(self, password: str) -> bool:
        """Validate password strength."""
        return len(password) >= 8


class UserRepository:
    """Handles user data persistence only."""

    def save(self, user: User) -> int:
        """Save user to database."""
        pass

    def get(self, user_id: int) -> User | None:
        """Retrieve user from database."""
        pass


# Bad: Multiple responsibilities
class UserManager:
    """Does everything - hard to test and maintain!"""

    def validate_and_save_user(self, user_data: dict) -> int:
        # Validation logic
        # Database logic
        # Business logic
        # Email sending
        pass
```

### Interface Segregation

Create focused interfaces rather than large, monolithic ones.

**Why**: Clients shouldn't depend on methods they don't use.

```python
from abc import ABC, abstractmethod

# Good: Focused interfaces
class Readable(ABC):
    @abstractmethod
    def read(self, key: str) -> str | None:
        """Read data."""
        pass


class Writable(ABC):
    @abstractmethod
    def write(self, key: str, value: str) -> None:
        """Write data."""
        pass


class Cache(Readable, Writable):
    """Full cache implements both."""

    def read(self, key: str) -> str | None:
        pass

    def write(self, key: str, value: str) -> None:
        pass


class ReadOnlyCache(Readable):
    """Read-only cache only implements Readable."""

    def read(self, key: str) -> str | None:
        pass
```

## Directory Structure

### Small Projects (< 1000 lines)

```
project_name/
├── pyproject.toml          # Poetry configuration
├── .python-version         # pyenv version
├── README.md               # Project documentation
├── claude/                 # Project-specific standards (optional)
├── src/
│   └── project_name/
│       ├── __init__.py
│       ├── main.py         # Entry point
│       ├── models.py       # Data models
│       ├── services.py     # Business logic
│       └── utils.py        # Utilities
├── tests/
│   ├── __init__.py
│   ├── test_models.py
│   └── test_services.py
└── sql/                    # SQL files (if applicable)
    ├── queries/
    └── schemas/
```

### Medium Projects (1000-10000 lines)

```
project_name/
├── pyproject.toml
├── .python-version
├── README.md
├── claude/
├── src/
│   └── project_name/
│       ├── __init__.py
│       ├── main.py
│       ├── core/           # Core business logic
│       │   ├── __init__.py
│       │   ├── models.py
│       │   └── services.py
│       ├── api/            # API layer (if applicable)
│       │   ├── __init__.py
│       │   └── routes.py
│       ├── data/           # Data access layer
│       │   ├── __init__.py
│       │   ├── repositories.py
│       │   └── database.py
│       └── utils/          # Shared utilities
│           ├── __init__.py
│           └── helpers.py
├── tests/
│   ├── unit/
│   ├── integration/
│   └── conftest.py         # Pytest fixtures
└── sql/
    ├── queries/
    ├── schemas/
    └── migrations/
```

### Large Projects (> 10000 lines)

```
project_name/
├── pyproject.toml
├── .python-version
├── README.md
├── claude/
├── src/
│   └── project_name/
│       ├── __init__.py
│       ├── main.py
│       ├── domain/         # Domain models (business entities)
│       │   ├── __init__.py
│       │   ├── user/
│       │   ├── order/
│       │   └── product/
│       ├── application/    # Use cases / services
│       │   ├── __init__.py
│       │   ├── user_service.py
│       │   └── order_service.py
│       ├── infrastructure/ # External concerns
│       │   ├── __init__.py
│       │   ├── database/
│       │   ├── cache/
│       │   └── messaging/
│       ├── interfaces/     # API/CLI/UI
│       │   ├── __init__.py
│       │   ├── api/
│       │   └── cli/
│       └── shared/         # Shared utilities
│           ├── __init__.py
│           └── utils/
├── tests/
│   ├── unit/
│   ├── integration/
│   ├── e2e/
│   └── conftest.py
└── sql/
    └── (organized by domain)
```

**Why these structures**: Start simple, add complexity only as needed. Clear boundaries make navigation intuitive.

## Layered Architecture

### Typical Layers (Outside to Inside)

```
┌─────────────────────────────────┐
│  Presentation Layer (API/CLI)   │  ← User interaction
├─────────────────────────────────┤
│  Application Layer (Services)   │  ← Business logic orchestration
├─────────────────────────────────┤
│  Domain Layer (Models/Entities) │  ← Core business rules
├─────────────────────────────────┤
│  Data Layer (Repositories)      │  ← Data persistence
└─────────────────────────────────┘
```

**Dependency Rule**: Inner layers don't depend on outer layers.

**Why**: Changes in outer layers (UI, database) don't affect core business logic.

### Example Implementation

```python
# Domain Layer - No external dependencies
from dataclasses import dataclass
from decimal import Decimal

@dataclass
class Order:
    """Core business entity."""
    id: int
    customer_id: int
    total: Decimal
    status: str

    def can_be_cancelled(self) -> bool:
        """Business rule: only pending orders can be cancelled."""
        return self.status == "pending"


# Data Layer - Depends on Domain
from abc import ABC, abstractmethod

class OrderRepository(ABC):
    """Abstract repository - domain doesn't know about database."""

    @abstractmethod
    def get(self, order_id: int) -> Order | None:
        """Fetch order."""
        pass

    @abstractmethod
    def save(self, order: Order) -> None:
        """Persist order."""
        pass


# Application Layer - Orchestrates domain and data
class OrderService:
    """Business logic orchestration."""

    def __init__(self, order_repo: OrderRepository):
        self.order_repo = order_repo

    def cancel_order(self, order_id: int) -> bool:
        """Cancel an order if possible."""
        order = self.order_repo.get(order_id)
        if not order:
            return False

        if not order.can_be_cancelled():
            raise ValueError("Order cannot be cancelled")

        order.status = "cancelled"
        self.order_repo.save(order)
        return True


# Presentation Layer - HTTP, CLI, etc.
def cancel_order_endpoint(order_id: int) -> dict:
    """API endpoint."""
    service = OrderService(order_repo)
    try:
        success = service.cancel_order(order_id)
        return {"success": success}
    except ValueError as e:
        return {"error": str(e)}
```

## SQL File Organisation

### Keep SQL Separate from Python

**Why**: Easier to read, test, and optimize queries independently.

```python
# Good: SQL in separate files
from pathlib import Path

class UserRepository:
    def __init__(self, db_connection):
        self.db = db_connection
        self.queries_path = Path(__file__).parent / "sql" / "queries"

    def get_active_users(self) -> list[dict]:
        """Get all active users."""
        query = (self.queries_path / "get_active_users.sql").read_text()
        return self.db.execute(query).fetchall()


# sql/queries/get_active_users.sql
SELECT
    user_id,
    username,
    email,
    created_at
FROM users
WHERE is_active = TRUE
ORDER BY created_at DESC;
```

### SQL Directory Structure

```
sql/
├── queries/            # SELECT queries
│   ├── users/
│   │   ├── get_active_users.sql
│   │   └── get_user_by_email.sql
│   └── orders/
│       └── get_recent_orders.sql
├── schemas/            # Table definitions
│   ├── users.sql
│   └── orders.sql
└── migrations/         # Database migrations
    ├── 001_create_users_table.sql
    └── 002_add_email_index.sql
```

**Why**: Clear organisation makes queries easy to find, review, and version control.

## Dependency Management

### Use Abstract Base Classes for Contracts

```python
from abc import ABC, abstractmethod

# Define contract
class EmailService(ABC):
    @abstractmethod
    def send(self, to: str, subject: str, body: str) -> bool:
        """Send an email."""
        pass


# Implementations
class SendGridEmailService(EmailService):
    def send(self, to: str, subject: str, body: str) -> bool:
        # SendGrid implementation
        pass


class SMTPEmailService(EmailService):
    def send(self, to: str, subject: str, body: str) -> bool:
        # SMTP implementation
        pass


# Business logic depends on abstraction, not implementation
class UserNotificationService:
    def __init__(self, email_service: EmailService):
        self.email_service = email_service  # Any implementation works!

    def send_welcome_email(self, user_email: str) -> bool:
        return self.email_service.send(
            to=user_email,
            subject="Welcome!",
            body="Thank you for joining."
        )
```

**Why**: Code depends on abstractions, not concrete implementations. Easy to swap, test, and extend.

## Modularity

### Keep Modules Focused and Cohesive

```python
# Good: Each module has clear purpose
# auth/
#   ├── __init__.py
#   ├── password.py      # Password hashing/verification
#   ├── tokens.py        # Token generation/validation
#   └── session.py       # Session management

# Bad: Single monolithic module
# auth.py  # 2000 lines of everything
```

**Why**: Small, focused modules are easier to understand, test, and reuse.

### Avoid Circular Dependencies

```python
# Bad: Circular dependency
# user.py
from order import Order  # Imports Order

class User:
    def get_orders(self) -> list[Order]:
        pass


# order.py
from user import User  # Imports User

class Order:
    def get_customer(self) -> User:
        pass


# Good: Break the cycle with abstraction or restructuring
# models.py - Define both in same module if tightly coupled
class User:
    pass

class Order:
    customer: User
```

## Configuration Management

### Use Configuration Classes

```python
from dataclasses import dataclass
from pathlib import Path

@dataclass
class DatabaseConfig:
    """Database connection configuration."""
    host: str
    port: int
    database: str
    username: str
    password: str
    pool_size: int = 10


@dataclass
class AppConfig:
    """Application configuration."""
    database: DatabaseConfig
    debug: bool = False
    log_level: str = "INFO"
    data_dir: Path = Path("./data")


# Load from environment, config file, etc.
def load_config() -> AppConfig:
    """Load configuration from environment."""
    # Implementation
    pass
```

**Why**: Type-safe, validated, and self-documenting configuration.

## Related Standards

- See `python_style.md` for code formatting and naming conventions
- See `python_error_handling.md` for error handling across architectural layers
- See `python_testing_standards.md` for testing layered architecture and dependency injection
- See `sql_database_standards.md` for SQL-specific architecture and repository patterns
- See `performance_considerations.md` for architectural performance implications

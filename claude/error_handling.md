# Error Handling Standards

**Status**: Strong preference - deviations require justification and approval
**Scope**: Exception handling and error management for Python and SQL

## Overview

Good error handling makes applications robust, debuggable, and user-friendly. Errors should be caught at the right level, logged appropriately, and provide actionable information.

**Why**: Clear error handling prevents silent failures, aids debugging, and improves user experience.

## Python Exception Handling

### Use Specific Exception Types

**Why**: Specific exceptions enable targeted error handling and better debugging.

```python
# Good: Specific exception types
def get_user(user_id: int) -> User:
    """Get user by ID."""
    try:
        result = database.query(
            "SELECT * FROM users WHERE id = :user_id",
            params={"user_id": user_id}
        )
        if not result:
            raise UserNotFoundError(f"User {user_id} not found")
        return User(**result)
    except DatabaseConnectionError as e:
        logger.error(f"Database connection failed: {e}")
        raise
    except DatabaseQueryError as e:
        logger.error(f"Query failed for user {user_id}: {e}")
        raise


# Bad: Generic exceptions
def get_user(user_id: int) -> User:
    """Get user by ID."""
    try:
        result = database.query(
            "SELECT * FROM users WHERE id = :user_id",
            params={"user_id": user_id}
        )
        return User(**result)
    except Exception as e:  # Too broad!
        print(f"Error: {e}")
        return None
```

### Custom Exception Hierarchy

Create domain-specific exceptions for better error handling.

**Why**: Custom exceptions make error handling more semantic and maintainable.

```python
# base_exceptions.py
class ApplicationError(Exception):
    """Base exception for all application errors."""
    pass


class ValidationError(ApplicationError):
    """Raised when input validation fails."""
    pass


class NotFoundError(ApplicationError):
    """Raised when a requested resource is not found."""
    pass


class AuthenticationError(ApplicationError):
    """Raised when authentication fails."""
    pass


class AuthorizationError(ApplicationError):
    """Raised when user lacks required permissions."""
    pass


class DatabaseError(ApplicationError):
    """Base exception for database-related errors."""
    pass


class DatabaseConnectionError(DatabaseError):
    """Raised when database connection fails."""
    pass


class DatabaseQueryError(DatabaseError):
    """Raised when a database query fails."""
    pass


# Usage
class UserNotFoundError(NotFoundError):
    """Raised when a user is not found."""

    def __init__(self, user_id: int):
        self.user_id = user_id
        super().__init__(f"User {user_id} not found")


class InvalidEmailError(ValidationError):
    """Raised when email format is invalid."""

    def __init__(self, email: str):
        self.email = email
        super().__init__(f"Invalid email format: {email}")
```

### Exception Context

Provide useful context in exceptions.

**Why**: Context helps debugging and provides actionable error information.

```python
# Good: Rich context
class OrderProcessingError(ApplicationError):
    """Raised when order processing fails."""

    def __init__(self, order_id: int, reason: str, **context):
        self.order_id = order_id
        self.reason = reason
        self.context = context
        message = f"Failed to process order {order_id}: {reason}"
        if context:
            message += f" | Context: {context}"
        super().__init__(message)


# Usage
try:
    process_payment(order.id, payment_info)
except PaymentGatewayError as e:
    raise OrderProcessingError(
        order_id=order.id,
        reason="Payment gateway failure",
        payment_method=payment_info.method,
        gateway_error=str(e),
        timestamp=datetime.now()
    ) from e


# Bad: Minimal context
class OrderError(Exception):
    """Order error."""
    pass

raise OrderError("Failed")  # No useful information!
```

### Exception Chaining

Use `raise ... from` to preserve exception context.

**Why**: Maintains full error chain for debugging.

```python
# Good: Preserve exception chain
def fetch_user_data(user_id: int) -> dict:
    """Fetch user data from API."""
    try:
        response = requests.get(f"{API_URL}/users/{user_id}")
        response.raise_for_status()
        return response.json()
    except requests.HTTPError as e:
        raise UserServiceError(
            f"Failed to fetch user {user_id}"
        ) from e  # Preserves original exception
    except requests.ConnectionError as e:
        raise ServiceUnavailableError(
            "User service is unavailable"
        ) from e


# Bad: Loses original exception
def fetch_user_data(user_id: int) -> dict:
    """Fetch user data from API."""
    try:
        response = requests.get(f"{API_URL}/users/{user_id}")
        return response.json()
    except Exception:
        raise UserServiceError("Failed")  # Original error lost!
```

### Don't Catch Unless You Can Handle

**Why**: Catching exceptions you can't handle hides problems.

```python
# Good: Only catch what you can handle
def save_user(user: User) -> int:
    """Save user to database."""
    try:
        return database.save(user)
    except DatabaseConnectionError:
        # We can handle this by retrying
        logger.warning("Database connection lost, retrying...")
        reconnect_database()
        return database.save(user)
    # Let other exceptions propagate


# Bad: Catch everything and do nothing useful
def save_user(user: User) -> int:
    """Save user to database."""
    try:
        return database.save(user)
    except Exception as e:
        print(f"Error: {e}")
        return -1  # Silent failure!
```

### Validate Early, Fail Fast

**Why**: Catch errors as early as possible, before they cause side effects.

```python
# Good: Validate at boundary
def create_order(
    customer_id: int,
    items: list[OrderItem],
    payment_method: str
) -> Order:
    """Create a new order."""
    # Validate inputs immediately
    if not items:
        raise ValidationError("Order must contain at least one item")

    if customer_id <= 0:
        raise ValidationError("Invalid customer ID")

    if payment_method not in ["credit_card", "paypal", "bank_transfer"]:
        raise ValidationError(f"Invalid payment method: {payment_method}")

    # Proceed with business logic
    order = Order(customer_id=customer_id, items=items)
    process_payment(order, payment_method)
    return order


# Bad: Validate late or not at all
def create_order(customer_id: int, items: list[OrderItem], payment_method: str) -> Order:
    """Create a new order."""
    order = Order(customer_id=customer_id, items=items)
    # Start processing before validation
    process_payment(order, payment_method)  # Might fail after side effects!
    return order
```

## Logging Standards

### Use Structured Logging

**Why**: Structured logs are searchable, filterable, and machine-readable.

```python
import structlog

# Configure structured logging
structlog.configure(
    processors=[
        structlog.processors.add_log_level,
        structlog.processors.TimeStamper(fmt="iso"),
        structlog.processors.JSONRenderer()
    ]
)

logger = structlog.get_logger()

# Good: Structured logging
def process_order(order_id: int) -> None:
    """Process an order."""
    logger.info(
        "processing_order_started",
        order_id=order_id,
        timestamp=datetime.now().isoformat()
    )

    try:
        order = get_order(order_id)
        result = process_payment(order)
        logger.info(
            "order_processed_successfully",
            order_id=order_id,
            amount=order.total,
            payment_method=result.method
        )
    except PaymentError as e:
        logger.error(
            "order_processing_failed",
            order_id=order_id,
            error_type=type(e).__name__,
            error_message=str(e),
            exc_info=True  # Include stack trace
        )
        raise


# Bad: Unstructured string logging
import logging

logging.info(f"Processing order {order_id}")  # Hard to parse
logging.error(f"Error processing order {order_id}: {e}")  # No structure
```

### Log Levels

Use appropriate log levels.

**Why**: Correct levels enable filtering and alerting on important events.

```python
import logging

# ERROR: Something failed, needs attention
logger.error(
    "payment_processing_failed",
    order_id=order_id,
    error=str(e),
    exc_info=True
)

# WARNING: Something unexpected but handled
logger.warning(
    "database_connection_slow",
    response_time_ms=response_time,
    threshold_ms=1000
)

# INFO: Significant business events
logger.info(
    "user_registered",
    user_id=user.id,
    email=user.email
)

# DEBUG: Detailed diagnostic information (development)
logger.debug(
    "cache_lookup",
    cache_key=key,
    cache_hit=hit,
    cache_size=len(cache)
)
```

**Levels guide**:
- `ERROR`: Failures requiring attention
- `WARNING`: Unexpected conditions, degraded operation
- `INFO`: Significant business events
- `DEBUG`: Detailed diagnostic information

### Never Log Sensitive Information

**Why**: Prevents security breaches and compliance violations.

```python
# Good: Sanitize sensitive data
logger.info(
    "user_login_attempt",
    user_id=user.id,
    email=user.email,
    ip_address=request.ip
)

logger.debug(
    "payment_processed",
    order_id=order.id,
    amount=order.total,
    card_last_four=payment.card_last_four  # Only last 4 digits
)


# Bad: Logs sensitive information
logger.info(
    "user_login",
    password=password  # NEVER LOG PASSWORDS!
)

logger.debug(
    "payment_info",
    credit_card=payment.card_number,  # NEVER LOG FULL CARD NUMBERS!
    cvv=payment.cvv  # NEVER LOG CVV!
)
```

## Error Responses

### Consistent Error Response Format

```python
from dataclasses import dataclass
from typing import Optional

@dataclass
class ErrorResponse:
    """Standard error response format."""
    error_code: str
    message: str
    details: Optional[dict] = None
    request_id: Optional[str] = None


# Usage in API
def handle_error(error: Exception, request_id: str) -> ErrorResponse:
    """Convert exceptions to error responses."""
    if isinstance(error, ValidationError):
        return ErrorResponse(
            error_code="VALIDATION_ERROR",
            message=str(error),
            details={"field": error.field, "value": error.value},
            request_id=request_id
        )

    if isinstance(error, NotFoundError):
        return ErrorResponse(
            error_code="NOT_FOUND",
            message=str(error),
            request_id=request_id
        )

    # Generic error (don't expose internals)
    logger.error("unexpected_error", error=str(error), exc_info=True)
    return ErrorResponse(
        error_code="INTERNAL_ERROR",
        message="An unexpected error occurred",
        request_id=request_id
    )
```

## SQL Error Handling

### Connection Error Handling

```python
import psycopg2
from psycopg2 import OperationalError, InterfaceError

class DatabaseConnection:
    """Manages database connections with error handling."""

    def __init__(self, connection_string: str, max_retries: int = 3):
        self.connection_string = connection_string
        self.max_retries = max_retries
        self.connection = None

    def connect(self) -> None:
        """Connect to database with retry logic."""
        for attempt in range(self.max_retries):
            try:
                self.connection = psycopg2.connect(self.connection_string)
                logger.info("database_connected", attempt=attempt + 1)
                return
            except OperationalError as e:
                logger.warning(
                    "database_connection_failed",
                    attempt=attempt + 1,
                    max_retries=self.max_retries,
                    error=str(e)
                )
                if attempt == self.max_retries - 1:
                    raise DatabaseConnectionError(
                        f"Failed to connect after {self.max_retries} attempts"
                    ) from e
                time.sleep(2 ** attempt)  # Exponential backoff
```

### Query Error Handling

```python
def execute_query(connection, query: str, params: dict) -> list[dict]:
    """Execute query with error handling."""
    try:
        cursor = connection.cursor()
        cursor.execute(query, params)
        results = cursor.fetchall()
        cursor.close()
        return results

    except psycopg2.IntegrityError as e:
        # Constraint violation (duplicate key, foreign key, etc.)
        logger.error("database_integrity_error", query=query, error=str(e))
        raise DatabaseIntegrityError(f"Database constraint violated: {e}") from e

    except psycopg2.DataError as e:
        # Invalid data (wrong type, out of range, etc.)
        logger.error("database_data_error", query=query, params=params, error=str(e))
        raise DatabaseDataError(f"Invalid data in query: {e}") from e

    except psycopg2.ProgrammingError as e:
        # SQL syntax error
        logger.error("database_programming_error", query=query, error=str(e))
        raise DatabaseQueryError(f"SQL programming error: {e}") from e

    except psycopg2.OperationalError as e:
        # Connection lost during query
        logger.error("database_operational_error", error=str(e))
        raise DatabaseConnectionError(f"Database connection lost: {e}") from e
```

### Transaction Error Handling

```python
def process_order_transaction(order: Order) -> None:
    """Process order in a database transaction."""
    connection = get_database_connection()

    try:
        # Start transaction
        connection.autocommit = False

        # Execute multiple operations
        insert_order(connection, order)
        update_inventory(connection, order.items)
        create_invoice(connection, order)

        # Commit if all successful
        connection.commit()
        logger.info("order_transaction_committed", order_id=order.id)

    except DatabaseError as e:
        # Rollback on any database error
        connection.rollback()
        logger.error(
            "order_transaction_failed",
            order_id=order.id,
            error=str(e),
            exc_info=True
        )
        raise OrderProcessingError(
            f"Failed to process order {order.id}"
        ) from e

    finally:
        # Always restore autocommit mode
        connection.autocommit = True
```

## Graceful Degradation

### Fallback Strategies

**Why**: Maintain partial functionality when dependencies fail.

```python
class UserService:
    """User service with graceful degradation."""

    def __init__(self, db: Database, cache: Cache):
        self.db = db
        self.cache = cache

    def get_user(self, user_id: int) -> User | None:
        """Get user with fallback to database if cache fails."""
        # Try cache first
        try:
            cached_user = self.cache.get(f"user:{user_id}")
            if cached_user:
                logger.debug("user_cache_hit", user_id=user_id)
                return cached_user
        except CacheError as e:
            # Cache failure shouldn't break the service
            logger.warning(
                "cache_error_fallback_to_database",
                user_id=user_id,
                error=str(e)
            )

        # Fallback to database
        try:
            user = self.db.get_user(user_id)

            # Try to update cache (best effort)
            try:
                self.cache.set(f"user:{user_id}", user)
            except CacheError as e:
                logger.warning("cache_update_failed", user_id=user_id, error=str(e))

            return user

        except DatabaseError as e:
            logger.error("user_fetch_failed", user_id=user_id, error=str(e))
            raise UserServiceError(f"Failed to fetch user {user_id}") from e
```

### Circuit Breaker Pattern

**Why**: Prevent cascading failures by temporarily disabling failing services.

```python
from datetime import datetime, timedelta

class CircuitBreaker:
    """Circuit breaker for external service calls."""

    def __init__(self, failure_threshold: int = 5, timeout: int = 60):
        self.failure_threshold = failure_threshold
        self.timeout = timeout
        self.failures = 0
        self.last_failure_time = None
        self.state = "closed"  # closed, open, half_open

    def call(self, func, *args, **kwargs):
        """Execute function with circuit breaker protection."""
        if self.state == "open":
            if datetime.now() - self.last_failure_time > timedelta(seconds=self.timeout):
                self.state = "half_open"
                logger.info("circuit_breaker_half_open")
            else:
                raise ServiceUnavailableError("Circuit breaker is open")

        try:
            result = func(*args, **kwargs)
            if self.state == "half_open":
                self.reset()
            return result

        except Exception as e:
            self.record_failure()
            raise

    def record_failure(self) -> None:
        """Record a failure and open circuit if threshold reached."""
        self.failures += 1
        self.last_failure_time = datetime.now()

        if self.failures >= self.failure_threshold:
            self.state = "open"
            logger.error(
                "circuit_breaker_opened",
                failures=self.failures,
                threshold=self.failure_threshold
            )

    def reset(self) -> None:
        """Reset circuit breaker to closed state."""
        self.failures = 0
        self.state = "closed"
        logger.info("circuit_breaker_closed")
```

## Related Standards

- See `python_style.md` for exception formatting
- See `documentation_standards.md` for documenting exceptions
- See `architecture_patterns.md` for error handling across layers

---

**Last Updated**: 2025-10-11
**Status**: Strong preference - deviations require justification

# Logging Standards

**Status**: Strong preference - deviations require justification and approval
**Scope**: Logging levels, formats, and structured logging for Python projects
**Last Updated**: 2025-10-31

---

## Overview

Effective logging is critical for debugging, monitoring, and understanding application behavior in production. Good logs are structured, searchable, and contain the right information without overwhelming the system or exposing sensitive data.

**Why**: Structured, well-leveled logging enables rapid debugging, effective monitoring, and operational insights without performance degradation.

## Log Levels

### Standard Log Levels (Use Python's logging module)

```python
import logging

# DEBUG: Detailed diagnostic information for development
logger.debug("cache_lookup", cache_key=key, cache_size=len(cache))

# INFO: General informational messages about application flow
logger.info("user_registered", user_id=user.id, email=user.email)

# WARNING: Unexpected situation, but application can continue
logger.warning("database_connection_slow", response_time_ms=1500, threshold_ms=1000)

# ERROR: Error condition, feature failed but application continues
logger.error("payment_processing_failed", order_id=order_id, error=str(e))

# CRITICAL: Severe error, application may be unable to continue
logger.critical("database_unavailable", error=str(e), retry_count=max_retries)
```

### When to Use Each Level

| Level | Use When | Examples |
|-------|----------|----------|
| **DEBUG** | Detailed diagnostic info for developers | Variable values, cache hits/misses, query execution |
| **INFO** | Significant business events or application state changes | User actions, successful operations, config changes |
| **WARNING** | Unexpected but handled situations | Deprecated API usage, slow performance, approaching limits |
| **ERROR** | Operation failed but application continues | Failed API calls, validation errors, processing failures |
| **CRITICAL** | System failure, application may not recover | Database unavailable, disk full, critical service down |

**Why**: Appropriate log levels enable filtering in production to show only relevant information.

## Structured Logging

### Use Structured Logging (structlog recommended)

**Why**: Structured logs are machine-readable, searchable, and easier to analyze than plain text.

```python
import structlog

# Configure structlog
structlog.configure(
    processors=[
        structlog.stdlib.add_log_level,
        structlog.stdlib.add_logger_name,
        structlog.processors.TimeStamper(fmt="iso"),
        structlog.processors.StackInfoRenderer(),
        structlog.processors.format_exc_info,
        structlog.processors.JSONRenderer()
    ],
    wrapper_class=structlog.stdlib.BoundLogger,
    context_class=dict,
    logger_factory=structlog.stdlib.LoggerFactory(),
    cache_logger_on_first_use=True,
)

logger = structlog.get_logger()

# Good: Structured logging with context
logger.info(
    "order_processed",
    order_id=order.id,
    customer_id=order.customer_id,
    total_amount=float(order.total),
    item_count=len(order.items),
    processing_time_ms=processing_time * 1000
)

# Bad: Unstructured string logging
logging.info(f"Processed order {order.id} for customer {order.customer_id}")
```

### Output Example

```json
{
  "event": "order_processed",
  "order_id": 12345,
  "customer_id": 67890,
  "total_amount": 149.99,
  "item_count": 3,
  "processing_time_ms": 234.5,
  "timestamp": "2025-10-31T14:32:10.123456Z",
  "level": "info",
  "logger": "order_service"
}
```

**Why**: JSON logs are easily indexed by log aggregation systems (ELK, Splunk, Datadog).

## What to Log

### DO Log

**Business Events**:
```python
# User actions
logger.info("user_login", user_id=user.id, ip_address=request.ip)
logger.info("order_placed", order_id=order.id, amount=order.total)

# State changes
logger.info("user_activated", user_id=user.id, activated_by=admin.id)
logger.info("subscription_renewed", subscription_id=sub.id, expires_at=sub.expires_at)
```

**Errors and Exceptions**:
```python
# With exception info
try:
    process_payment(order)
except PaymentError as e:
    logger.error(
        "payment_failed",
        order_id=order.id,
        error_type=type(e).__name__,
        error_message=str(e),
        exc_info=True  # Include stack trace
    )
```

**Performance Metrics**:
```python
# Slow operations
logger.warning(
    "slow_database_query",
    query="get_active_users",
    duration_ms=duration * 1000,
    threshold_ms=500
)

# Resource usage
logger.info(
    "cache_statistics",
    cache_size=len(cache),
    hit_rate=hits / (hits + misses),
    memory_mb=cache_memory / 1024 / 1024
)
```

**Security Events**:
```python
# Authentication attempts
logger.warning(
    "login_failed",
    user_email=email,
    ip_address=request.ip,
    attempt_count=attempts
)

# Authorization failures
logger.warning(
    "unauthorized_access_attempt",
    user_id=user.id,
    resource=resource_path,
    required_permission=permission
)
```

### DON'T Log

**Never Log Sensitive Information**:
```python
# Bad: Logs sensitive data
logger.info("user_login", password=password)  # NEVER!
logger.debug("payment_processing", credit_card=card_number)  # NEVER!
logger.info("api_call", api_key=api_key)  # NEVER!

# Good: Sanitize or omit sensitive data
logger.info("user_login", user_id=user.id)  # No password
logger.debug("payment_processing", card_last_four=card[-4:])  # Only last 4 digits
logger.info("api_call", api_key_prefix=api_key[:8])  # Only prefix
```

**Sensitive Data Includes**:
- Passwords, password hashes, security tokens
- API keys, secrets, credentials
- Full credit card numbers, CVV codes, SSN
- Personal health information (PHI)
- Personally identifiable information (PII) unless required

**Why**: Logs may be stored insecurely, viewed by many people, or accidentally exposed.

## Log Formatting

### Consistent Event Naming

Use snake_case for event names, make them descriptive:

```python
# Good: Descriptive, consistent event names
logger.info("user_registration_started", ...)
logger.info("user_registration_completed", ...)
logger.error("user_registration_failed", ...)

logger.info("payment_processing_started", ...)
logger.info("payment_authorized", ...)
logger.error("payment_declined", ...)

# Bad: Inconsistent, vague event names
logger.info("UserReg", ...)  # Inconsistent case
logger.info("done", ...)  # Too vague
logger.info("payment", ...)  # Ambiguous
```

### Include Contextual Information

```python
# Good: Rich context
logger.info(
    "api_request_completed",
    endpoint="/api/users",
    method="GET",
    status_code=200,
    duration_ms=123,
    user_id=user.id,
    request_id=request_id
)

# Bad: Minimal context
logger.info("request completed")
```

### Use Consistent Field Names

```python
# Good: Consistent field naming across logs
logger.info("user_created", user_id=user.id, ...)
logger.info("order_placed", user_id=user.id, order_id=order.id, ...)
logger.info("payment_processed", user_id=user.id, amount=amount, ...)

# Bad: Inconsistent field names
logger.info("user_created", id=user.id, ...)
logger.info("order_placed", customer=user.id, order=order.id, ...)
logger.info("payment_processed", user=user.id, total=amount, ...)
```

**Standard Field Names**:
- `user_id` (not `id`, `user`, `customer_id`)
- `order_id` (not `id`, `order`)
- `duration_ms` (not `time`, `duration`, `elapsed`)
- `error_message` (not `error`, `message`, `exception`)
- `timestamp` (ISO 8601 format)

## Configuration

### Basic Logger Setup

```python
# logger_config.py
import logging.config
import structlog

def configure_logging(log_level: str = "INFO", log_format: str = "json"):
    """Configure application logging."""

    # Standard library logging configuration
    logging.basicConfig(
        format="%(message)s",
        level=log_level.upper(),
        handlers=[logging.StreamHandler()]
    )

    # Structlog configuration
    processors = [
        structlog.stdlib.add_log_level,
        structlog.stdlib.add_logger_name,
        structlog.processors.TimeStamper(fmt="iso"),
        structlog.processors.StackInfoRenderer(),
        structlog.processors.format_exc_info,
    ]

    if log_format == "json":
        processors.append(structlog.processors.JSONRenderer())
    else:
        processors.append(structlog.dev.ConsoleRenderer())

    structlog.configure(
        processors=processors,
        wrapper_class=structlog.stdlib.BoundLogger,
        context_class=dict,
        logger_factory=structlog.stdlib.LoggerFactory(),
        cache_logger_on_first_use=True,
    )

# Usage in application
configure_logging(log_level="INFO", log_format="json")
logger = structlog.get_logger(__name__)
```

### Environment-Based Configuration

```python
import os
from pathlib import Path
from dotenv import load_dotenv

# Load environment variables
load_dotenv()

# Configure based on environment
environment = os.getenv("ENVIRONMENT", "development")

if environment == "production":
    log_level = "INFO"
    log_format = "json"
elif environment == "development":
    log_level = "DEBUG"
    log_format = "console"  # Human-readable for development
else:
    log_level = "WARNING"
    log_format = "json"

configure_logging(log_level=log_level, log_format=log_format)
```

## Performance Considerations

### Avoid Expensive Operations in Log Messages

```python
# Bad: Expensive operation always executed
logger.debug(f"User data: {expensive_serialize_user(user)}")

# Good: Only serialize if debug logging is enabled
if logger.isEnabledFor(logging.DEBUG):
    logger.debug("user_data", user_data=expensive_serialize_user(user))

# Better: Use lazy logging with lambda
from structlog import processors

logger.debug("user_data", user_data=lambda: expensive_serialize_user(user))
```

### Limit Log Volume in Hot Paths

```python
# Bad: Logs on every iteration
for item in large_dataset:
    logger.debug("processing_item", item_id=item.id)  # Could be millions!
    process(item)

# Good: Log summary or sample
logger.info("processing_batch_started", item_count=len(large_dataset))
for i, item in enumerate(large_dataset):
    process(item)
    # Log every 1000 items or on errors
    if i % 1000 == 0:
        logger.debug("processing_progress", processed=i, total=len(large_dataset))
logger.info("processing_batch_completed", item_count=len(large_dataset))
```

### Use Sampling for High-Volume Events

```python
import random

# Sample 1% of requests for detailed logging
if random.random() < 0.01:
    logger.debug(
        "detailed_request_info",
        headers=request.headers,
        body=request.body,
        sampled=True
    )
```

## Testing Logs

### Verify Important Logs in Tests

```python
import pytest
from logging import LogRecord

def test_order_processing_logs_success(caplog):
    """Test that successful order processing logs correctly."""
    # Arrange
    order = create_test_order()

    # Act
    with caplog.at_level(logging.INFO):
        process_order(order)

    # Assert - Check that success was logged
    assert any(
        record.message == "order_processed" and
        record.order_id == order.id
        for record in caplog.records
    )

def test_payment_failure_logs_error(caplog):
    """Test that payment failures are logged at ERROR level."""
    # Arrange
    order = create_test_order()

    # Act
    with caplog.at_level(logging.ERROR):
        with pytest.raises(PaymentError):
            process_payment(order, invalid_card)

    # Assert - Check error was logged
    assert any(
        record.levelname == "ERROR" and
        "payment_failed" in record.message
        for record in caplog.records
    )
```

## Common Patterns

### Request/Response Logging

```python
import time
import uuid

def log_request(func):
    """Decorator to log API requests."""
    def wrapper(request, *args, **kwargs):
        request_id = str(uuid.uuid4())
        start_time = time.time()

        logger.info(
            "api_request_started",
            request_id=request_id,
            method=request.method,
            path=request.path,
            user_id=getattr(request, 'user_id', None)
        )

        try:
            response = func(request, *args, **kwargs)
            duration_ms = (time.time() - start_time) * 1000

            logger.info(
                "api_request_completed",
                request_id=request_id,
                method=request.method,
                path=request.path,
                status_code=response.status_code,
                duration_ms=duration_ms
            )

            return response

        except Exception as e:
            duration_ms = (time.time() - start_time) * 1000

            logger.error(
                "api_request_failed",
                request_id=request_id,
                method=request.method,
                path=request.path,
                error_type=type(e).__name__,
                error_message=str(e),
                duration_ms=duration_ms,
                exc_info=True
            )
            raise

    return wrapper
```

### Database Query Logging

```python
def log_slow_queries(func):
    """Decorator to log slow database queries."""
    def wrapper(*args, **kwargs):
        start_time = time.time()
        result = func(*args, **kwargs)
        duration_ms = (time.time() - start_time) * 1000

        if duration_ms > 500:  # Log queries taking > 500ms
            logger.warning(
                "slow_query",
                function=func.__name__,
                duration_ms=duration_ms,
                threshold_ms=500
            )

        return result
    return wrapper

@log_slow_queries
def get_user_orders(user_id: int) -> list:
    """Get all orders for a user."""
    return db.execute(
        "SELECT * FROM orders WHERE customer_id = :user_id",
        {"user_id": user_id}
    ).fetchall()
```

### Background Task Logging

```python
def process_background_task(task_id: str):
    """Process a background task with comprehensive logging."""
    logger.info("background_task_started", task_id=task_id)

    try:
        # Process task
        result = perform_task(task_id)

        logger.info(
            "background_task_completed",
            task_id=task_id,
            result_summary=result.summary
        )

    except Exception as e:
        logger.error(
            "background_task_failed",
            task_id=task_id,
            error_type=type(e).__name__,
            error_message=str(e),
            exc_info=True
        )
        raise
```

## Log Aggregation

### Prepare Logs for Aggregation Systems

**Good log format for ELK stack, Splunk, Datadog**:
```json
{
  "timestamp": "2025-10-31T14:32:10.123456Z",
  "level": "info",
  "logger": "order_service",
  "event": "order_processed",
  "order_id": 12345,
  "customer_id": 67890,
  "total_amount": 149.99,
  "processing_time_ms": 234.5,
  "environment": "production",
  "service": "api",
  "version": "1.2.3"
}
```

**Include Standard Fields**:
- `timestamp`: ISO 8601 format
- `level`: Log level (debug, info, warning, error, critical)
- `logger`: Logger name (usually module name)
- `event`: Event type/name
- `environment`: production, staging, development
- `service`: Service name
- `version`: Application version

## Related Standards

- See `python_error_handling.md` for exception logging and error handling patterns
- See `python_style.md` for logger naming conventions
- See `performance_considerations.md` for logging performance optimization
- See `python_testing_standards.md` for testing logging behavior

---

**Last Updated**: 2025-10-31
**Status**: Strong preference - deviations require justification and approval

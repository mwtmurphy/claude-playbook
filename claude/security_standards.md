# Security Standards

**Status**: Strong preference - deviations require justification and approval
**Scope**: Authentication, authorization, secrets management, and input validation
**Last Updated**: 2025-10-31

---

## Overview

Security must be built into applications from the start, not added as an afterthought. These standards cover essential security practices for Python applications, focusing on preventing common vulnerabilities.

**Why**: Security vulnerabilities can lead to data breaches, financial losses, and loss of user trust. Following security best practices protects users and the business.

## Authentication

### Password Storage

**Never store passwords in plain text**. Always use secure hashing algorithms.

```python
from passlib.context import CryptContext

# Configure password hashing
pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")

# Good: Hash passwords before storing
def create_user(username: str, password: str) -> User:
    """Create user with securely hashed password."""
    password_hash = pwd_context.hash(password)
    user = User(username=username, password_hash=password_hash)
    return db.save(user)

# Good: Verify passwords against hash
def verify_password(plain_password: str, password_hash: str) -> bool:
    """Verify password against stored hash."""
    return pwd_context.verify(plain_password, password_hash)

# Bad: Storing plain text passwords
def create_user(username: str, password: str) -> User:
    """NEVER DO THIS!"""
    user = User(username=username, password=password)  # Plain text!
    return db.save(user)
```

**Why**: Bcrypt includes salts automatically and is designed to be slow, making brute-force attacks impractical.

### Password Requirements

```python
import re

def validate_password(password: str) -> tuple[bool, str]:
    """Validate password meets security requirements.

    Requirements:
    - At least 12 characters
    - At least one uppercase letter
    - At least one lowercase letter
    - At least one number
    - At least one special character
    """
    if len(password) < 12:
        return False, "Password must be at least 12 characters"

    if not re.search(r'[A-Z]', password):
        return False, "Password must contain at least one uppercase letter"

    if not re.search(r'[a-z]', password):
        return False, "Password must contain at least one lowercase letter"

    if not re.search(r'\d', password):
        return False, "Password must contain at least one number"

    if not re.search(r'[!@#$%^&*(),.?":{}|<>]', password):
        return False, "Password must contain at least one special character"

    return True, "Password meets requirements"
```

### JSON Web Tokens (JWT)

```python
from datetime import datetime, timedelta
import jwt

# Configuration
SECRET_KEY = os.getenv("JWT_SECRET_KEY")  # Never hardcode!
ALGORITHM = "HS256"
TOKEN_EXPIRY_MINUTES = 30

def create_access_token(user_id: int, email: str) -> str:
    """Create JWT access token."""
    payload = {
        "user_id": user_id,
        "email": email,
        "exp": datetime.utcnow() + timedelta(minutes=TOKEN_EXPIRY_MINUTES),
        "iat": datetime.utcnow()
    }
    token = jwt.encode(payload, SECRET_KEY, algorithm=ALGORITHM)
    return token

def verify_access_token(token: str) -> dict | None:
    """Verify and decode JWT token."""
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
        return payload
    except jwt.ExpiredSignatureError:
        logger.warning("token_expired", token_prefix=token[:16])
        return None
    except jwt.InvalidTokenError as e:
        logger.warning("invalid_token", error=str(e))
        return None

# Use in authentication
def authenticate_request(token: str) -> User | None:
    """Authenticate request using JWT token."""
    payload = verify_access_token(token)
    if not payload:
        return None

    user_id = payload.get("user_id")
    return get_user_by_id(user_id)
```

**Why**: JWT tokens are stateless, scalable, and can be verified without database lookups.

## Authorization

### Role-Based Access Control (RBAC)

```python
from enum import Enum
from functools import wraps

class Permission(Enum):
    """Define application permissions."""
    READ_USERS = "read:users"
    WRITE_USERS = "write:users"
    DELETE_USERS = "delete:users"
    READ_ORDERS = "read:orders"
    WRITE_ORDERS = "write:orders"
    ADMIN = "admin:*"

class Role(Enum):
    """Define user roles."""
    USER = "user"
    MANAGER = "manager"
    ADMIN = "admin"

# Role to permission mapping
ROLE_PERMISSIONS = {
    Role.USER: [
        Permission.READ_USERS,
        Permission.READ_ORDERS,
    ],
    Role.MANAGER: [
        Permission.READ_USERS,
        Permission.WRITE_USERS,
        Permission.READ_ORDERS,
        Permission.WRITE_ORDERS,
    ],
    Role.ADMIN: [Permission.ADMIN],  # All permissions
}

def user_has_permission(user: User, permission: Permission) -> bool:
    """Check if user has required permission."""
    user_role = Role(user.role)
    role_permissions = ROLE_PERMISSIONS.get(user_role, [])

    # Admin has all permissions
    if Permission.ADMIN in role_permissions:
        return True

    return permission in role_permissions

def require_permission(permission: Permission):
    """Decorator to require specific permission."""
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            # Get current user from context
            user = get_current_user()

            if not user_has_permission(user, permission):
                logger.warning(
                    "unauthorized_access",
                    user_id=user.id,
                    required_permission=permission.value,
                    function=func.__name__
                )
                raise PermissionDenied(
                    f"Permission required: {permission.value}"
                )

            return func(*args, **kwargs)
        return wrapper
    return decorator

# Usage
@require_permission(Permission.DELETE_USERS)
def delete_user(user_id: int) -> None:
    """Delete user (requires DELETE_USERS permission)."""
    db.delete_user(user_id)
```

### Resource-Level Authorization

```python
def can_modify_resource(user: User, resource: Resource) -> bool:
    """Check if user can modify specific resource."""
    # Owner can always modify
    if resource.owner_id == user.id:
        return True

    # Admin can modify anything
    if user.role == Role.ADMIN.value:
        return True

    # Manager can modify if in same team
    if user.role == Role.MANAGER.value and user.team_id == resource.team_id:
        return True

    return False

def get_order(order_id: int, user: User) -> Order:
    """Get order with authorization check."""
    order = db.get_order(order_id)

    if not order:
        raise NotFoundError(f"Order {order_id} not found")

    # User can only access their own orders (unless admin)
    if order.customer_id != user.id and user.role != Role.ADMIN.value:
        logger.warning(
            "unauthorized_order_access",
            user_id=user.id,
            order_id=order_id,
            order_owner=order.customer_id
        )
        raise PermissionDenied("Cannot access this order")

    return order
```

## Secrets Management

### Never Hardcode Secrets

```python
# Bad: Hardcoded secrets
API_KEY = "sk_live_1234567890abcdef"  # NEVER!
DATABASE_PASSWORD = "my_secret_password"  # NEVER!

# Good: Load from environment variables
import os
from dotenv import load_dotenv

load_dotenv()

API_KEY = os.getenv("API_KEY")
DATABASE_PASSWORD = os.getenv("DATABASE_PASSWORD")

if not API_KEY:
    raise ValueError("API_KEY environment variable not set")
```

### Environment Variables

```bash
# .env (NEVER commit to git!)
API_KEY=sk_live_1234567890abcdef
DATABASE_PASSWORD=my_secret_password
JWT_SECRET_KEY=your-secret-key-here
STRIPE_API_KEY=sk_test_xyz123
```

```gitignore
# .gitignore
.env
.env.local
.env.*.local
```

### Secrets in Configuration

```python
from dataclasses import dataclass
from pathlib import Path
import os

@dataclass
class SecretConfig:
    """Configuration for application secrets."""
    api_key: str
    database_password: str
    jwt_secret: str

    @classmethod
    def from_env(cls) -> "SecretConfig":
        """Load secrets from environment variables."""
        return cls(
            api_key=os.getenv("API_KEY", ""),
            database_password=os.getenv("DATABASE_PASSWORD", ""),
            jwt_secret=os.getenv("JWT_SECRET_KEY", "")
        )

    def validate(self) -> None:
        """Validate that all required secrets are present."""
        if not self.api_key:
            raise ValueError("API_KEY is required")
        if not self.database_password:
            raise ValueError("DATABASE_PASSWORD is required")
        if not self.jwt_secret:
            raise ValueError("JWT_SECRET_KEY is required")

# Usage
secrets = SecretConfig.from_env()
secrets.validate()
```

## Input Validation

### Validate All User Input

```python
from pydantic import BaseModel, EmailStr, Field, validator

class UserCreateRequest(BaseModel):
    """Request model for creating users."""
    username: str = Field(..., min_length=3, max_length=50, regex=r'^[a-zA-Z0-9_]+$')
    email: EmailStr
    password: str = Field(..., min_length=12)
    age: int = Field(..., ge=18, le=120)

    @validator('password')
    def validate_password(cls, v):
        """Validate password meets security requirements."""
        is_valid, message = validate_password(v)
        if not is_valid:
            raise ValueError(message)
        return v

# Usage in endpoint
def create_user_endpoint(request_data: dict) -> dict:
    """Create user with validated input."""
    try:
        # Validate input
        validated_data = UserCreateRequest(**request_data)

        # Create user
        user = create_user(
            username=validated_data.username,
            email=validated_data.email,
            password=validated_data.password
        )

        return {"success": True, "user_id": user.id}

    except ValidationError as e:
        logger.warning("invalid_user_input", errors=e.errors())
        raise BadRequest(str(e))
```

### SQL Injection Prevention

```python
# Good: Use parameterized queries
def get_user_by_email(email: str) -> User | None:
    """Get user by email (SQL injection safe)."""
    query = "SELECT * FROM users WHERE email = :email"
    result = db.execute(query, {"email": email}).fetchone()
    return User(**result) if result else None

# Bad: String concatenation (SQL injection vulnerability!)
def get_user_by_email(email: str) -> User | None:
    """NEVER DO THIS!"""
    query = f"SELECT * FROM users WHERE email = '{email}'"  # Vulnerable!
    result = db.execute(query).fetchone()
    return User(**result) if result else None
```

**Why**: Parameterized queries prevent SQL injection by treating user input as data, not code.

### Path Traversal Prevention

```python
from pathlib import Path

def read_user_file(user_id: int, filename: str) -> str:
    """Read user file with path traversal prevention."""
    # Define allowed directory
    user_dir = Path(f"/data/users/{user_id}")
    requested_file = user_dir / filename

    # Resolve to absolute path and check if it's within allowed directory
    resolved_path = requested_file.resolve()

    if not str(resolved_path).startswith(str(user_dir.resolve())):
        logger.warning(
            "path_traversal_attempt",
            user_id=user_id,
            requested=filename,
            resolved=str(resolved_path)
        )
        raise PermissionDenied("Invalid file path")

    # Read file if it exists and is within allowed directory
    if resolved_path.exists() and resolved_path.is_file():
        return resolved_path.read_text()

    raise NotFoundError(f"File {filename} not found")
```

## HTTPS/TLS

### Always Use HTTPS in Production

```python
# FastAPI example
from fastapi import FastAPI, Request
from fastapi.middleware.httpsredirect import HTTPSRedirectMiddleware

app = FastAPI()

# Redirect HTTP to HTTPS in production
if os.getenv("ENVIRONMENT") == "production":
    app.add_middleware(HTTPSRedirectMiddleware)

# Check for HTTPS in requests
@app.middleware("http")
async def enforce_https(request: Request, call_next):
    """Enforce HTTPS in production."""
    if os.getenv("ENVIRONMENT") == "production":
        if request.url.scheme != "https":
            logger.warning(
                "insecure_request",
                url=str(request.url),
                ip=request.client.host
            )
            return JSONResponse(
                status_code=403,
                content={"error": "HTTPS required"}
            )

    response = await call_next(request)
    return response
```

## Security Headers

### Set Security HTTP Headers

```python
from fastapi import FastAPI, Response

app = FastAPI()

@app.middleware("http")
async def add_security_headers(request: Request, call_next):
    """Add security headers to all responses."""
    response = await call_next(request)

    # Prevent clickjacking
    response.headers["X-Frame-Options"] = "DENY"

    # Prevent MIME type sniffing
    response.headers["X-Content-Type-Options"] = "nosniff"

    # Enable XSS protection
    response.headers["X-XSS-Protection"] = "1; mode=block"

    # Content Security Policy
    response.headers["Content-Security-Policy"] = "default-src 'self'"

    # HSTS (HTTPS only)
    if request.url.scheme == "https":
        response.headers["Strict-Transport-Security"] = "max-age=31536000; includeSubDomains"

    return response
```

## Rate Limiting

### Prevent Abuse with Rate Limiting

```python
from datetime import datetime, timedelta
from collections import defaultdict

class RateLimiter:
    """Simple in-memory rate limiter."""

    def __init__(self, max_requests: int, window_seconds: int):
        self.max_requests = max_requests
        self.window_seconds = window_seconds
        self.requests = defaultdict(list)

    def is_allowed(self, key: str) -> bool:
        """Check if request is allowed under rate limit."""
        now = datetime.now()
        window_start = now - timedelta(seconds=self.window_seconds)

        # Remove old requests
        self.requests[key] = [
            req_time for req_time in self.requests[key]
            if req_time > window_start
        ]

        # Check if under limit
        if len(self.requests[key]) >= self.max_requests:
            logger.warning(
                "rate_limit_exceeded",
                key=key,
                requests_in_window=len(self.requests[key]),
                max_requests=self.max_requests
            )
            return False

        # Record this request
        self.requests[key].append(now)
        return True

# Usage
rate_limiter = RateLimiter(max_requests=100, window_seconds=60)

def login_endpoint(email: str, password: str) -> dict:
    """Login with rate limiting."""
    # Rate limit by IP or email
    if not rate_limiter.is_allowed(f"login:{email}"):
        raise TooManyRequests("Rate limit exceeded. Try again later.")

    # Process login
    user = authenticate_user(email, password)
    return {"token": create_access_token(user.id, user.email)}
```

## Data Sanitization

### Sanitize Output

```python
import html
from typing import Any

def sanitize_html(text: str) -> str:
    """Escape HTML to prevent XSS attacks."""
    return html.escape(text)

def sanitize_user_data(user_data: dict) -> dict:
    """Sanitize user data before returning to client."""
    return {
        "id": user_data["id"],
        "username": sanitize_html(user_data["username"]),
        "email": user_data["email"],
        "bio": sanitize_html(user_data.get("bio", "")),
        # Never return password_hash or sensitive fields
    }
```

## Security Checklist

Before deploying to production:

- [ ] All secrets loaded from environment variables, none hardcoded
- [ ] Passwords hashed with bcrypt or argon2
- [ ] HTTPS enforced in production
- [ ] Security headers configured
- [ ] Input validation on all endpoints
- [ ] Parameterized queries for all database access
- [ ] Rate limiting on authentication endpoints
- [ ] Authorization checks on all protected resources
- [ ] Sensitive data not logged
- [ ] Dependencies scanned for vulnerabilities (`pip-audit`)
- [ ] CORS configured appropriately
- [ ] CSRF protection enabled for state-changing operations

## Vulnerability Scanning

### Scan Dependencies Regularly

```bash
# Install pip-audit
poetry add --group dev pip-audit

# Scan for vulnerabilities
poetry run pip-audit

# In CI/CD pipeline
poetry run pip-audit --strict  # Fail if vulnerabilities found
```

```yaml
# GitHub Actions example
name: Security Scan

on: [push, pull_request]

jobs:
  security:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-python@v4
      - name: Install dependencies
        run: |
          pip install poetry
          poetry install
      - name: Run security audit
        run: poetry run pip-audit
```

## Related Standards

- See `python_style.md` for secure coding conventions
- See `python_error_handling.md` for secure error handling (avoid leaking sensitive info)
- See `logging_standards.md` for secure logging (never log secrets)
- See `python_environment_setup.md` for environment variable management
- See `api_design_standards.md` for API authentication and authorization patterns

---

**Last Updated**: 2025-10-31
**Status**: Strong preference - deviations require justification and approval

# API Design Standards

**Status**: Strong preference - deviations require justification and approval
**Scope**: REST API design, versioning, error responses, and HTTP best practices
**Last Updated**: 2025-10-31

---

## Overview

Well-designed APIs are intuitive, consistent, and maintainable. These standards define RESTful API conventions for endpoint design, request/response formats, versioning, and error handling.

**Why**: Consistent API design reduces learning curve, improves developer experience, and makes APIs easier to maintain and evolve.

## REST Principles

### Resource-Oriented Design

APIs should be built around resources (nouns), not actions (verbs).

```python
# Good: Resource-oriented endpoints
GET    /users              # List users
GET    /users/123          # Get user 123
POST   /users              # Create new user
PUT    /users/123          # Update user 123
DELETE /users/123          # Delete user 123

GET    /users/123/orders   # Get orders for user 123

# Bad: Action-oriented endpoints
POST   /getUser            # Should be GET /users/123
POST   /createUser         # Should be POST /users
POST   /deleteUser/123     # Should be DELETE /users/123
```

### Use HTTP Methods Correctly

| Method | Purpose | Idempotent | Request Body | Response Body |
|--------|---------|------------|--------------|---------------|
| GET | Retrieve resource(s) | Yes | No | Yes |
| POST | Create resource | No | Yes | Yes (created resource) |
| PUT | Update/replace resource | Yes | Yes | Yes (updated resource) |
| PATCH | Partial update | No | Yes | Yes (updated resource) |
| DELETE | Delete resource | Yes | No | Optional |

**Why**: Standard HTTP method usage makes APIs predictable and cacheable.

## URL Structure

### Naming Conventions

```python
# Good: Clear, hierarchical URLs
/api/v1/users
/api/v1/users/123
/api/v1/users/123/orders
/api/v1/users/123/orders/456
/api/v1/orders?status=pending
/api/v1/products?category=electronics&limit=20

# Bad: Unclear or inconsistent URLs
/api/v1/getUsers
/api/v1/user-management
/api/v1/user_data/123
/api/v1/123/user  # ID before resource
```

**Rules**:
- Use plural nouns for collections (`/users`, not `/user`)
- Use lowercase letters
- Use hyphens for multi-word resources (`/product-categories`)
- Keep URLs hierarchical and logical
- Avoid deep nesting (max 2-3 levels)

### Query Parameters

```python
# Filtering
GET /api/v1/users?status=active&role=admin

# Sorting
GET /api/v1/products?sort=price&order=desc

# Pagination
GET /api/v1/orders?page=2&limit=50
GET /api/v1/orders?offset=100&limit=50

# Field selection (partial response)
GET /api/v1/users/123?fields=id,name,email

# Search
GET /api/v1/products?q=laptop&category=electronics
```

## Request Format

### JSON Request Bodies

```python
# Good: Clean JSON structure
{
  "username": "alice",
  "email": "alice@example.com",
  "profile": {
    "first_name": "Alice",
    "last_name": "Smith"
  }
}

# Bad: Inconsistent or unclear structure
{
  "UserName": "alice",  # Inconsistent casing
  "EMail": "alice@example.com",
  "profile_first_name": "Alice",  # Flat when hierarchical is better
  "profile_last_name": "Smith"
}
```

**Field naming**:
- Use `snake_case` for JSON fields (matches Python conventions)
- Be consistent across all endpoints
- Use descriptive names

### Content Negotiation

```python
from fastapi import FastAPI, Header

app = FastAPI()

@app.post("/api/v1/users")
async def create_user(
    user_data: UserCreateRequest,
    content_type: str | None = Header(None)
):
    """Create user with content type validation."""
    if content_type != "application/json":
        raise BadRequest("Content-Type must be application/json")

    user = create_user_service(user_data)
    return user
```

## Response Format

### Successful Responses

```python
# GET /api/v1/users/123 - Single resource
{
  "id": 123,
  "username": "alice",
  "email": "alice@example.com",
  "created_at": "2025-10-31T14:30:00Z",
  "updated_at": "2025-10-31T15:45:00Z"
}

# GET /api/v1/users - Collection with pagination
{
  "data": [
    {
      "id": 123,
      "username": "alice",
      "email": "alice@example.com"
    },
    {
      "id": 124,
      "username": "bob",
      "email": "bob@example.com"
    }
  ],
  "pagination": {
    "page": 1,
    "limit": 50,
    "total": 150,
    "pages": 3
  }
}

# POST /api/v1/users - Created resource
# Status: 201 Created
# Location: /api/v1/users/125
{
  "id": 125,
  "username": "charlie",
  "email": "charlie@example.com",
  "created_at": "2025-10-31T16:00:00Z"
}

# DELETE /api/v1/users/123 - Successful deletion
# Status: 204 No Content
# (Empty response body)
```

### Error Responses

**Standard error format**:

```python
from pydantic import BaseModel
from typing import Optional

class ErrorResponse(BaseModel):
    """Standard API error response."""
    error: str  # Error type/code
    message: str  # Human-readable message
    details: Optional[dict] = None  # Additional context
    request_id: Optional[str] = None  # For tracking
    timestamp: str  # ISO 8601 timestamp

# 400 Bad Request - Validation error
{
  "error": "VALIDATION_ERROR",
  "message": "Invalid request data",
  "details": {
    "email": ["Invalid email format"],
    "age": ["Must be at least 18"]
  },
  "request_id": "req_abc123",
  "timestamp": "2025-10-31T16:00:00Z"
}

# 401 Unauthorized
{
  "error": "UNAUTHORIZED",
  "message": "Authentication required",
  "request_id": "req_def456",
  "timestamp": "2025-10-31T16:00:00Z"
}

# 403 Forbidden
{
  "error": "FORBIDDEN",
  "message": "Insufficient permissions to access this resource",
  "details": {
    "required_permission": "write:users"
  },
  "request_id": "req_ghi789",
  "timestamp": "2025-10-31T16:00:00Z"
}

# 404 Not Found
{
  "error": "NOT_FOUND",
  "message": "User not found",
  "details": {
    "user_id": 999
  },
  "request_id": "req_jkl012",
  "timestamp": "2025-10-31T16:00:00Z"
}

# 429 Too Many Requests
{
  "error": "RATE_LIMIT_EXCEEDED",
  "message": "Too many requests. Please try again later.",
  "details": {
    "retry_after": 60
  },
  "request_id": "req_mno345",
  "timestamp": "2025-10-31T16:00:00Z"
}

# 500 Internal Server Error
{
  "error": "INTERNAL_SERVER_ERROR",
  "message": "An unexpected error occurred",
  "request_id": "req_pqr678",
  "timestamp": "2025-10-31T16:00:00Z"
}
```

**Why**: Consistent error format makes it easy for clients to handle errors programmatically.

## HTTP Status Codes

### Use Appropriate Status Codes

**Success codes**:
- `200 OK` - Successful GET, PUT, PATCH
- `201 Created` - Successful POST (resource created)
- `204 No Content` - Successful DELETE, or successful operation with no response body

**Client error codes**:
- `400 Bad Request` - Invalid request data (validation errors)
- `401 Unauthorized` - Authentication required or failed
- `403 Forbidden` - Authenticated but insufficient permissions
- `404 Not Found` - Resource not found
- `409 Conflict` - Request conflicts with current state (e.g., duplicate)
- `422 Unprocessable Entity` - Validation errors
- `429 Too Many Requests` - Rate limit exceeded

**Server error codes**:
- `500 Internal Server Error` - Unexpected server error
- `503 Service Unavailable` - Service temporarily unavailable

```python
from fastapi import FastAPI, HTTPException, status
from fastapi.responses import JSONResponse

app = FastAPI()

@app.post("/api/v1/users", status_code=status.HTTP_201_CREATED)
async def create_user(user_data: UserCreateRequest):
    """Create new user."""
    # Validate
    if user_exists(user_data.email):
        raise HTTPException(
            status_code=status.HTTP_409_CONFLICT,
            detail={
                "error": "USER_EXISTS",
                "message": "User with this email already exists"
            }
        )

    # Create
    user = create_user_service(user_data)

    # Return 201 with Location header
    return JSONResponse(
        status_code=status.HTTP_201_CREATED,
        content=user.dict(),
        headers={"Location": f"/api/v1/users/{user.id}"}
    )

@app.delete("/api/v1/users/{user_id}", status_code=status.HTTP_204_NO_CONTENT)
async def delete_user(user_id: int):
    """Delete user."""
    if not user_exists_by_id(user_id):
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail={
                "error": "NOT_FOUND",
                "message": f"User {user_id} not found"
            }
        )

    delete_user_service(user_id)
    return None  # 204 No Content
```

## Pagination

### Cursor-Based Pagination (Recommended for large datasets)

```python
# Request
GET /api/v1/orders?limit=50&cursor=eyJpZCI6MTIzNDV9

# Response
{
  "data": [ /* 50 orders */ ],
  "pagination": {
    "limit": 50,
    "next_cursor": "eyJpZCI6MTIzOTV9",
    "has_more": true
  }
}
```

### Offset-Based Pagination (Simpler, for smaller datasets)

```python
# Request
GET /api/v1/users?page=2&limit=50

# Response
{
  "data": [ /* 50 users */ ],
  "pagination": {
    "page": 2,
    "limit": 50,
    "total": 150,
    "pages": 3,
    "has_next": true,
    "has_prev": true
  }
}
```

**Implementation**:

```python
from pydantic import BaseModel
from typing import Generic, TypeVar, Optional

T = TypeVar('T')

class PaginatedResponse(BaseModel, Generic[T]):
    """Generic paginated response."""
    data: list[T]
    pagination: "PaginationInfo"

class PaginationInfo(BaseModel):
    """Pagination metadata."""
    page: int
    limit: int
    total: int
    pages: int
    has_next: bool
    has_prev: bool

@app.get("/api/v1/users")
async def list_users(page: int = 1, limit: int = 50) -> PaginatedResponse[User]:
    """List users with pagination."""
    # Validate limits
    limit = min(limit, 100)  # Cap at 100

    offset = (page - 1) * limit
    users = get_users_paginated(offset=offset, limit=limit)
    total = count_users()

    return PaginatedResponse(
        data=users,
        pagination=PaginationInfo(
            page=page,
            limit=limit,
            total=total,
            pages=(total + limit - 1) // limit,
            has_next=offset + limit < total,
            has_prev=page > 1
        )
    )
```

## Versioning

### URL Versioning (Recommended)

```python
# Include version in URL path
GET /api/v1/users
GET /api/v2/users

# Implementation
from fastapi import FastAPI, APIRouter

app = FastAPI()

# V1 router
v1_router = APIRouter(prefix="/api/v1")

@v1_router.get("/users")
async def get_users_v1():
    """V1 endpoint."""
    return get_users()

# V2 router
v2_router = APIRouter(prefix="/api/v2")

@v2_router.get("/users")
async def get_users_v2():
    """V2 endpoint with new features."""
    return get_users_enhanced()

app.include_router(v1_router)
app.include_router(v2_router)
```

**When to version**:
- Breaking changes to request/response format
- Removing fields
- Changing data types
- Changing behavior

**When NOT to version**:
- Adding new optional fields
- Adding new endpoints
- Bug fixes
- Performance improvements

## Authentication

### Bearer Token Authentication

```python
from fastapi import Header, HTTPException, Depends

async def get_current_user(
    authorization: str | None = Header(None)
) -> User:
    """Extract and validate bearer token."""
    if not authorization:
        raise HTTPException(
            status_code=401,
            detail={
                "error": "UNAUTHORIZED",
                "message": "Authentication required"
            }
        )

    # Extract token
    scheme, _, token = authorization.partition(" ")

    if scheme.lower() != "bearer":
        raise HTTPException(
            status_code=401,
            detail={
                "error": "INVALID_AUTH_SCHEME",
                "message": "Authentication scheme must be Bearer"
            }
        )

    # Verify token
    user = verify_token(token)
    if not user:
        raise HTTPException(
            status_code=401,
            detail={
                "error": "INVALID_TOKEN",
                "message": "Invalid or expired token"
            }
        )

    return user

# Usage
@app.get("/api/v1/users/me")
async def get_current_user_info(
    current_user: User = Depends(get_current_user)
):
    """Get current user info (requires authentication)."""
    return current_user
```

## Rate Limiting

### Include Rate Limit Headers

```python
from fastapi import Response

@app.get("/api/v1/users")
async def list_users(response: Response):
    """List users with rate limit headers."""
    # Check rate limit
    remaining = check_rate_limit(request.client.host)

    # Add rate limit headers
    response.headers["X-RateLimit-Limit"] = "100"
    response.headers["X-RateLimit-Remaining"] = str(remaining)
    response.headers["X-RateLimit-Reset"] = "1635699600"

    users = get_users()
    return users
```

## CORS Configuration

```python
from fastapi.middleware.cors import CORSMiddleware

app.add_middleware(
    CORSMiddleware,
    allow_origins=["https://example.com"],  # Specific origins in production
    allow_credentials=True,
    allow_methods=["GET", "POST", "PUT", "DELETE"],
    allow_headers=["Authorization", "Content-Type"],
)
```

## API Documentation

### Use OpenAPI/Swagger

```python
from fastapi import FastAPI
from pydantic import BaseModel, Field

app = FastAPI(
    title="My API",
    description="API for managing users and orders",
    version="1.0.0",
    docs_url="/api/docs",
    redoc_url="/api/redoc"
)

class UserCreateRequest(BaseModel):
    """Request model for creating users."""
    username: str = Field(..., description="Unique username", example="alice")
    email: str = Field(..., description="User email address", example="alice@example.com")
    password: str = Field(..., description="User password (min 12 chars)")

    class Config:
        schema_extra = {
            "example": {
                "username": "alice",
                "email": "alice@example.com",
                "password": "SecurePassword123!"
            }
        }

@app.post(
    "/api/v1/users",
    status_code=201,
    response_model=User,
    summary="Create new user",
    description="Create a new user account with username, email, and password",
    responses={
        201: {"description": "User created successfully"},
        400: {"description": "Invalid request data"},
        409: {"description": "User already exists"}
    }
)
async def create_user(user_data: UserCreateRequest):
    """Create new user account."""
    return create_user_service(user_data)
```

## Best Practices Checklist

API design checklist:

- [ ] RESTful resource-oriented URLs
- [ ] Appropriate HTTP methods and status codes
- [ ] Consistent JSON field naming (snake_case)
- [ ] Standardized error response format
- [ ] Pagination for list endpoints
- [ ] Filtering and sorting query parameters
- [ ] API versioning strategy
- [ ] Authentication and authorization
- [ ] Rate limiting
- [ ] CORS configuration
- [ ] Comprehensive API documentation (OpenAPI/Swagger)
- [ ] Request/response validation (Pydantic)
- [ ] Security headers
- [ ] HTTPS only in production

## Related Standards

- See `python_style.md` for Python code formatting
- See `security_standards.md` for authentication, authorization, and input validation
- See `error_handling.md` for exception handling in API endpoints
- See `logging_standards.md` for API request/response logging
- See `documentation_standards.md` for API documentation requirements

---

**Last Updated**: 2025-10-31
**Status**: Strong preference - deviations require justification and approval

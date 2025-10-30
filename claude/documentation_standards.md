# Documentation Standards

**Status**: Strong preference - deviations require justification and approval
**Scope**: Documentation requirements for all code, projects, and playbook standards

## Overview

Good documentation makes code accessible, maintainable, and collaborative. Documentation should explain the "why" more than the "what" - code should be self-documenting for the "what".

**Why**: Well-documented code reduces onboarding time, prevents misuse, and serves as a reference for future maintenance.

## Language and Style

### British English Required

**Scope**: All documentation - code docstrings, comments, READMEs, and playbook standards.

**Why**: Consistency in spelling and terminology across all documentation improves professionalism and reduces confusion. British English is the standard for this organisation.

#### Common British vs American Spelling Differences

```python
# Good: British English
def initialise_colour_scheme(behaviour: str) -> None:
    """Initialise the colour scheme based on user behaviour.

    Analyses user preferences and organises the colour palette
    to optimise the visualisation experience.

    Args:
        behaviour: The user behaviour pattern to analyse.
    """
    pass


# Bad: American English
def initialize_color_scheme(behavior: str) -> None:
    """Initialize the color scheme based on user behavior.

    Analyzes user preferences and organizes the color palette
    to optimize the visualization experience.

    Args:
        behavior: The user behavior pattern to analyze.
    """
    pass
```

#### Key Spelling Rules

- **-ise vs -ize**: Use -ise (organise, realise, recognise)
- **-our vs -or**: Use -our (colour, behaviour, favour)
- **-re vs -er**: Use -re (centre, metre, litre)
- **-ogue vs -og**: Use -ogue (catalogue, dialogue, analogue)
- **-ence vs -ense**: Use -ence (defence, licence as noun)
- **-ll- vs -l-**: Use double-l (travelling, modelling, cancelled)

#### README and Documentation

```markdown
# Good: British English
## Background

This project analyses user behaviour patterns to optimise resource allocation.
It synchronises data across distributed centres while minimising latency.

## Configuration

Customise the colour scheme by editing `config.yml`. The system will
automatically recognise changes and reinitialise the visualisation engine.

# Bad: American English
## Background

This project analyzes user behavior patterns to optimize resource allocation.
It synchronizes data across distributed centers while minimizing latency.

## Configuration

Customize the color scheme by editing `config.yml`. The system will
automatically recognize changes and reinitialize the visualization engine.
```

## Readability Best Practices

### Plain Language Principles

**Why**: Clear, accessible documentation reduces cognitive load and accelerates understanding.

#### Write Short, Focused Sentences

Aim for 15-20 words per sentence. Break complex ideas into multiple sentences.

```python
# Good: Clear, concise sentences
def validate_transaction(transaction: Transaction) -> ValidationResult:
    """Validate a financial transaction.

    This function checks three criteria. First, it verifies the account
    has sufficient funds. Second, it confirms the transaction type is
    allowed. Third, it validates the recipient account exists.

    Args:
        transaction: The transaction to validate.

    Returns:
        ValidationResult indicating success or specific failure reason.
    """
    pass


# Bad: Long, complex sentence
def validate_transaction(transaction: Transaction) -> ValidationResult:
    """Validate a financial transaction by checking that the account has
    sufficient funds and that the transaction type is allowed for this account
    and that the recipient account exists and is active in the system.

    Args:
        transaction: The transaction to validate.

    Returns:
        ValidationResult indicating success or specific failure reason.
    """
    pass
```

#### Use Active Voice

Active voice is clearer and more direct than passive voice.

```python
# Good: Active voice
def process_payment(amount: float) -> Receipt:
    """Process a payment and return a receipt.

    The system validates the amount, charges the account, and generates
    a receipt. If validation fails, the function raises an exception.
    """
    pass


# Bad: Passive voice
def process_payment(amount: float) -> Receipt:
    """Process a payment and return a receipt.

    The amount is validated by the system, the account is charged, and
    a receipt is generated. If validation fails, an exception is raised.
    """
    pass
```

#### Define Technical Terms

When using domain-specific terminology, provide brief context or definitions.

```python
# Good: Terms explained
def calculate_apr(principal: float, rate: float) -> float:
    """Calculate the Annual Percentage Rate (APR).

    APR represents the yearly cost of a loan including interest and fees.
    This differs from the nominal interest rate, which excludes fees.

    Args:
        principal: The loan amount.
        rate: The nominal interest rate as a decimal (e.g., 0.05 for 5%).

    Returns:
        The APR as a decimal.
    """
    pass


# Bad: Undefined jargon
def calculate_apr(principal: float, rate: float) -> float:
    """Calculate the APR.

    Args:
        principal: The loan amount.
        rate: The rate.

    Returns:
        The APR.
    """
    pass
```

### Structure and Formatting

**Why**: Well-structured documentation is scannable and helps readers find information quickly.

#### Use Clear Heading Hierarchy

Organise content with descriptive headings that follow a logical hierarchy.

```markdown
# Good: Clear hierarchy
# User Authentication Module

## Overview
Brief description of the module.

## Authentication Methods

### Password Authentication
How password auth works.

### OAuth Integration
How OAuth works.

### Two-Factor Authentication
How 2FA works.

## Configuration

### Required Settings
List of required settings.

### Optional Settings
List of optional settings.


# Bad: Flat structure
# User Authentication Module

Description of password authentication.

Description of OAuth.

Description of 2FA.

Configuration settings.
```

#### Use Lists for Multiple Items

Lists make information easier to scan and remember than prose.

```python
# Good: List format
def setup_database(config: dict) -> None:
    """Set up the database connection.

    This function performs the following steps:
    - Validates the configuration parameters
    - Establishes a connection to the database
    - Runs initial migration scripts
    - Verifies the schema matches expectations
    - Sets up connection pooling

    Args:
        config: Database configuration dictionary.
    """
    pass


# Bad: Prose format
def setup_database(config: dict) -> None:
    """Set up the database connection.

    This function validates the configuration parameters and then establishes
    a connection to the database and runs initial migration scripts and verifies
    the schema matches expectations and sets up connection pooling.

    Args:
        config: Database configuration dictionary.
    """
    pass
```

#### Provide Context for Code Examples

Always explain what an example demonstrates and why it matters.

```python
# Good: Example with context
class UserRepository:
    """Repository for user data operations.

    Example:
        Create a repository and fetch a user by ID:

        >>> repo = UserRepository(db_connection)
        >>> user = repo.get_by_id(123)
        >>> print(user.username)
        'alice'
    """
    pass


# Bad: Example without context
class UserRepository:
    """Repository for user data operations.

    Example:
        >>> repo = UserRepository(db_connection)
        >>> user = repo.get_by_id(123)
    """
    pass
```

#### Use White Space Effectively

Break up dense content with blank lines and spacing.

```python
# Good: Readable spacing
def complex_calculation(
    base_amount: float,
    tax_rate: float,
    discount: float,
    shipping: float
) -> float:
    """Calculate the final price with tax, discount, and shipping.

    The calculation follows this sequence:
    1. Apply discount to base amount
    2. Add shipping cost
    3. Calculate tax on subtotal
    4. Return final total

    Args:
        base_amount: The original price before adjustments.
        tax_rate: Tax rate as a decimal (e.g., 0.20 for 20%).
        discount: Discount amount to subtract.
        shipping: Shipping cost to add.

    Returns:
        The final price including all adjustments.

    Example:
        >>> calculate_final_price(100.0, 0.20, 10.0, 5.0)
        114.0
    """
    # Apply discount
    discounted = base_amount - discount

    # Add shipping
    subtotal = discounted + shipping

    # Calculate tax
    tax = subtotal * tax_rate

    # Return total
    return subtotal + tax


# Bad: Dense, hard to parse
def complex_calculation(base_amount: float, tax_rate: float, discount: float, shipping: float) -> float:
    """Calculate the final price with tax, discount, and shipping.
    Args:
        base_amount: The original price before adjustments.
        tax_rate: Tax rate as a decimal (e.g., 0.20 for 20%).
        discount: Discount amount to subtract.
        shipping: Shipping cost to add.
    Returns:
        The final price including all adjustments.
    """
    discounted = base_amount - discount
    subtotal = discounted + shipping
    tax = subtotal * tax_rate
    return subtotal + tax
```

### Technical Writing Standards

**Why**: Professional technical writing ensures documentation is trustworthy and maintainable.

#### Maintain Consistent Terminology

Use the same term for the same concept throughout all documentation.

```python
# Good: Consistent terminology
class UserAccount:
    """Manage user account data.

    A user account contains profile information and preferences.
    Create an account with create_account(). Retrieve an account
    with get_account(). Update an account with update_account().
    """
    pass


# Bad: Inconsistent terminology
class UserAccount:
    """Manage user account data.

    A profile contains user information and settings.
    Create a record with create_account(). Retrieve the user
    with get_account(). Update the profile with update_account().
    """
    pass
```

#### Be Precise Without Being Verbose

Include necessary details, but avoid unnecessary words.

```python
# Good: Precise and concise
def send_email(recipient: str, subject: str, body: str) -> bool:
    """Send an email to the specified recipient.

    Args:
        recipient: Email address of the recipient.
        subject: Email subject line.
        body: Email body content.

    Returns:
        True if email was sent successfully, False otherwise.
    """
    pass


# Bad: Verbose
def send_email(recipient: str, subject: str, body: str) -> bool:
    """This function is responsible for sending an email message to the
    recipient that is specified by the recipient parameter.

    Args:
        recipient: This parameter contains the email address of the person
                  who will receive the email.
        subject: This parameter contains the text that will appear in the
                subject line of the email.
        body: This parameter contains the text that will appear in the
             main body section of the email.

    Returns:
        This function returns a boolean value of True in the case that the
        email was sent successfully, or False in the case that it was not.
    """
    pass
```

#### Know Your Audience

Write for the intended reader's knowledge level.

```python
# Good: Appropriate for developers
def hash_password(password: str) -> str:
    """Hash a password using bcrypt.

    Uses bcrypt with a cost factor of 12. The salt is generated
    automatically and included in the returned hash.

    Args:
        password: The plaintext password to hash.

    Returns:
        The bcrypt hash string.
    """
    pass


# Bad: Too basic or too complex
def hash_password(password: str) -> str:
    """Hash a password.

    This function makes the password secure by scrambling it so that
    nobody can read it. It uses a special algorithm called bcrypt which
    applies the Blowfish cipher in the Eksblowfish key setup mode with
    adaptive cost parameterisation to resist brute-force attacks.

    Args:
        password: The password.

    Returns:
        The hash.
    """
    pass
```

#### Reference Established Style Guides

Follow industry-standard style guides for consistency:

- **Google Developer Documentation Style Guide**: Comprehensive technical writing guidelines
- **Microsoft Writing Style Guide**: Modern, accessible writing principles
- **The Economist Style Guide**: Clear, precise language (British English)

**Note**: Whilst Google and Microsoft style guides use American English, this project requires British English spelling. Follow their structural and technical writing guidance but adapt spelling to British English conventions.

```markdown
# Style guide references in README

## Documentation Standards

This project follows British English spelling and the Google Developer
Documentation Style Guide for technical writing. When in doubt:

1. Check the Google Developer Documentation Style Guide
2. Refer to The Economist Style Guide for British English conventions
3. Consult the Oxford English Dictionary for spelling

See `docs/STYLE_GUIDE.md` for project-specific conventions.
```

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
        """Initialise the repository with a database connection.

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
- [ ] British English spelling used throughout
- [ ] Sentences are clear and concise (15-20 words)
- [ ] Active voice used where possible
- [ ] Technical terms defined or explained

For every project:
- [ ] README follows Standard Readme spec
- [ ] Installation instructions are current
- [ ] Usage examples are tested
- [ ] Contributing guidelines present
- [ ] License specified
- [ ] British English spelling consistent throughout all documentation
- [ ] Documentation structure is clear with proper heading hierarchy
- [ ] Lists used for multiple related items
- [ ] Code examples include context and explanations
- [ ] Terminology is consistent across all documentation

## Related Standards

- See `python_style.md` for docstring formatting
- See `testing_standards.md` for test documentation
- See `git_workflow.md` for commit message documentation

---

**Last Updated**: 2025-10-30
**Status**: Strong preference - deviations require justification

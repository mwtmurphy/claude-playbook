# Streamlit Development Standards

**Version**: 2.0
**Last Updated**: 2025-10-18
**Applies To**: All Streamlit projects in this workspace

## Overview

This document defines workspace-level standards for Streamlit application development, with comprehensive coverage of multi-page app structure, navigation patterns, session state management, component reusability, performance optimization, and testing best practices using the AppTest framework.

## Table of Contents

1. [Project Structure](#project-structure)
2. [Multi-Page Navigation](#multi-page-navigation)
3. [Import Patterns](#import-patterns)
4. [Validation Requirements](#validation-requirements)
5. [Page Development Guidelines](#page-development-guidelines)
6. [Component Reusability](#component-reusability)
7. [Performance Optimization](#performance-optimization)
8. [Testing Practices](#testing-practices)
9. [Common Pitfalls](#common-pitfalls)
10. [Development Workflow](#development-workflow)

---

## Project Structure

### Recommended Directory Layout

```
project/
├── app/
│   ├── __init__.py           # Package marker
│   ├── streamlit_app.py      # Main entry point
│   ├── pages/                # Multi-page apps
│   │   ├── 1_📊_Page_One.py
│   │   └── 2_💰_Page_Two.py
│   └── utils/                # Shared utilities
│       ├── __init__.py
│       ├── data_loader.py    # Data loading functions
│       └── formatters.py     # Display formatters
├── scripts/
│   └── validate_streamlit_pages.py  # Validation script
├── tests/
│   ├── unit/
│   │   ├── test_app_data_loader.py
│   │   └── test_app_formatters.py
│   └── integration/
│       └── test_streamlit_import.py
├── pyproject.toml
└── README.md
```

### Key Principles

1. **Keep Streamlit code isolated**: Place all Streamlit-specific code in `app/` directory
2. **Separate concerns**: Use `utils/` for data processing, keep pages focused on display
3. **Name pages with prefixes**: Use numeric prefixes for ordering (e.g., `1_Page_One.py`)
4. **Include __init__.py**: Ensure all directories are proper Python packages

---

## Multi-Page Navigation

Streamlit provides two approaches for multi-page applications: the traditional `pages/` folder approach and the modern `st.navigation` API.

### Approach 1: Pages Folder (Traditional)

The simplest approach for basic multi-page apps.

**How it works:**
- Create a `pages/` directory next to your main app file
- Files in `pages/` automatically appear in the sidebar
- Naming convention: `{order}_{icon}_{name}.py` (e.g., `1_📊_Dashboard.py`)

**Example structure:**
```
app/
├── streamlit_app.py        # Main page
└── pages/
    ├── 1_📊_Dashboard.py
    ├── 2_📈_Analytics.py
    └── 3_⚙️_Settings.py
```

**Pros:**
- Simple, zero configuration
- Automatic page discovery
- Built-in sidebar navigation

**Cons:**
- Limited customization
- Fixed sidebar location
- No programmatic navigation

### Approach 2: st.navigation (Modern)

The modern API with full control over navigation.

**Why**: Provides programmatic navigation, custom navigation UI, conditional page visibility, and better organization.

**Basic example:**
```python
import streamlit as st

# Define pages
home_page = st.Page("home.py", title="Home", icon="🏠")
dashboard = st.Page("dashboard.py", title="Dashboard", icon="📊")
settings = st.Page("settings.py", title="Settings", icon="⚙️")

# Create navigation
pg = st.navigation([home_page, dashboard, settings])

# Run selected page
pg.run()
```

**With page groups:**
```python
import streamlit as st

# Define page groups
analysis_pages = [
    st.Page("pages/overview.py", title="Overview", icon="📊"),
    st.Page("pages/detailed.py", title="Detailed Analysis", icon="🔍"),
]

admin_pages = [
    st.Page("pages/users.py", title="Users", icon="👥"),
    st.Page("pages/settings.py", title="Settings", icon="⚙️"),
]

# Create navigation with groups
pg = st.navigation({
    "Analysis": analysis_pages,
    "Admin": admin_pages,
})

pg.run()
```

**Conditional navigation:**
```python
import streamlit as st

# Check authentication state
if "authenticated" not in st.session_state:
    st.session_state.authenticated = False

# Different navigation based on auth status
if st.session_state.authenticated:
    pages = [
        st.Page("pages/dashboard.py", title="Dashboard", icon="📊"),
        st.Page("pages/profile.py", title="Profile", icon="👤"),
        st.Page("pages/logout.py", title="Logout", icon="🚪"),
    ]
else:
    pages = [
        st.Page("pages/login.py", title="Login", icon="🔐"),
        st.Page("pages/signup.py", title="Sign Up", icon="✍️"),
    ]

pg = st.navigation(pages)
pg.run()
```

**Dynamic navigation:**
```python
import streamlit as st

def get_pages():
    """Return pages based on user role."""
    base_pages = [
        st.Page("pages/home.py", title="Home", icon="🏠"),
    ]

    # Add admin pages for admin users
    if st.session_state.get("user_role") == "admin":
        base_pages.extend([
            st.Page("pages/admin_dashboard.py", title="Admin", icon="🔒"),
            st.Page("pages/analytics.py", title="Analytics", icon="📈"),
        ])

    return base_pages

pg = st.navigation(get_pages())
pg.run()
```

### Choosing the Right Approach

**Use `pages/` folder when:**
- Building a simple app with static navigation
- Don't need programmatic page switching
- Want automatic page discovery
- Prefer convention over configuration

**Use `st.navigation` when:**
- Need conditional page visibility (authentication, roles)
- Want custom navigation UI
- Need programmatic navigation
- Want to organize pages in logical groups
- Building complex multi-page applications

### Programmatic Navigation

Navigate to pages programmatically using session state:

```python
# In main app with st.navigation
import streamlit as st

pages = {
    "home": st.Page("pages/home.py", title="Home"),
    "results": st.Page("pages/results.py", title="Results"),
}

# Check if navigation requested
if "navigate_to" in st.session_state:
    pg = st.navigation(
        [pages[st.session_state.navigate_to]],
        position="hidden"  # Hide navigation widget
    )
    del st.session_state.navigate_to
else:
    pg = st.navigation(list(pages.values()))

pg.run()

# In any page: trigger navigation
if st.button("View Results"):
    st.session_state.navigate_to = "results"
    st.rerun()
```

### Navigation Best Practices

1. **Consistent naming**: Use clear, descriptive page names
2. **Logical grouping**: Group related pages together
3. **Icon usage**: Use icons for visual navigation (but don't overdo it)
4. **State management**: Use session state to pass data between pages
5. **Loading states**: Show spinners when navigating to data-heavy pages

```python
# Good: Clear structure with groups
navigation = {
    "Data": [
        st.Page("pages/upload.py", title="Upload Data", icon="📤"),
        st.Page("pages/view.py", title="View Data", icon="👁️"),
    ],
    "Analysis": [
        st.Page("pages/explore.py", title="Explore", icon="🔍"),
        st.Page("pages/visualize.py", title="Visualize", icon="📊"),
    ],
    "Settings": [
        st.Page("pages/config.py", title="Configuration", icon="⚙️"),
    ],
}
```

---

## Import Patterns

### The Problem

Streamlit runs files as **scripts**, not as **modules**. This causes issues with relative imports:

```python
# ❌ WRONG - Fails with "attempted relative import with no known parent package"
from .utils.data_loader import load_all_data
from ..utils.formatters import apply_position_filter
```

### The Solution

Use **absolute imports** with **sys.path setup**:

#### For Main App (`app/streamlit_app.py`)

```python
"""App description."""

import sys
from pathlib import Path

# Add project root to path for imports
project_root = Path(__file__).parent.parent
if str(project_root) not in sys.path:
    sys.path.insert(0, str(project_root))

import streamlit as st

from app.utils.data_loader import load_all_data
from app.utils.formatters import apply_position_filter
```

**Path Calculation**: `parent.parent` (file → app → **root**)

#### For Pages (`app/pages/1_Page.py`)

```python
"""Page description."""

import sys
from pathlib import Path

# Add project root to path for imports
project_root = Path(__file__).parent.parent.parent
if str(project_root) not in sys.path:
    sys.path.insert(0, str(project_root))

import streamlit as st

from app.utils.data_loader import load_all_data
from app.utils.formatters import apply_position_filter
```

**Path Calculation**: `parent.parent.parent` (file → pages → app → **root**)

### Import Pattern Template

Every Streamlit file should follow this template:

```python
"""Module docstring explaining the purpose."""

import sys
from pathlib import Path

# Add project root to path for imports
# Adjust parent count based on file location:
# - Main app (app/file.py): parent.parent
# - Pages (app/pages/file.py): parent.parent.parent
project_root = Path(__file__).parent.parent  # Adjust as needed
if str(project_root) not in sys.path:
    sys.path.insert(0, str(project_root))

# Standard library imports
import asyncio
from datetime import datetime

# Third-party imports
import streamlit as st
import pandas as pd

# Local imports (use absolute imports from project root)
from app.utils.data_loader import load_all_data
from app.utils.formatters import apply_position_filter

# Page configuration (must be first Streamlit command)
st.set_page_config(
    page_title="Page Title",
    page_icon="🏆",
    layout="wide",
)

# Rest of the code...
```

### Rules for Imports

1. **Always use absolute imports** from project root (e.g., `from app.utils.data_loader`)
2. **Never use relative imports** (e.g., `from .utils`, `from ..utils`)
3. **Always include sys.path setup** at the top of each Streamlit file
4. **Calculate path correctly** based on file depth in directory structure
5. **Group imports** in order: stdlib, third-party, local

---

## Validation Requirements

### After Every Change Rule

**CRITICAL**: After making ANY changes to Streamlit files, you **MUST** run the validation script:

```bash
poetry run python scripts/validate_streamlit_pages.py
```

### What Validation Checks

1. ✅ **Compilation**: All Python files compile without syntax errors
2. ✅ **Imports**: All import statements work correctly
3. ✅ **Path Setup**: sys.path manipulation is correct
4. ✅ **Structure**: Pages follow Streamlit best practices
5. ✅ **No Relative Imports**: Ensures absolute imports are used

### Creating the Validation Script

Every Streamlit project should include `scripts/validate_streamlit_pages.py`:

```python
#!/usr/bin/env python3
"""Validate all Streamlit pages can be imported and run."""

import sys
from pathlib import Path

# Add project root to path
project_root = Path(__file__).parent.parent
sys.path.insert(0, str(project_root))

# Validation logic here...
# See full template in workspace examples
```

### Integration with Development

Add to `pyproject.toml`:

```toml
[tool.poetry.scripts]
validate-streamlit = "scripts.validate_streamlit_pages:main"
```

Then run with:

```bash
poetry run validate-streamlit
```

---

## Page Development Guidelines

### Page Configuration

Always call `st.set_page_config()` as the **first Streamlit command**:

```python
st.set_page_config(
    page_title="Page Title",
    page_icon="🏆",
    layout="wide",
    initial_sidebar_state="expanded",
)
```

### Page Structure

Follow this structure for consistency:

```python
# 1. Imports and path setup (see Import Patterns above)

# 2. Page configuration
st.set_page_config(...)

# 3. Title and description
st.title("📊 Page Title")
st.markdown("Page description...")

# 4. Data loading
with st.spinner("Loading data..."):
    data = load_data()

# 5. Filters (in sidebar if applicable)
with st.sidebar:
    filter1 = st.multiselect("Filter 1", options)
    filter2 = st.slider("Filter 2", 0, 100)

# 6. Main content
col1, col2 = st.columns(2)
with col1:
    st.metric("Metric 1", value1)
with col2:
    st.metric("Metric 2", value2)

# 7. Data display
st.dataframe(filtered_data)

# 8. Footer/additional info
st.divider()
st.caption("Data last updated: ...")
```

### Caching Best Practices

Use Streamlit's caching decorators appropriately:

```python
@st.cache_data(ttl=3600)  # Cache for 1 hour
def load_all_data():
    """Load data with caching."""
    # Expensive operations here
    return data

@st.cache_resource
def init_connection():
    """Cache connections/resources."""
    return connection
```

**When to use which:**
- `@st.cache_data`: Use for data transformations, API calls, dataframe operations
- `@st.cache_resource`: Use for database connections, ML models, global objects

**Cache invalidation:**
```python
@st.cache_data(ttl=3600)  # Auto-invalidate after 1 hour
def load_data():
    return fetch_expensive_data()

# Manual cache clearing
st.cache_data.clear()  # Clear all cached data
load_data.clear()      # Clear specific function's cache
```

### Session State Management

Session state persists data across reruns and pages within a user's session.

**Why**: Streamlit reruns the entire script on every interaction. Session state prevents data loss and enables complex multi-page workflows.

#### Initialization Patterns

Always initialize session state keys before use:

```python
# ✅ Good: Safe initialization pattern
if "user_data" not in st.session_state:
    st.session_state.user_data = None

if "counter" not in st.session_state:
    st.session_state.counter = 0

# ❌ Avoid: Direct access without initialization
st.session_state.counter += 1  # KeyError if counter doesn't exist
```

**Better: Initialization function**

```python
def initialize_session_state():
    """Initialize all session state variables."""
    defaults = {
        "authenticated": False,
        "username": None,
        "user_data": None,
        "filters": {},
        "page_history": [],
    }
    for key, default_value in defaults.items():
        if key not in st.session_state:
            st.session_state[key] = default_value

# Call at the top of your app/pages
initialize_session_state()
```

#### Sharing State Between Pages

Session state is **automatically shared** across all pages:

```python
# Page 1: Set values
st.session_state.selected_team = "Team A"
st.session_state.filters = {"position": "FWD"}

# Page 2: Access values (automatically available)
team = st.session_state.get("selected_team", "All Teams")
filters = st.session_state.get("filters", {})
```

#### Common State Patterns

**Pattern 1: Authentication State**

```python
def check_authentication():
    """Check if user is authenticated."""
    if "authenticated" not in st.session_state:
        st.session_state.authenticated = False

    if not st.session_state.authenticated:
        password = st.text_input("Password", type="password")
        if st.button("Login"):
            if password == "secret":
                st.session_state.authenticated = True
                st.rerun()
        st.stop()

# Use in any page
check_authentication()
```

**Pattern 2: Form Data Persistence**

```python
# Initialize form data
if "form_data" not in st.session_state:
    st.session_state.form_data = {
        "name": "",
        "email": "",
        "preferences": []
    }

# Use session state as default values
name = st.text_input(
    "Name",
    value=st.session_state.form_data["name"],
    key="name_input"
)

# Save on change
if name != st.session_state.form_data["name"]:
    st.session_state.form_data["name"] = name
```

**Pattern 3: Data Loading State**

```python
@st.cache_data
def load_expensive_data():
    """Load data once, cache it."""
    return fetch_data()

# Store loaded data in session state
if "data" not in st.session_state:
    with st.spinner("Loading data..."):
        st.session_state.data = load_expensive_data()

# Use the cached data
df = st.session_state.data
```

#### Callbacks and Session State

Use callbacks to update state without full reruns:

```python
def update_filter():
    """Callback to update filter state."""
    st.session_state.filter_applied = True

# Widget with callback
team = st.selectbox(
    "Select Team",
    options=teams,
    key="selected_team",
    on_change=update_filter
)

# Access the value via session state
if st.session_state.get("filter_applied", False):
    st.success(f"Filter applied: {st.session_state.selected_team}")
```

#### Avoiding Common Session State Pitfalls

**Pitfall 1: Widget key conflicts**

```python
# ❌ Wrong: Same key in different pages causes conflicts
# Page 1
st.text_input("Name", key="input")

# Page 2
st.number_input("Age", key="input")  # Conflict!

# ✅ Correct: Unique, descriptive keys
# Page 1
st.text_input("Name", key="user_name_input")

# Page 2
st.number_input("Age", key="user_age_input")
```

**Pitfall 2: Storing non-serializable objects**

```python
# ❌ Avoid: Database connections in session state
st.session_state.db_connection = create_connection()  # Problems on rerun

# ✅ Correct: Use st.cache_resource for connections
@st.cache_resource
def get_connection():
    return create_connection()

conn = get_connection()
```

**Pitfall 3: Unnecessary reruns**

```python
# ❌ Inefficient: Causes extra reruns
if st.button("Click me"):
    st.session_state.clicked = True
    st.rerun()  # Unnecessary

# ✅ Better: State updates automatically trigger rerun
if st.button("Click me"):
    st.session_state.clicked = True
    # No st.rerun() needed
```

#### Debugging Session State

View all session state variables:

```python
# Add to sidebar for debugging
with st.sidebar:
    with st.expander("🔍 Session State Debug"):
        st.write(st.session_state)
```

---

## Component Reusability

Create reusable components to avoid code duplication across pages.

**Why**: Reusable components reduce maintenance burden, ensure consistency, and make testing easier.

### Creating Reusable Components

Store reusable UI components in a dedicated module:

```
app/
├── utils/
│   ├── __init__.py
│   ├── data_loader.py
│   └── components.py    # Reusable UI components
└── pages/
    ├── 1_Dashboard.py
    └── 2_Analytics.py
```

### Component Patterns

**Pattern 1: Data Display Components**

```python
# app/utils/components.py
import streamlit as st
import pandas as pd

def display_metric_card(title: str, value: float, delta: float | None = None, icon: str = "📊"):
    """Display a metric in a styled card."""
    col1, col2 = st.columns([1, 4])
    with col1:
        st.markdown(f"<h1>{icon}</h1>", unsafe_allow_html=True)
    with col2:
        st.metric(label=title, value=value, delta=delta)

def display_dataframe_with_download(
    df: pd.DataFrame,
    title: str,
    filename: str = "data.csv"
):
    """Display dataframe with download button."""
    st.subheader(title)

    col1, col2 = st.columns([4, 1])
    with col1:
        st.dataframe(df, use_container_width=True)
    with col2:
        csv = df.to_csv(index=False)
        st.download_button(
            label="📥 Download",
            data=csv,
            file_name=filename,
            mime="text/csv"
        )
```

**Pattern 2: Filter Components**

```python
# app/utils/components.py
import streamlit as st
from typing import Any

def create_filter_sidebar(
    filters: dict[str, dict[str, Any]]
) -> dict[str, Any]:
    """
    Create a standardized filter sidebar.

    Args:
        filters: Dict mapping filter name to filter config
            {
                "team": {
                    "type": "multiselect",
                    "label": "Select Team",
                    "options": ["Team A", "Team B"]
                },
                "date_range": {
                    "type": "date_input",
                    "label": "Date Range"
                }
            }

    Returns:
        Dict of selected filter values
    """
    selected_filters = {}

    with st.sidebar:
        st.header("🔍 Filters")

        for key, config in filters.items():
            if config["type"] == "multiselect":
                selected_filters[key] = st.multiselect(
                    config["label"],
                    options=config["options"],
                    default=config.get("default", [])
                )
            elif config["type"] == "selectbox":
                selected_filters[key] = st.selectbox(
                    config["label"],
                    options=config["options"],
                    index=config.get("index", 0)
                )
            elif config["type"] == "slider":
                selected_filters[key] = st.slider(
                    config["label"],
                    min_value=config["min"],
                    max_value=config["max"],
                    value=config.get("default", config["min"])
                )

    return selected_filters
```

**Pattern 3: Authentication Components**

```python
# app/utils/components.py
import streamlit as st

def require_authentication(password: str = "secret"):
    """
    Require authentication before showing page content.

    Usage:
        if not require_authentication():
            return
        # Rest of page code
    """
    if "authenticated" not in st.session_state:
        st.session_state.authenticated = False

    if not st.session_state.authenticated:
        st.title("🔐 Authentication Required")

        with st.form("login_form"):
            entered_password = st.text_input("Password", type="password")
            submit = st.form_submit_button("Login")

            if submit:
                if entered_password == password:
                    st.session_state.authenticated = True
                    st.rerun()
                else:
                    st.error("Invalid password")

        st.stop()

    return True
```

**Pattern 4: Loading State Components**

```python
# app/utils/components.py
import streamlit as st
from typing import Callable, Any

def with_loading_state(
    func: Callable,
    message: str = "Loading...",
    *args,
    **kwargs
) -> Any:
    """Execute function with loading spinner."""
    with st.spinner(message):
        return func(*args, **kwargs)

def show_error_boundary(func: Callable, *args, **kwargs) -> Any:
    """Execute function with error handling."""
    try:
        return func(*args, **kwargs)
    except Exception as e:
        st.error(f"❌ Error: {str(e)}")
        with st.expander("View Details"):
            st.exception(e)
        return None
```

### Using Components in Pages

```python
# app/pages/1_Dashboard.py
import sys
from pathlib import Path

project_root = Path(__file__).parent.parent.parent
if str(project_root) not in sys.path:
    sys.path.insert(0, str(project_root))

import streamlit as st
from app.utils.components import (
    require_authentication,
    display_metric_card,
    create_filter_sidebar,
    with_loading_state
)
from app.utils.data_loader import load_data

st.set_page_config(page_title="Dashboard", page_icon="📊", layout="wide")

# Require authentication
if not require_authentication():
    st.stop()

# Create filters
filters = create_filter_sidebar({
    "team": {
        "type": "multiselect",
        "label": "Select Teams",
        "options": ["Team A", "Team B", "Team C"]
    }
})

# Load data with loading state
data = with_loading_state(load_data, "Loading dashboard data...")

# Display metrics
col1, col2, col3 = st.columns(3)
with col1:
    display_metric_card("Total Users", 1234, delta=56, icon="👥")
with col2:
    display_metric_card("Revenue", 98765, delta=-123, icon="💰")
with col3:
    display_metric_card("Conversion", 4.5, delta=0.5, icon="📈")
```

### Component Best Practices

1. **Single Responsibility**: Each component should do one thing well
2. **Type Hints**: Use type hints for better IDE support and documentation
3. **Docstrings**: Document parameters and return values
4. **Sensible Defaults**: Provide default values where appropriate
5. **Error Handling**: Handle errors gracefully within components
6. **State Management**: Be explicit about session state usage

```python
# ✅ Good: Well-structured component
def create_data_table(
    df: pd.DataFrame,
    title: str,
    show_download: bool = True,
    show_search: bool = True,
    key_prefix: str = ""
) -> None:
    """
    Display a data table with optional features.

    Args:
        df: DataFrame to display
        title: Table title
        show_download: Show download button (default: True)
        show_search: Show search box (default: True)
        key_prefix: Prefix for widget keys to avoid conflicts
    """
    st.subheader(title)

    # Search functionality
    if show_search:
        search = st.text_input(
            "Search",
            key=f"{key_prefix}_search",
            placeholder="Type to search..."
        )
        if search:
            df = df[df.astype(str).apply(
                lambda x: x.str.contains(search, case=False).any(),
                axis=1
            )]

    # Display table
    st.dataframe(df, use_container_width=True)

    # Download button
    if show_download:
        csv = df.to_csv(index=False)
        st.download_button(
            "Download CSV",
            data=csv,
            file_name=f"{title.lower().replace(' ', '_')}.csv",
            mime="text/csv",
            key=f"{key_prefix}_download"
        )

# ❌ Avoid: Doing too much in one component
def do_everything(df, config, user, settings, ...):
    # Component that tries to do everything - hard to test and maintain
    pass
```

### Testing Components

Components should be testable independently:

```python
# tests/unit/test_components.py
import pandas as pd
from app.utils.components import display_metric_card

def test_display_metric_card(mocker):
    """Test metric card component."""
    # Mock streamlit functions
    mock_columns = mocker.patch("streamlit.columns")
    mock_metric = mocker.patch("streamlit.metric")

    # Test component
    display_metric_card("Test Metric", 100, delta=5)

    # Assert streamlit functions called correctly
    mock_columns.assert_called_once()
    mock_metric.assert_called_once_with(
        label="Test Metric",
        value=100,
        delta=5
    )
```

---

## Performance Optimization

Optimize Streamlit apps for faster load times and smoother user experience.

**Why**: Streamlit reruns the entire script on every interaction. Proper optimization prevents unnecessary computations and improves responsiveness.

### Widget Keys

Use unique keys to prevent unnecessary reruns:

```python
# ✅ Good: Unique keys prevent state conflicts
team = st.selectbox(
    "Select Team",
    options=teams,
    key="dashboard_team_selector"
)

position = st.multiselect(
    "Select Position",
    options=positions,
    key="dashboard_position_filter"
)

# ❌ Avoid: No keys or duplicate keys
team = st.selectbox("Select Team", teams)  # State may conflict
position = st.multiselect("Position", positions, key="filter")  # Vague key
```

### Avoid Unnecessary Reruns

Structure code to minimize reruns:

```python
# ✅ Good: Use callbacks to update state without full rerun
def on_filter_change():
    """Update filter state."""
    st.session_state.filter_changed = True

team = st.selectbox(
    "Team",
    options=teams,
    key="selected_team",
    on_change=on_filter_change
)

# Only reprocess if filter changed
if st.session_state.get("filter_changed", False):
    filtered_data = apply_filters(data, st.session_state.selected_team)
    st.session_state.filtered_data = filtered_data
    st.session_state.filter_changed = False

# ❌ Avoid: Reprocessing on every rerun
filtered_data = apply_filters(data, team)  # Runs every time
```

### Lazy Loading

Load data only when needed:

```python
# ✅ Good: Lazy load expensive data
if "detailed_data" not in st.session_state:
    st.session_state.detailed_data = None

if st.button("Load Detailed Data"):
    with st.spinner("Loading..."):
        st.session_state.detailed_data = load_expensive_data()

if st.session_state.detailed_data is not None:
    st.dataframe(st.session_state.detailed_data)

# ❌ Avoid: Loading everything upfront
data1 = load_data_1()  # Loaded even if not used
data2 = load_data_2()  # Loaded even if not used
data3 = load_data_3()  # Loaded even if not used
```

### Fragment Optimization

Use `@st.fragment` for partial reruns (Streamlit 1.33+):

```python
import streamlit as st

@st.fragment
def filter_section():
    """Fragment that reruns independently."""
    team = st.selectbox("Team", teams)
    position = st.multiselect("Position", positions)
    return team, position

# Fragment reruns don't trigger full app rerun
team, position = filter_section()

# This only runs when app reruns, not when fragment updates
expensive_data = load_expensive_data()
```

### Minimize Widget Count

Reduce the number of widgets on a page:

```python
# ✅ Good: Use containers to organize and conditionally show widgets
view_mode = st.radio("View", ["Simple", "Advanced"])

if view_mode == "Advanced":
    with st.expander("Advanced Filters"):
        filter1 = st.text_input("Filter 1")
        filter2 = st.slider("Filter 2", 0, 100)
        # More filters...

# ❌ Avoid: Always showing all widgets
filter1 = st.text_input("Filter 1")
filter2 = st.slider("Filter 2", 0, 100)
filter3 = st.date_input("Filter 3")
# 20 more filters that users rarely need...
```

### Performance Monitoring

Add timing to identify bottlenecks:

```python
import time
import streamlit as st

def time_operation(name: str):
    """Context manager for timing operations."""
    class Timer:
        def __enter__(self):
            self.start = time.time()
            return self

        def __exit__(self, *args):
            elapsed = time.time() - self.start
            st.caption(f"{name}: {elapsed:.2f}s")

    return Timer()

# Usage
with time_operation("Data Loading"):
    data = load_data()

with time_operation("Data Processing"):
    processed = process_data(data)
```

---

## Testing Practices

Comprehensive testing ensures Streamlit apps work correctly and catch regressions early.

**Why**: Streamlit's reactive model makes manual testing time-consuming. Automated tests catch bugs early and enable confident refactoring.

### Testing Strategy

Use a layered testing approach:

1. **Unit Tests**: Test utility functions and business logic independently
2. **Component Tests**: Test reusable UI components with mocks
3. **App Tests**: Test Streamlit pages with AppTest framework
4. **Integration Tests**: Test full workflows across multiple pages
5. **Manual Tests**: Final verification before deployment

### Unit Tests for Business Logic

Test utility functions independently (no Streamlit):

```python
# tests/unit/test_data_loader.py
from unittest.mock import patch, MagicMock
import pytest
import pandas as pd
from app.utils.data_loader import load_all_data, filter_by_team

def test_filter_by_team():
    """Test team filtering logic."""
    # Arrange
    df = pd.DataFrame({
        "player": ["Alice", "Bob", "Charlie"],
        "team": ["Team A", "Team B", "Team A"]
    })

    # Act
    result = filter_by_team(df, ["Team A"])

    # Assert
    assert len(result) == 2
    assert all(result["team"] == "Team A")

@patch("app.utils.data_loader.fetch_api_data")
def test_load_all_data(mock_fetch):
    """Test data loading with mocked API."""
    # Arrange
    mock_fetch.return_value = [
        {"player": "Alice", "team": "Team A"},
        {"player": "Bob", "team": "Team B"}
    ]

    # Act
    df = load_all_data()

    # Assert
    assert len(df) == 2
    assert "player" in df.columns
    mock_fetch.assert_called_once()
```

### AppTest Framework (Streamlit's Official Testing Tool)

AppTest simulates a Streamlit app and allows interaction testing.

**Installation:**
```bash
poetry add --group dev streamlit[testing]
```

**Basic AppTest example:**

```python
# tests/app/test_main_page.py
from streamlit.testing.v1 import AppTest

def test_main_page_renders():
    """Test that main page renders without errors."""
    # Initialize the app
    at = AppTest.from_file("app/streamlit_app.py")

    # Run the app
    at.run()

    # Assert no exceptions
    assert not at.exception

def test_page_title():
    """Test that page displays correct title."""
    at = AppTest.from_file("app/streamlit_app.py")
    at.run()

    # Check title exists
    assert len(at.title) > 0
    assert "Dashboard" in at.title[0].value

def test_user_interaction():
    """Test user interactions with widgets."""
    at = AppTest.from_file("app/streamlit_app.py")
    at.run()

    # Interact with a selectbox
    at.selectbox[0].select("Team A")
    at.run()

    # Verify the selection was applied
    assert at.selectbox[0].value == "Team A"

    # Check that dataframe updated
    assert len(at.dataframe) > 0
```

### Testing Multi-Page Apps

Test navigation between pages:

```python
# tests/app/test_multipage.py
from streamlit.testing.v1 import AppTest

def test_page_navigation():
    """Test navigation between pages."""
    at = AppTest.from_file("app/streamlit_app.py")
    at.run()

    # Verify initial page loads
    assert not at.exception

    # Navigate to another page (if using st.navigation)
    if len(at.button) > 0:
        at.button("Go to Analytics").click()
        at.run()

        # Verify new page loaded
        assert "Analytics" in at.title[0].value

def test_state_persists_across_pages():
    """Test that session state persists across pages."""
    at = AppTest.from_file("app/streamlit_app.py")
    at.run()

    # Set a value in session state
    at.session_state["user_selection"] = "Team A"

    # Navigate to another page
    # ... navigation logic ...

    # Verify state persisted
    assert at.session_state["user_selection"] == "Team A"
```

### Testing Session State

Test session state initialization and updates:

```python
# tests/app/test_session_state.py
from streamlit.testing.v1 import AppTest

def test_session_state_initialization():
    """Test that session state is properly initialized."""
    at = AppTest.from_file("app/streamlit_app.py")
    at.run()

    # Check default values
    assert "authenticated" in at.session_state
    assert at.session_state["authenticated"] == False

def test_session_state_update():
    """Test session state updates on interaction."""
    at = AppTest.from_file("app/pages/1_Dashboard.py")
    at.run()

    # Interact with widget
    at.text_input[0].input("test_user")
    at.run()

    # Verify session state updated
    assert "username" in at.session_state
    assert at.session_state["username"] == "test_user"

def test_session_state_callback():
    """Test callbacks update session state."""
    at = AppTest.from_file("app/streamlit_app.py")
    at.run()

    # Trigger callback via widget interaction
    at.button[0].click()
    at.run()

    # Verify callback executed
    assert at.session_state.get("button_clicked", False) == True
```

### Mocking Streamlit Components

Mock Streamlit components for faster unit tests:

```python
# tests/unit/test_components.py
import pytest
from unittest.mock import Mock, patch, MagicMock

@patch("streamlit.columns")
@patch("streamlit.metric")
def test_display_metric_card(mock_metric, mock_columns):
    """Test metric card component with mocks."""
    from app.utils.components import display_metric_card

    # Setup mock
    mock_col1 = MagicMock()
    mock_col2 = MagicMock()
    mock_columns.return_value = [mock_col1, mock_col2]

    # Call component
    display_metric_card("Revenue", 1000, delta=50)

    # Verify calls
    mock_columns.assert_called_once_with([1, 4])
    mock_metric.assert_called_once_with(
        label="Revenue",
        value=1000,
        delta=50
    )

@patch("streamlit.session_state", {})
def test_component_with_session_state():
    """Test component that uses session state."""
    import streamlit as st
    from app.utils.components import require_authentication

    # Mock session state
    st.session_state["authenticated"] = True

    # Test component
    result = require_authentication()

    # Verify behavior
    assert result == True
```

### Pytest Fixtures for Streamlit

Create reusable test fixtures:

```python
# tests/conftest.py
import pytest
import pandas as pd
from streamlit.testing.v1 import AppTest

@pytest.fixture
def sample_dataframe():
    """Provide sample dataframe for tests."""
    return pd.DataFrame({
        "player": ["Alice", "Bob", "Charlie"],
        "team": ["Team A", "Team B", "Team A"],
        "points": [100, 150, 120]
    })

@pytest.fixture
def app_test():
    """Provide AppTest instance."""
    at = AppTest.from_file("app/streamlit_app.py")
    at.run()
    return at

@pytest.fixture
def authenticated_app():
    """Provide authenticated AppTest instance."""
    at = AppTest.from_file("app/streamlit_app.py")
    at.session_state["authenticated"] = True
    at.run()
    return at

@pytest.fixture(autouse=True)
def reset_session_state():
    """Reset session state between tests."""
    import streamlit as st
    if hasattr(st, 'session_state'):
        st.session_state.clear()
    yield
```

**Using fixtures:**

```python
# tests/app/test_with_fixtures.py
def test_with_sample_data(sample_dataframe):
    """Test using sample data fixture."""
    assert len(sample_dataframe) == 3
    assert "player" in sample_dataframe.columns

def test_with_app_test(app_test):
    """Test using app_test fixture."""
    assert not app_test.exception
    assert len(app_test.title) > 0

def test_authenticated_flow(authenticated_app):
    """Test flow with authenticated user."""
    assert authenticated_app.session_state["authenticated"] == True
    # Test authenticated-only features
```

### Integration Tests

Test full workflows across imports and configuration:

```python
# tests/integration/test_streamlit_import.py
def test_import_main_app():
    """Test main app can be imported."""
    from app import streamlit_app
    assert hasattr(streamlit_app, '__file__')

def test_import_all_pages():
    """Test all pages can be imported."""
    from pathlib import Path

    pages_dir = Path("app/pages")
    for page_file in pages_dir.glob("*.py"):
        # Import each page
        page_name = page_file.stem
        exec(f"from app.pages import {page_name}")

def test_app_uses_absolute_imports():
    """Test app uses absolute imports."""
    with open('app/streamlit_app.py') as f:
        content = f.read()
    assert 'from app.utils' in content
    assert 'from .utils' not in content

def test_all_pages_have_page_config():
    """Test all pages configure page settings."""
    from pathlib import Path

    pages_dir = Path("app/pages")
    for page_file in pages_dir.glob("*.py"):
        with open(page_file) as f:
            content = f.read()
        assert "st.set_page_config" in content
```

### Testing Cached Functions

Test functions with caching decorators:

```python
# tests/unit/test_caching.py
import pytest
from unittest.mock import patch

def test_cached_function_called_once():
    """Test that cached function is only called once."""
    with patch("app.utils.data_loader.expensive_api_call") as mock_api:
        mock_api.return_value = {"data": "value"}

        from app.utils.data_loader import load_cached_data

        # First call - should hit API
        result1 = load_cached_data()

        # Second call - should use cache
        result2 = load_cached_data()

        # Verify API only called once
        assert mock_api.call_count == 1
        assert result1 == result2

def test_cache_invalidation():
    """Test cache can be cleared."""
    from app.utils.data_loader import load_cached_data

    # Load data
    result1 = load_cached_data()

    # Clear cache
    load_cached_data.clear()

    # Load again (should recompute)
    result2 = load_cached_data()

    # Both results should be valid
    assert result1 is not None
    assert result2 is not None
```

### Manual Testing Checklist

Before committing, manually verify:

1. **Start the app**: `poetry run streamlit run app/streamlit_app.py`
2. **Test all pages**: Click through each page in navigation
3. **Test interactions**:
   - All filters work
   - Buttons trigger correct actions
   - Forms submit successfully
4. **Test edge cases**:
   - Empty data scenarios
   - Invalid inputs
   - Network errors (if applicable)
5. **Check console**: No errors or warnings
6. **Test responsiveness**: Verify layout on different screen sizes
7. **Test performance**: Pages load within acceptable time

### Test Organization

Organize tests by type:

```
tests/
├── conftest.py              # Shared fixtures
├── unit/                    # Unit tests (no Streamlit)
│   ├── test_data_loader.py
│   ├── test_formatters.py
│   └── test_validators.py
├── components/              # Component tests (mocked Streamlit)
│   ├── test_components.py
│   └── test_filters.py
├── app/                     # AppTest tests (simulated app)
│   ├── test_main_page.py
│   ├── test_page_1.py
│   └── test_multipage.py
└── integration/             # Integration tests
    ├── test_imports.py
    └── test_end_to_end.py
```

### Running Tests

```bash
# Run all tests
poetry run pytest

# Run specific test type
poetry run pytest tests/unit/
poetry run pytest tests/app/
poetry run pytest tests/integration/

# Run with coverage
poetry run pytest --cov=app --cov-report=html

# Run with verbose output
poetry run pytest -v

# Run specific test file
poetry run pytest tests/app/test_main_page.py

# Run specific test
poetry run pytest tests/app/test_main_page.py::test_page_title
```

### Test Configuration

Add to `pyproject.toml`:

```toml
[tool.pytest.ini_options]
testpaths = ["tests"]
python_files = ["test_*.py"]
python_classes = ["Test*"]
python_functions = ["test_*"]
addopts = [
    "-v",
    "--strict-markers",
    "--cov=app",
    "--cov-report=term-missing",
    "--cov-report=html",
]
markers = [
    "unit: Unit tests",
    "integration: Integration tests",
    "app: AppTest tests",
    "slow: Slow running tests",
]
```

**Run by marker:**
```bash
poetry run pytest -m unit       # Only unit tests
poetry run pytest -m app        # Only AppTest tests
poetry run pytest -m "not slow" # Skip slow tests
```

---

## Common Pitfalls

### 1. Using Relative Imports

❌ **Problem**:
```python
from .utils.data_loader import load_all_data
```

**Error**: `ImportError: attempted relative import with no known parent package`

✅ **Solution**:
```python
import sys
from pathlib import Path
project_root = Path(__file__).parent.parent
sys.path.insert(0, str(project_root))

from app.utils.data_loader import load_all_data
```

### 2. Incorrect Path Calculation

❌ **Problem**:
```python
# In app/pages/file.py (wrong depth)
project_root = Path(__file__).parent.parent  # Only goes to app/, not root
```

✅ **Solution**:
```python
# In app/pages/file.py (correct depth)
project_root = Path(__file__).parent.parent.parent  # Goes to root
```

### 3. Missing sys.path Setup

❌ **Problem**:
```python
import streamlit as st
from app.utils.data_loader import load_all_data  # Fails - app not in path
```

✅ **Solution**:
```python
import sys
from pathlib import Path
project_root = Path(__file__).parent.parent
sys.path.insert(0, str(project_root))

import streamlit as st
from app.utils.data_loader import load_all_data  # Works
```

### 4. st.set_page_config() Not First

❌ **Problem**:
```python
import streamlit as st
st.title("My App")
st.set_page_config(...)  # Error: must be first command
```

✅ **Solution**:
```python
import streamlit as st
st.set_page_config(...)  # First!
st.title("My App")
```

### 5. Async Functions in Streamlit

❌ **Problem**:
```python
@st.cache_data
async def load_data():  # Streamlit cache doesn't support async
    return await fetch_data()
```

✅ **Solution**:
```python
@st.cache_data
def load_data():
    """Sync wrapper for async function."""
    return asyncio.run(fetch_data())
```

---

## Development Workflow

### Initial Setup

1. Create project structure:
   ```bash
   mkdir -p app/pages app/utils scripts tests/unit tests/integration
   touch app/__init__.py app/utils/__init__.py
   ```

2. Create validation script: `scripts/validate_streamlit_pages.py`

3. Add validation to pyproject.toml

### Making Changes

1. **Edit** Streamlit file(s)
2. **Validate** immediately:
   ```bash
   poetry run python scripts/validate_streamlit_pages.py
   ```
3. **Fix** any errors reported
4. **Test manually**:
   ```bash
   poetry run streamlit run app/streamlit_app.py
   ```
5. **Commit** only after validation passes

### Pre-Commit Checklist

- [ ] All files pass validation script
- [ ] Manual testing completed (all pages load)
- [ ] Unit tests pass
- [ ] Integration tests pass
- [ ] No console errors/warnings
- [ ] Code formatted (black)
- [ ] Code linted (ruff)

### Continuous Integration

Add to CI pipeline:

```yaml
# .github/workflows/test.yml
- name: Validate Streamlit pages
  run: poetry run python scripts/validate_streamlit_pages.py
```

---

## Quick Reference

### Command Cheat Sheet

```bash
# Validate all pages
poetry run python scripts/validate_streamlit_pages.py

# Run app
poetry run streamlit run app/streamlit_app.py

# Run tests
poetry run pytest tests/unit/test_app*.py
poetry run pytest tests/integration/

# Format code
poetry run black app/

# Lint code
poetry run ruff check app/
```

### Import Template Copy-Paste

**For main app** (`app/streamlit_app.py`):
```python
import sys
from pathlib import Path
project_root = Path(__file__).parent.parent
if str(project_root) not in sys.path:
    sys.path.insert(0, str(project_root))
```

**For pages** (`app/pages/*.py`):
```python
import sys
from pathlib import Path
project_root = Path(__file__).parent.parent.parent
if str(project_root) not in sys.path:
    sys.path.insert(0, str(project_root))
```

---

## Version History

- **2.0** (2025-10-18): Major expansion - Multi-page development and testing best practices
  - Added Multi-Page Navigation section (st.navigation API, programmatic navigation)
  - Added Session State Management section (initialization patterns, sharing state, common patterns)
  - Added Component Reusability section (reusable UI components, patterns, testing)
  - Added Performance Optimization section (widget keys, lazy loading, fragments)
  - Expanded Testing Practices with AppTest framework (comprehensive testing patterns)
  - Expanded Caching Best Practices (cache invalidation, data vs resource)
  - Updated Table of Contents

- **1.0** (2025-10-17): Initial standards document
  - Import patterns for Streamlit scripts
  - Validation requirements
  - Common pitfalls and solutions
  - Development workflow

---

## Related Standards

- [Python Style Guide](./python_style.md)
- [Testing Standards](./testing_standards.md)
- [Architecture Patterns](./architecture_patterns.md)
- [Documentation Standards](./documentation_standards.md)

# Playbook Improvement Checklist

**Created**: 2025-10-31
**Status**: In Progress
**Branch**: feat/migrate-reference-files

## Completed ✅

### Phase 1: Quick Wins (Partial)
- [x] claude/README.md - Updated with all new standards, date to 2025-10-31
- [x] Root README.md - Already complete from migration
- [x] project_setup_prompt.md - Updated URL and date to 2025-10-31
- [x] python_style.md - Standardized metadata, updated date, enhanced Related Standards

### Commits Made
1. `49d72a6` - feat: migrate reference files from workspace .claude directory
2. `ecb47b7` - docs: update claude/README.md with new standards files
3. `954decc` - docs: update project_setup_prompt.md with current repo URL
4. `[pending]` - docs: standardize python_style.md metadata

## Remaining Work 🔄

### Phase 1: Metadata Standardization (High Priority)

**Format to apply to all files:**
```markdown
# [Title]

**Status**: Strong preference - deviations require justification and approval
**Scope**: [One-line description]
**Last Updated**: 2025-10-31

---

## Overview
```

**Files needing updates:**
- [ ] sql_style.md - Update date to 2025-10-31, add Last Updated to header
- [ ] testing_standards.md - Update date to 2025-10-31
- [ ] git_workflow.md - Update date to 2025-10-31
- [ ] architecture_patterns.md - Update date to 2025-10-31
- [ ] error_handling.md - Update date to 2025-10-31
- [ ] database_standards.md - Update date to 2025-10-31
- [ ] performance_considerations.md - Update date to 2025-10-31
- [ ] environment_setup.md - Update date to 2025-10-31
- [ ] documentation_standards.md - Update date to 2025-10-31 (currently 2025-10-11)
- [ ] claude_workflow.md - Update date to 2025-10-31
- [ ] streamlit_standards.md - Already has 2025-10-18, consider updating
- [ ] data_visualization_standards.md - Already at 2025-10-31 ✓
- [ ] documentation_for_claude.md - Already at 2025-10-21, update to 2025-10-31
- [ ] documentation_for_employees.md - Already at 2025-10-21, update to 2025-10-31
- [ ] interactive_visualization_testing.md - Already at 2025-10-21, update to 2025-10-31
- [ ] user_journey_diagrams.md - Check and update if needed
- [ ] reference_guide.md - Check and update if needed

### Phase 2: Cross-References (Medium Priority)

**Add "Related Standards" sections where missing:**

**python_style.md** ✅ (Done)
- documentation_standards.md
- git_workflow.md
- sql_style.md
- testing_standards.md

**sql_style.md**
- database_standards.md
- python_style.md
- documentation_standards.md

**testing_standards.md** ✅ (Already has some)
- python_style.md
- documentation_standards.md
- streamlit_standards.md (for Streamlit testing)
- interactive_visualization_testing.md

**architecture_patterns.md**
- python_style.md
- error_handling.md
- performance_considerations.md
- testing_standards.md

**streamlit_standards.md**
- python_style.md
- testing_standards.md
- data_visualization_standards.md
- architecture_patterns.md
- interactive_visualization_testing.md

**data_visualization_standards.md**
- streamlit_standards.md
- documentation_for_employees.md
- interactive_visualization_testing.md

**documentation_standards.md**
- documentation_for_claude.md
- documentation_for_employees.md
- python_style.md
- reference_guide.md

**documentation_for_claude.md**
- documentation_standards.md
- documentation_for_employees.md
- reference_guide.md

**documentation_for_employees.md**
- documentation_standards.md
- documentation_for_claude.md
- data_visualization_standards.md

**git_workflow.md**
- python_style.md
- documentation_standards.md

**error_handling.md**
- python_style.md
- architecture_patterns.md
- [future] logging_standards.md

**database_standards.md**
- sql_style.md
- architecture_patterns.md
- performance_considerations.md

**performance_considerations.md**
- architecture_patterns.md
- database_standards.md
- python_style.md

**environment_setup.md**
- python_style.md
- testing_standards.md

**claude_workflow.md**
- git_workflow.md
- documentation_standards.md

**interactive_visualization_testing.md**
- streamlit_standards.md
- data_visualization_standards.md
- testing_standards.md

**user_journey_diagrams.md**
- documentation_for_claude.md
- streamlit_standards.md

**reference_guide.md**
- documentation_for_claude.md
- documentation_for_employees.md
- documentation_standards.md

### Phase 3: New Content Files (High Value)

#### security_standards.md
**Scope**: Authentication, authorization, secrets management, input validation
**Sections**:
- Authentication patterns
- Authorization and permissions
- Secrets management
- Input validation and sanitization
- Security headers
- HTTPS/TLS requirements
- Vulnerability scanning
- Related Standards: python_style.md, environment_setup.md

#### logging_standards.md
**Scope**: Logging levels, formats, structured logging
**Sections**:
- Log levels (DEBUG, INFO, WARNING, ERROR, CRITICAL)
- Structured logging with JSON
- What to log and what not to log
- Log formatting standards
- Performance considerations
- Related Standards: error_handling.md, python_style.md, performance_considerations.md

#### api_design_standards.md
**Scope**: REST API design, versioning, error responses
**Sections**:
- REST principles
- URL structure and naming
- HTTP methods and status codes
- Request/response formats (JSON)
- API versioning strategies
- Error response format
- Pagination patterns
- Authentication/authorization
- Rate limiting
- Related Standards: python_style.md, error_handling.md, documentation_standards.md

### Phase 4: Templates (Medium Priority)

**Add to claude/templates/:**

#### streamlit_page_template.py
```python
"""[Page description]."""

import sys
from pathlib import Path

# Add project root to path for imports
project_root = Path(__file__).parent.parent.parent
if str(project_root) not in sys.path:
    sys.path.insert(0, str(project_root))

import streamlit as st

# Page configuration (must be first Streamlit command)
st.set_page_config(
    page_title="Page Title",
    page_icon="📊",
    layout="wide",
)

# Title and description
st.title("📊 Page Title")
st.markdown("Page description...")

# Main content
# TODO: Add page content
```

#### pytest_test_template.py
```python
"""Tests for [module_name] module."""

import pytest


def test_example():
    """Test example function."""
    # Arrange
    expected = True

    # Act
    result = True

    # Assert
    assert result == expected
```

#### README_template.md
```markdown
# Project Name

Brief project description.

## Installation

\`\`\`bash
# Installation instructions
\`\`\`

## Usage

\`\`\`python
# Usage examples
\`\`\`

## Development

See [CONTRIBUTING.md](CONTRIBUTING.md) for development guidelines.

## Standards

This project follows [mwtmurphy/claude-playbook](https://github.com/mwtmurphy/claude-playbook) standards.

## License

[License information]
```

#### github_issue_template.md
```markdown
---
name: Standards Suggestion
about: Suggest an improvement to the playbook standards
title: '[STANDARD] '
labels: enhancement
---

## Standard File
Which standard file does this relate to?

## Current Behavior
What does the current standard say (or not say)?

## Proposed Change
What should be changed or added?

## Rationale
Why is this change beneficial?

## Impact
What projects or patterns would be affected?
```

### Phase 5: Documentation (Low Priority)

#### CHANGELOG.md
**Structure**:
```markdown
# Changelog

All notable changes to the claude-playbook standards will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/).

## [Unreleased]

### Added
- security_standards.md
- logging_standards.md
- api_design_standards.md

### Changed
- Standardized metadata across all files
- Updated all Last Updated dates to 2025-10-31

## [1.0.0] - 2025-10-31

### Added
- Initial migration of 7 standards files
- data_visualization_standards.md
- documentation_for_claude.md
- documentation_for_employees.md
- interactive_visualization_testing.md
- user_journey_diagrams.md
- templates/d3_visualization_template.html

### Changed
- Repository renamed to claude-playbook
- Added British English guidance to documentation_standards.md
```

## Notes

- Keep flat directory structure (no subdirectories) to avoid breaking URLs
- All GitHub URLs must remain stable
- Use cross-references and READMEs for organization
- Commit incrementally to track progress
- Test all changes before pushing

## Quick Commands

```bash
# Update a file's last updated date
sed -i '' 's/\*\*Last Updated\*\*: 2025-10-[0-9][0-9]/**Last Updated**: 2025-10-31/' claude/[file].md

# Check all last updated dates
grep "Last Updated" claude/*.md

# Stage all changes
git add claude/

# Commit
git commit -m "docs: standardize metadata across playbook"
```

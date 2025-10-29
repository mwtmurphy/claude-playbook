# Environment Setup Standards

**Status**: Strong preference - deviations require justification and approval
**Scope**: Development environment configuration for Python projects

## Overview

Consistent environment setup ensures all developers work with the same tools and dependencies, reducing "works on my machine" problems.

**Why**: Reproducible environments prevent bugs, simplify onboarding, and streamline collaboration.

## Python Version Management: pyenv

### Why pyenv

- Manages multiple Python versions per project
- Doesn't require system Python modifications
- Simple, explicit version specification
- Works seamlessly with poetry

### Python Version Selection Strategy

**Important**: Python does not have traditional "LTS" (Long-Term Support) releases. Instead, every Python release receives **5 years of support** (18 months of full support with bug fixes, followed by 3+ years of security-only updates).

**Guidelines for choosing Python version**:

1. **Use the latest stable version with mature ecosystem support** at project start
2. **Minimum maturity**: Choose a Python version released at least **6 months ago**
   - Gives time for major packages to add support
   - Allows early bugs to be identified and fixed
   - Ensures tooling compatibility (Black, Ruff, mypy, etc.)

3. **Package compatibility**: All dependencies should use their **latest versions** compatible with chosen Python
   - Avoid pinning to old package versions unless absolutely necessary
   - Start projects with modern, well-supported dependencies
   - Check package compatibility before committing to Python version

**Example decision process** (as of 2025):
```
Python 3.13: Released Oct 2024 → Use after Apr 2025
Python 3.12: Released Oct 2023 → ✓ Mature, excellent choice
Python 3.11: Released Oct 2022 → ✓ Very mature, stable choice
Python 3.10: Released Oct 2021 → Still supported, but prefer newer
```

**Why this matters**:
- Starting with latest supported packages reduces technical debt
- Modern Python versions have performance improvements
- Newer type hint features improve code quality
- Longer support runway before needing to upgrade

### Installation

```bash
# macOS with Homebrew
brew install pyenv

# Add to shell configuration (~/.zshrc or ~/.bashrc)
export PYENV_ROOT="$HOME/.pyenv"
export PATH="$PYENV_ROOT/bin:$PATH"
eval "$(pyenv init --path)"
eval "$(pyenv init -)"
```

### Usage

```bash
# List available Python versions
pyenv install --list

# Install specific Python version (use latest mature version)
pyenv install 3.12.0

# Set Python version for project
cd /path/to/project
pyenv local 3.12.0  # Creates .python-version file

# Verify version
python --version  # Should show 3.12.0
```

## Dependency Management: poetry

### Why poetry

- Modern dependency management and packaging
- Deterministic builds with lock files
- Virtual environment management
- Separates dev and production dependencies
- Publishes to PyPI easily

### Installation

```bash
# Install poetry
curl -sSL https://install.python-poetry.org | python3 -

# Add to PATH (add to ~/.zshrc or ~/.bashrc)
export PATH="$HOME/.local/bin:$PATH"

# Verify installation
poetry --version
```

### Configuration

```bash
# Configure poetry to create virtualenvs in project directory
poetry config virtualenvs.in-project true

# Verify configuration
poetry config --list
```

## Project Initialization

### New Project Setup

```bash
# Create new project
poetry new my-project
cd my-project

# Or initialize in existing directory
cd existing-project
poetry init  # Interactive setup

# Set Python version (use latest mature version at project start)
pyenv local 3.12.0

# Create .python-version file manually if needed
echo "3.12.0" > .python-version
```

### Project Structure

```
my-project/
├── .python-version          # pyenv version specification
├── pyproject.toml           # Poetry configuration and dependencies
├── poetry.lock              # Lock file with exact versions
├── README.md                # Project documentation
├── .gitignore               # Git ignore rules
├── claude/                  # Project-specific standards (optional)
├── src/
│   └── my_project/
│       ├── __init__.py
│       └── main.py
└── tests/
    ├── __init__.py
    └── test_main.py
```

### pyproject.toml Configuration

```toml
[tool.poetry]
name = "my-project"
version = "0.1.0"
description = "Project description"
authors = ["Your Name <your.email@example.com>"]
readme = "README.md"
packages = [{include = "my_project", from = "src"}]

[tool.poetry.dependencies]
python = "^3.12"  # Use latest mature version at project start
# Production dependencies (use latest compatible versions)
requests = "^2.31.0"
pydantic = "^2.5.0"

[tool.poetry.group.dev.dependencies]
# Development dependencies (use latest compatible versions)
pytest = "^7.4.0"
pytest-cov = "^4.1.0"
black = "^24.10.0"
ruff = "^0.14.0"
mypy = "^1.7.0"
pre-commit = "^3.6.0"

[tool.poetry.group.docs]
optional = true

[tool.poetry.group.docs.dependencies]
sphinx = "^7.2.0"

[build-system]
requires = ["poetry-core"]
build-backend = "poetry.core.masonry.api"

# Tool configurations
[tool.black]
line-length = 88
target-version = ['py312']  # Match project Python version

[tool.ruff]
line-length = 88
target-version = "py312"  # Match project Python version

[tool.ruff.lint]
select = ["E", "F", "I", "N", "W"]
ignore = []

[tool.mypy]
python_version = "3.12"  # Match project Python version
strict = true
warn_return_any = true
warn_unused_configs = true

[tool.pytest.ini_options]
testpaths = ["tests"]
python_files = ["test_*.py"]
addopts = [
    "-v",
    "--cov=src",
    "--cov-report=term-missing",
    "--cov-fail-under=80",
]
```

## Dependency Management

### Adding Dependencies

```bash
# Add production dependency
poetry add requests

# Add development dependency
poetry add --group dev pytest

# Add with version constraint
poetry add "fastapi>=0.104.0,<0.105.0"

# Add from git repository
poetry add git+https://github.com/user/repo.git

# Add optional dependency group
poetry add --group docs sphinx
```

### Removing Dependencies

```bash
# Remove dependency
poetry remove requests

# Remove dev dependency
poetry remove --group dev pytest
```

### Updating Dependencies

```bash
# Update all dependencies
poetry update

# Update specific dependency
poetry update requests

# Show outdated dependencies
poetry show --outdated
```

### Installing Dependencies

```bash
# Install all dependencies (including dev)
poetry install

# Install only production dependencies
poetry install --only main

# Install without dev dependencies
poetry install --without dev

# Install with optional groups
poetry install --with docs
```

## Virtual Environment

### Managing Virtual Environments

```bash
# Create/activate virtual environment (automatic with poetry)
poetry shell

# Run command in virtual environment without activating
poetry run python script.py
poetry run pytest

# Show virtual environment info
poetry env info

# List virtual environments
poetry env list

# Remove virtual environment
poetry env remove python3.12
```

## Development Tools Setup

### Pre-commit Hooks

```yaml
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/psf/black
    rev: 23.12.0
    hooks:
      - id: black

  - repo: https://github.com/astral-sh/ruff-pre-commit
    rev: v0.1.9
    hooks:
      - id: ruff
        args: [--fix]

  - repo: https://github.com/pre-commit/mirrors-mypy
    rev: v1.7.1
    hooks:
      - id: mypy
        additional_dependencies: [types-requests]

  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v4.5.0
    hooks:
      - id: trailing-whitespace
      - id: end-of-file-fixer
      - id: check-yaml
      - id: check-added-large-files
      - id: check-merge-conflict
```

```bash
# Install pre-commit
poetry add --group dev pre-commit

# Install git hooks
poetry run pre-commit install

# Run on all files manually
poetry run pre-commit run --all-files
```

### EditorConfig

```ini
# .editorconfig
root = true

[*]
charset = utf-8
end_of_line = lf
insert_final_newline = true
trim_trailing_whitespace = true

[*.py]
indent_style = space
indent_size = 4
max_line_length = 88

[*.{json,yml,yaml,toml}]
indent_style = space
indent_size = 2

[Makefile]
indent_style = tab
```

## Git Configuration

### .gitignore

```gitignore
# Python
__pycache__/
*.py[cod]
*$py.class
*.so
.Python
build/
develop-eggs/
dist/
downloads/
eggs/
.eggs/
lib/
lib64/
parts/
sdist/
var/
wheels/
*.egg-info/
.installed.cfg
*.egg

# Virtual environments
.venv/
venv/
ENV/
env/

# IDEs
.vscode/
.idea/
*.swp
*.swo
*~

# Testing
.pytest_cache/
.coverage
htmlcov/
.tox/

# Type checking
.mypy_cache/
.dmypy.json
dmypy.json

# OS
.DS_Store
Thumbs.db

# Environment variables
.env
.env.local

# Logs
*.log
```

## Project Initialization Checklist

### New Project Setup Steps

```bash
# 1. Create project directory
mkdir my-project
cd my-project

# 2. Initialize git
git init
git branch -M main

# 3. Set Python version (use latest mature version)
pyenv local 3.12.0

# 4. Initialize poetry
poetry init

# 5. Create project structure
mkdir -p src/my_project tests

# 6. Create initial files
touch src/my_project/__init__.py
touch src/my_project/main.py
touch tests/__init__.py
touch tests/test_main.py
touch README.md

# 7. Add development tools
poetry add --group dev pytest pytest-cov black ruff mypy pre-commit

# 8. Create configuration files
touch .editorconfig
touch .pre-commit-config.yaml
touch .gitignore

# 9. Install dependencies
poetry install

# 10. Set up pre-commit hooks
poetry run pre-commit install

# 11. Initial commit
git add .
git commit -m "chore: initial project setup"
```

## Environment Variables

### Managing Secrets and Configuration

```bash
# .env (never commit to git!)
DATABASE_URL=postgresql://user:pass@localhost/dbname
API_KEY=secret_key_here
DEBUG=true
```

```python
# Load environment variables
from pathlib import Path
from dotenv import load_dotenv
import os

# Load .env file
env_path = Path('.') / '.env'
load_dotenv(dotenv_path=env_path)

# Access variables
database_url = os.getenv('DATABASE_URL')
api_key = os.getenv('API_KEY')
debug = os.getenv('DEBUG', 'false').lower() == 'true'
```

```bash
# Install python-dotenv
poetry add python-dotenv
```

## Development vs Production

### Separate Dependency Groups

```toml
[tool.poetry.dependencies]
python = "^3.12"  # Use latest mature version at project start
# Only production dependencies (use latest compatible versions)
requests = "^2.31.0"
pydantic = "^2.5.0"

[tool.poetry.group.dev.dependencies]
# Development only (use latest compatible versions)
pytest = "^7.4.0"
black = "^24.10.0"
mypy = "^1.7.0"
```

```bash
# Production install (no dev dependencies)
poetry install --only main

# Development install (all dependencies)
poetry install
```

### Lock File Management

```bash
# Generate/update poetry.lock
poetry lock

# Install from lock file (recommended for production)
poetry install --only main

# Update lock file without upgrading dependencies
poetry lock --no-update
```

**Why lock files**: Ensure identical dependency versions across environments.

## Makefile for Common Tasks

```makefile
# Makefile
.PHONY: install test format lint type-check clean

install:
	poetry install

test:
	poetry run pytest

test-cov:
	poetry run pytest --cov=src --cov-report=html

format:
	poetry run black src tests
	poetry run ruff check --fix src tests

lint:
	poetry run ruff check src tests

type-check:
	poetry run mypy src

clean:
	find . -type d -name __pycache__ -exec rm -rf {} +
	find . -type f -name "*.pyc" -delete
	rm -rf .pytest_cache .mypy_cache .coverage htmlcov

all: format lint type-check test
```

```bash
# Usage
make install    # Install dependencies
make test       # Run tests
make format     # Format code
make lint       # Lint code
make type-check # Type checking
make all        # Run all checks
```

## IDE Configuration

### VS Code Settings

```json
{
  "python.defaultInterpreterPath": "${workspaceFolder}/.venv/bin/python",
  "python.formatting.provider": "black",
  "python.linting.enabled": true,
  "python.linting.ruffEnabled": true,
  "python.testing.pytestEnabled": true,
  "editor.formatOnSave": true,
  "editor.codeActionsOnSave": {
    "source.organizeImports": true
  }
}
```

## Troubleshooting

### Common Issues

```bash
# Poetry not finding Python version
pyenv local 3.12.0
poetry env use $(pyenv which python)

# Clear poetry cache
poetry cache clear pypi --all

# Recreate virtual environment
poetry env remove python3.12
poetry install

# Update poetry itself
poetry self update
```

## Related Standards

- See `python_style.md` for code formatting tools
- See `testing_standards.md` for test configuration
- See `git_workflow.md` for version control setup
- See `documentation_standards.md` for README requirements

---

**Last Updated**: 2025-10-19
**Status**: Strong preference - deviations require justification

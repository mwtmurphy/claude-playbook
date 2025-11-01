# Git Workflow Standards

**Status**: Strong preference - deviations require justification and approval
**Scope**: Version control practices for all projects
**Last Updated**: 2025-10-31

---

## Overview

Consistent git practices create clear project history, make debugging easier, and improve collaboration. Good commits tell a story about how and why code evolved.

**Why**: Clean git history is documentation that shows the reasoning behind changes.

## Commit Message Format

### Conventional Commits

Follow the [Conventional Commits](https://www.conventionalcommits.org/) specification.

**Why**: Machine-readable format enables automated changelog generation and semantic versioning.

### Format

```
<type>(<scope>): <subject>

<body>

<footer>
```

### Types

- `feat:` - New feature
- `fix:` - Bug fix
- `docs:` - Documentation only changes
- `style:` - Code style changes (formatting, missing semicolons, etc.)
- `refactor:` - Code change that neither fixes a bug nor adds a feature
- `perf:` - Performance improvement
- `test:` - Adding or updating tests
- `build:` - Changes to build system or dependencies
- `ci:` - CI/CD configuration changes
- `chore:` - Other changes that don't modify src or test files

### Examples

```bash
# Feature
feat(auth): add OAuth2 authentication support

Implements OAuth2 authentication flow using the authorization code
grant type. Supports Google and GitHub as identity providers.

Closes #123

# Bug fix
fix(database): prevent connection pool exhaustion

The connection pool was not properly releasing connections after
query timeouts. Added explicit connection cleanup in error handlers.

Fixes #456

# Documentation
docs(readme): update installation instructions

Added pyenv setup steps and clarified poetry installation process.

# Refactoring
refactor(user-service): extract validation logic

Moved user validation logic from UserService to separate
UserValidator class for better separation of concerns.

# Multiple related changes
feat(orders): add order cancellation feature

- Add cancel_order endpoint
- Implement cancellation business logic
- Add cancellation email notification
- Update order status model

Closes #789
```

### Commit Subject Guidelines

**Do**:
- Use imperative mood ("add feature" not "added feature")
- Keep under 72 characters
- Don't end with a period
- Be specific and descriptive

```bash
# Good
feat(api): add pagination to user list endpoint
fix(auth): prevent duplicate session creation
docs(contributing): add code review guidelines

# Bad
feat: stuff
fix: fixed bug
update things
```

### Commit Body (Optional but Encouraged)

**When to use**:
- Complex changes requiring explanation
- Multiple related changes in one commit
- Context about why the change was made

**Structure**:
- Blank line after subject
- Wrap at 72 characters
- Explain what and why, not how (code shows how)

```bash
refactor(database): migrate from SQLAlchemy to raw SQL

SQLAlchemy's ORM overhead was causing performance issues in high-throughput
endpoints. Profiling showed 40% of request time spent in ORM operations.

This change:
- Replaces ORM models with dataclasses
- Uses raw SQL with named parameters
- Maintains existing repository interface
- Improves endpoint response time by ~35%

Migration guide: docs/migration/sqlalchemy-to-sql.md

Refs #234
```

### Commit Footer (Optional)

**Use for**:
- Issue references: `Closes #123`, `Fixes #456`, `Refs #789`
- Breaking changes: `BREAKING CHANGE: description`
- Co-authors: `Co-authored-by: Name <email>`

```bash
feat(api): redesign authentication endpoints

BREAKING CHANGE: /auth/login endpoint now returns JWT in response body
instead of Set-Cookie header. Clients must store and send tokens manually.

Migration guide: docs/migration/auth-v2.md

Closes #100
Co-authored-by: Jane Developer <jane@example.com>
```

## Branch Naming

### Convention

```
<type>/<short-description>
```

### Types

- `feature/` - New features
- `bugfix/` - Bug fixes
- `hotfix/` - Urgent production fixes
- `refactor/` - Code refactoring
- `docs/` - Documentation updates
- `test/` - Test additions or updates

### Examples

```bash
# Good
feature/oauth-authentication
bugfix/connection-pool-leak
hotfix/security-vulnerability-cve-2024-1234
refactor/extract-validation-logic
docs/api-documentation
test/user-service-integration-tests

# Bad
fix
johns-branch
new-stuff
WIP
```

**Why**: Clear branch names make it obvious what work is happening where.

## Workflow

### Feature Development

```bash
# Create feature branch from main
git checkout main
git pull origin main
git checkout -b feature/new-authentication

# Make changes, commit frequently
git add src/auth/
git commit -m "feat(auth): add password validation"

git add tests/test_auth.py
git commit -m "test(auth): add password validation tests"

# Push to remote
git push -u origin feature/new-authentication

# Create pull request when ready
```

### Bug Fixes

```bash
# Create bugfix branch from main
git checkout main
git pull origin main
git checkout -b bugfix/fix-memory-leak

# Fix and commit
git add src/cache.py
git commit -m "fix(cache): prevent memory leak in cache cleanup

The cache cleanup task was holding references to expired entries,
preventing garbage collection. Now explicitly clears references.

Fixes #567"

# Push and create PR
git push -u origin bugfix/fix-memory-leak
```

### Hotfixes

```bash
# Create hotfix branch from main (or production tag)
git checkout main
git pull origin main
git checkout -b hotfix/security-patch

# Make critical fix
git add src/security/
git commit -m "fix(security): patch SQL injection vulnerability

SECURITY: Properly escape user input in raw SQL queries.
All user-provided values now use parameterized queries.

Fixes CVE-2024-1234"

# Push and create urgent PR
git push -u origin hotfix/security-patch
```

## Pull Requests

### PR Title

Use Conventional Commit format:

```
feat(auth): add OAuth2 authentication support
fix(database): prevent connection pool exhaustion
docs(readme): update installation instructions
```

### PR Description Template

```markdown
## Summary
Brief description of what this PR does and why.

## Changes
- Bullet point list of main changes
- Keep it concise but informative
- Link to related files/functions

## Testing
- [ ] Unit tests added/updated
- [ ] Integration tests added/updated
- [ ] Manual testing completed

## Checklist
- [ ] Tests pass locally
- [ ] Code follows style guidelines
- [ ] Documentation updated
- [ ] No new warnings or errors

## Related Issues
Closes #123
Refs #456
```

### PR Best Practices

**Do**:
- Keep PRs focused and reasonably sized (< 500 lines when possible)
- Include tests for new functionality
- Update documentation
- Self-review before requesting review
- Respond promptly to review comments

**Don't**:
- Mix unrelated changes in one PR
- Submit untested code
- Leave TODO comments without tracking issues
- Force push after reviews have started (unless requested)

## Merge Strategy

### Prefer Squash Merges

**Why**: Keeps main branch history clean, one commit per feature/fix.

```bash
# Squash merge via GitHub/GitLab UI
# or manually:
git checkout main
git merge --squash feature/new-feature
git commit -m "feat(api): add new feature

Complete implementation of requested feature including:
- Core functionality
- Tests
- Documentation

Closes #123"
```

### When to Use Regular Merge

- When preserving detailed history is important
- For long-lived feature branches with multiple contributors
- When individual commits are meaningful milestones

### Never Force Push to Main

**Exception**: Only with explicit approval and team notification.

## Commit Frequency

### Commit Often, Push Regularly

**During development**:
- Commit logical units of work
- Don't wait until feature is "perfect"
- Use WIP commits if needed (squash before PR)

```bash
# Good: Logical commits during development
git commit -m "feat(api): add user endpoint structure"
git commit -m "feat(api): implement user validation"
git commit -m "feat(api): add user endpoint tests"
git commit -m "docs(api): document user endpoint"

# These will be squashed into one commit when merging to main
```

### Amending Commits

```bash
# Amend last commit (before pushing)
git commit --amend

# Add forgotten changes to last commit
git add forgotten_file.py
git commit --amend --no-edit
```

**Warning**: Never amend commits that have been pushed and reviewed.

## Git Hooks

### Pre-commit Hooks

Automatically run checks before allowing commits.

**Setup with pre-commit**:

```yaml
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/psf/black
    rev: 24.1.0
    hooks:
      - id: black

  - repo: https://github.com/pycqa/isort
    rev: 5.13.0
    hooks:
      - id: isort

  - repo: https://github.com/pycqa/flake8
    rev: 7.0.0
    hooks:
      - id: flake8

  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v4.5.0
    hooks:
      - id: trailing-whitespace
      - id: end-of-file-fixer
      - id: check-yaml
      - id: check-added-large-files
```

**Install**:
```bash
poetry add --group dev pre-commit
poetry run pre-commit install
```

**Why**: Catches issues before they enter version control.

## Stashing Changes

### Temporary Storage for Unfinished Work

```bash
# Save current changes
git stash save "WIP: working on feature"

# Switch branches, do other work
git checkout main
git pull origin main

# Return to work
git checkout feature/my-feature
git stash pop
```

## Tags and Releases

### Semantic Versioning

```bash
# Tag releases with semantic versioning
git tag -a v1.2.0 -m "Release version 1.2.0

- Added OAuth2 authentication
- Fixed connection pool issues
- Performance improvements"

git push origin v1.2.0
```

**Format**: `v<major>.<minor>.<patch>`
- Major: Breaking changes
- Minor: New features (backward compatible)
- Patch: Bug fixes

## Git Configuration

### Recommended Settings

```bash
# Set your identity
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"

# Set default branch name
git config --global init.defaultBranch main

# Enable helpful colors
git config --global color.ui auto

# Better diff output
git config --global diff.algorithm histogram

# Auto-correct typos
git config --global help.autocorrect 1
```

## Common Operations

### Undoing Changes

```bash
# Undo uncommitted changes to a file
git checkout -- filename.py

# Unstage file
git reset HEAD filename.py

# Undo last commit (keep changes)
git reset --soft HEAD~1

# Undo last commit (discard changes) - USE WITH CAUTION
git reset --hard HEAD~1
```

### Reviewing History

```bash
# View commit history
git log --oneline --graph --decorate

# View changes in a commit
git show <commit-hash>

# View changes for a file
git log -p filename.py

# Search commits
git log --grep="authentication"
```

## Related Standards

- See `python_style.md` for Python code formatting enforced by pre-commit hooks
- See `sql_style.md` for SQL formatting standards
- See `documentation_standards.md` for commit message documentation and PR descriptions
- See `testing_standards.md` for test requirements before commits
- See `claude_workflow.md` for Claude Code specific git integration

# Claude Code Permissions Patterns

## Purpose
This document defines common permission patterns for Claude Code across different project types. Permissions control which bash commands and tools Claude Code can execute automatically without requiring user approval.

## Permission Structure

### Configuration Location
Permissions are configured in `.claude/settings.local.json`:
- **Workspace-level**: `/Users/username/projects/.claude/settings.local.json`
- **Project-level**: `/project-root/.claude/settings.local.json`

### Permission Format
```json
{
  "permissions": {
    "allow": [
      "Bash(command:pattern)",
      "WebFetch(domain:example.com)"
    ],
    "deny": [],
    "ask": []
  }
}
```

## Permission Patterns

### Wildcards
- `*` - Matches any characters
- `:*` - Matches any arguments after command

```json
{
  "permissions": {
    "allow": [
      "Bash(git status)",        // Exact command
      "Bash(git diff:*)",        // Command with any arguments
      "Bash(npm test*)",         // Command starting with "npm test"
      "Bash(pytest:*)"           // pytest with any arguments
    ]
  }
}
```

## Python Project Permissions

### Basic Python Tools
```json
{
  "permissions": {
    "allow": [
      "Bash(python:*)",
      "Bash(python3:*)",
      "Bash(pytest:*)",
      "Bash(poetry run:*)",
      "Bash(poetry install:*)",
      "Bash(poetry add:*)",
      "Bash(poetry shell:*)"
    ]
  }
}
```

### Code Quality Tools
```json
{
  "permissions": {
    "allow": [
      "Bash(black:*)",
      "Bash(ruff:*)",
      "Bash(ruff check:*)",
      "Bash(mypy:*)",
      "Bash(pylint:*)",
      "Bash(flake8:*)",
      "Bash(isort:*)"
    ]
  }
}
```

### Virtual Environment
```json
{
  "permissions": {
    "allow": [
      "Bash(source venv/bin/activate:*)",
      "Bash(pip install:*)",
      "Bash(pip list)",
      "Bash(pip freeze)"
    ]
  }
}
```

## JavaScript/TypeScript Project Permissions

### npm Commands
```json
{
  "permissions": {
    "allow": [
      "Bash(npm install:*)",
      "Bash(npm run build:*)",
      "Bash(npm run dev:*)",
      "Bash(npm test)",
      "Bash(npm test:*)",
      "Bash(npm run lint)",
      "Bash(npm run lint:*)",
      "Bash(npm run type-check:*)",
      "Bash(npm run format:*)"
    ]
  }
}
```

### pnpm/yarn Alternatives
```json
{
  "permissions": {
    "allow": [
      "Bash(pnpm install:*)",
      "Bash(pnpm run:*)",
      "Bash(pnpm test:*)",
      "Bash(yarn install)",
      "Bash(yarn run:*)",
      "Bash(yarn test:*)"
    ]
  }
}
```

### Node and Testing
```json
{
  "permissions": {
    "allow": [
      "Bash(node:*)",
      "Bash(npx:*)",
      "Bash(jest:*)",
      "Bash(vitest:*)",
      "Bash(playwright test:*)"
    ]
  }
}
```

## Git Operations

### Read-Only Git Commands
Safe commands that don't modify repository:
```json
{
  "permissions": {
    "allow": [
      "Bash(git status)",
      "Bash(git diff:*)",
      "Bash(git log:*)",
      "Bash(git show:*)",
      "Bash(git branch:*)",
      "Bash(git remote:*)"
    ]
  }
}
```

### Modification Commands
Commands that modify the repository (use with caution):
```json
{
  "permissions": {
    "allow": [
      "Bash(git add:*)",
      "Bash(git commit:*)",
      "Bash(git push:*)",
      "Bash(git pull:*)",
      "Bash(git checkout:*)",
      "Bash(git restore:*)",
      "Bash(git merge:*)",
      "Bash(git rebase:*)"
    ]
  }
}
```

### GitHub CLI (gh)
```json
{
  "permissions": {
    "allow": [
      "Bash(gh pr create:*)",
      "Bash(gh pr merge:*)",
      "Bash(gh pr view:*)",
      "Bash(gh pr checks:*)",
      "Bash(gh pr list)",
      "Bash(gh issue create:*)",
      "Bash(gh issue view:*)",
      "Bash(gh repo view:*)"
    ]
  }
}
```

## Database Operations

### PostgreSQL
```json
{
  "permissions": {
    "allow": [
      "Bash(psql:*)",
      "Bash(pg_dump:*)",
      "Bash(createdb:*)",
      "Bash(dropdb:*)"
    ]
  }
}
```

### MySQL
```json
{
  "permissions": {
    "allow": [
      "Bash(mysql:*)",
      "Bash(mysqldump:*)",
      "Bash(mysqlshow:*)"
    ]
  }
}
```

### Database Migrations
```json
{
  "permissions": {
    "allow": [
      "Bash(alembic:*)",
      "Bash(flask db:*)",
      "Bash(django-admin migrate:*)",
      "Bash(python manage.py migrate:*)"
    ]
  }
}
```

## Docker & Container Operations

### Docker Commands
```json
{
  "permissions": {
    "allow": [
      "Bash(docker build:*)",
      "Bash(docker run:*)",
      "Bash(docker ps:*)",
      "Bash(docker logs:*)",
      "Bash(docker exec:*)",
      "Bash(docker-compose up:*)",
      "Bash(docker-compose down:*)",
      "Bash(docker-compose logs:*)"
    ]
  }
}
```

## Shell Scripting & System Tools

### Script Execution
```json
{
  "permissions": {
    "allow": [
      "Bash(./setup.sh:*)",
      "Bash(./build.sh:*)",
      "Bash(./test.sh:*)",
      "Bash(bash scripts/*)",
      "Bash(shellcheck:*)",
      "Bash(chmod:*)"
    ]
  }
}
```

### macOS Specific
```json
{
  "permissions": {
    "allow": [
      "Bash(brew install:*)",
      "Bash(brew update)",
      "Bash(brew upgrade:*)",
      "Bash(brew list)",
      "Bash(defaults write:*)",
      "Bash(defaults read:*)"
    ]
  }
}
```

## Web Fetch Permissions

### Domain Restrictions
```json
{
  "permissions": {
    "allow": [
      "WebFetch(domain:github.com)",
      "WebFetch(domain:api.example.com)",
      "WebFetch(domain:docs.example.com)"
    ]
  }
}
```

## Project Type Templates

### Python Web Application (Django/Flask)
```json
{
  "permissions": {
    "allow": [
      "Bash(python:*)",
      "Bash(python3:*)",
      "Bash(poetry run:*)",
      "Bash(poetry install:*)",
      "Bash(pytest:*)",
      "Bash(black:*)",
      "Bash(ruff:*)",
      "Bash(mypy:*)",
      "Bash(alembic:*)",
      "Bash(psql:*)",
      "Bash(git status)",
      "Bash(git diff:*)",
      "Bash(git log:*)"
    ]
  }
}
```

### JavaScript/TypeScript Chrome Extension
```json
{
  "permissions": {
    "allow": [
      "Bash(npm install:*)",
      "Bash(npm run build:*)",
      "Bash(npm test)",
      "Bash(npm test:*)",
      "Bash(npm run lint)",
      "Bash(npm run lint:*)",
      "Bash(npm run type-check:*)",
      "Bash(git status)",
      "Bash(git diff:*)",
      "Bash(git log:*)",
      "Bash(git add:*)",
      "Bash(git commit:*)",
      "Bash(git push:*)",
      "Bash(gh pr create:*)",
      "Bash(gh pr merge:*)"
    ]
  }
}
```

### Full-Stack Application
```json
{
  "permissions": {
    "allow": [
      "Bash(python:*)",
      "Bash(poetry run:*)",
      "Bash(pytest:*)",
      "Bash(black:*)",
      "Bash(ruff:*)",
      "Bash(mypy:*)",
      "Bash(npm install:*)",
      "Bash(npm run:*)",
      "Bash(npm test:*)",
      "Bash(docker-compose:*)",
      "Bash(psql:*)",
      "Bash(alembic:*)",
      "Bash(git status)",
      "Bash(git diff:*)",
      "Bash(git log:*)"
    ]
  }
}
```

### Environment Setup / DevOps
```json
{
  "permissions": {
    "allow": [
      "Bash(./setup.sh:*)",
      "Bash(./tests/test_setup.sh:*)",
      "Bash(chmod:*)",
      "Bash(brew install:*)",
      "Bash(shellcheck:*)",
      "Bash(git checkout:*)",
      "Bash(git add:*)",
      "Bash(git commit:*)",
      "Bash(git push:*)",
      "Bash(gh pr create:*)",
      "Bash(gh pr merge:*)",
      "Bash(gh pr view:*)",
      "Bash(gh pr checks:*)",
      "WebFetch(domain:github.com)"
    ]
  }
}
```

## Workspace vs Project Level

### Workspace-Level Permissions
Apply to all projects in the workspace:
```json
{
  "permissions": {
    "allow": [
      "Bash(pytest:*)",
      "Bash(poetry run:*)",
      "Bash(black:*)",
      "Bash(ruff:*)",
      "Bash(mypy:*)"
    ]
  }
}
```

Location: `/Users/username/projects/.claude/settings.local.json`

### Project-Level Permissions
Extend or override workspace permissions for specific projects:
```json
{
  "permissions": {
    "allow": [
      "Bash(npm test:*)",
      "Bash(git add:*)",
      "Bash(git commit:*)",
      "Bash(gh pr create:*)"
    ]
  }
}
```

Location: `/project-root/.claude/settings.local.json`

## Security Considerations

### Safe Patterns
✅ **Safe to allow**:
- Read-only git commands (status, diff, log)
- Testing commands (pytest, npm test)
- Linting/formatting tools (black, ruff, eslint)
- Read-only database queries

### Caution Required
⚠️ **Use with caution**:
- Git push/merge operations
- Database modification commands
- Docker container operations
- System-wide package installations

### Avoid Auto-Approval
❌ **Should require manual approval**:
- `rm -rf` or destructive file operations
- `sudo` commands
- Production database operations
- Force push operations
- System configuration changes

## Permission Inheritance

### Resolution Order
1. Project-level `deny` (highest priority)
2. Project-level `allow`
3. Workspace-level `deny`
4. Workspace-level `allow`
5. Default (ask user)

### Example
```json
// Workspace level
{
  "permissions": {
    "allow": ["Bash(git push:*)"]
  }
}

// Project level (overrides workspace)
{
  "permissions": {
    "deny": ["Bash(git push:*)"],
    "allow": ["Bash(git push origin feature/*:*)"]
  }
}
```

## Best Practices

### Permission Granularity
- **Specific over general**: Prefer `Bash(pytest:*)` over `Bash(pytest*)`
- **Argument constraints**: Use `:*` to require arguments
- **Read before write**: Allow read operations broadly, restrict write operations

### Maintenance
- Review permissions quarterly
- Remove unused permissions
- Document why certain permissions are granted
- Use comments in JSON (if tool supports) or separate documentation

### Testing Permissions
```bash
# Test if a command requires approval
claude --test-permission "git push origin main"

# List all allowed commands
claude --list-permissions
```

## Examples from Real Projects

### Centre (Chrome Extension)
```json
{
  "permissions": {
    "allow": [
      "Bash(git checkout:*)",
      "Bash(git add:*)",
      "Bash(git commit:*)",
      "Bash(git push:*)",
      "Bash(gh pr merge:*)",
      "Bash(git pull:*)",
      "Bash(git branch:*)",
      "Bash(git remote:*)",
      "Bash(npm run build:*)",
      "Bash(npm test)",
      "Bash(npm test:*)",
      "Bash(git restore:*)",
      "Bash(npm run lint)",
      "Bash(npm run type-check:*)",
      "Bash(npm install:*)",
      "Bash(npm run lint:*)",
      "Bash(gh pr create:*)",
      "Bash(npm run test:*)"
    ]
  }
}
```

### Mac-Env-Setup (DevOps)
```json
{
  "permissions": {
    "allow": [
      "Bash(git checkout:*)",
      "Bash(chmod:*)",
      "Bash(./tests/test_setup.sh:*)",
      "Bash(./setup.sh:*)",
      "Bash(git add:*)",
      "Bash(git push:*)",
      "Bash(gh pr view:*)",
      "Bash(gh pr checks:*)",
      "WebFetch(domain:github.com)",
      "Bash(shellcheck:*)",
      "Bash(git commit:*)",
      "Bash(brew install:*)",
      "Bash(gh pr merge:*)",
      "Bash(gh pr create:*)"
    ]
  }
}
```

## Troubleshooting

### Permission Denied Errors
If Claude Code asks for approval unexpectedly:
1. Check if the command matches the pattern exactly
2. Verify the pattern includes `:*` for arguments
3. Check project-level permissions don't override workspace
4. Ensure no typos in permission patterns

### Over-Permissive Warnings
If you receive security warnings:
1. Review all `allow` patterns
2. Move sensitive operations to `ask` list
3. Add specific `deny` patterns for dangerous operations
4. Consider splitting permissions between workspace and project

## Related Standards
- See `git_workflow.md` for recommended git operations
- See `python_style.md` for Python tooling
- See `typescript_style.md` for JavaScript/TypeScript tooling
- See `chrome_extension_standards.md` for extension development

## References
- [Claude Code Permissions Documentation](https://docs.claude.com/claude-code/permissions)
- [Security Best Practices](https://docs.claude.com/claude-code/security)

---

**Last Updated**: 2025-11-02
**Status**: Strong preference - deviations require justification and approval

# Usage Guide: Integrating Development Standards

This guide explains how to integrate the playbook standards into your projects using Claude Code.

## Table of Contents

- [Quick Start](#quick-start)
- [Claude Code Configuration](#claude-code-configuration)
- [Integration Patterns](#integration-patterns)
- [Example Workflows](#example-workflows)
- [Troubleshooting](#troubleshooting)

## Quick Start

### 1. Identify Relevant Standards

Choose which standards apply to your project:

- **All Python projects**: `python_style.md`, `testing_standards.md`, `documentation_standards.md`, `git_workflow.md`
- **Python with SQL**: Add `sql_style.md`, `database_standards.md`
- **Streamlit projects**: Add `streamlit_standards.md`
- **All projects**: `architecture_patterns.md`, `error_handling.md`, `performance_considerations.md`

### 2. Add Remote References

Use raw GitHub URLs to reference standards:

```
https://raw.githubusercontent.com/mwtmurphy/playbook/main/claude/python_style.md
https://raw.githubusercontent.com/mwtmurphy/playbook/main/claude/sql_style.md
https://raw.githubusercontent.com/mwtmurphy/playbook/main/claude/testing_standards.md
```

### 3. Configure Claude Code

Add these URLs to your project's Claude Code configuration or provide them in your context when starting a session.

## Claude Code Configuration

### Method 1: Project-Level Settings

Create or update `claude/settings.json` in your project:

```json
{
  "references": [
    "https://raw.githubusercontent.com/mwtmurphy/playbook/main/claude/python_style.md",
    "https://raw.githubusercontent.com/mwtmurphy/playbook/main/claude/testing_standards.md",
    "https://raw.githubusercontent.com/mwtmurphy/playbook/main/claude/architecture_patterns.md"
  ]
}
```

### Method 2: Manual Context

When starting a Claude Code session, provide standards in your initial prompt:

```
Please reference these standards while working:
- https://raw.githubusercontent.com/mwtmurphy/playbook/main/claude/python_style.md
- https://raw.githubusercontent.com/mwtmurphy/playbook/main/claude/sql_style.md

Help me implement a user authentication system following these standards.
```

### Method 3: Local Copy

For projects that need customization:

```bash
# Copy standards to your project
mkdir -p claude
cd claude
curl -O https://raw.githubusercontent.com/mwtmurphy/playbook/main/claude/python_style.md
curl -O https://raw.githubusercontent.com/mwtmurphy/playbook/main/claude/testing_standards.md
# Customize as needed for your project
```

## Integration Patterns

### Pattern 1: Full Standard Suite (Recommended for New Projects)

Reference all relevant standards from the start:

**Python + SQL + Streamlit Project:**
```
References:
- python_style.md
- sql_style.md
- streamlit_standards.md
- testing_standards.md
- documentation_standards.md
- architecture_patterns.md
- error_handling.md
- database_standards.md
- git_workflow.md
- claude_workflow.md
```

### Pattern 2: Incremental Adoption (For Existing Projects)

Start with core standards and add more as you refactor:

**Phase 1 - Style & Testing:**
```
- python_style.md
- testing_standards.md
- git_workflow.md
```

**Phase 2 - Architecture & Patterns:**
```
Add: architecture_patterns.md, error_handling.md
```

**Phase 3 - Framework-Specific:**
```
Add: streamlit_standards.md, database_standards.md
```

### Pattern 3: Hybrid Local/Remote

Keep commonly customized standards local, reference stable ones remotely:

**Local (in project claude/):**
- `architecture_patterns.md` (customized for your domain)
- `streamlit_standards.md` (project-specific components)

**Remote:**
- `python_style.md`
- `sql_style.md`
- `testing_standards.md`
- `git_workflow.md`

## Example Workflows

### Starting a New Python Project

```bash
# 1. Initialize project
mkdir my_project && cd my_project
git init
poetry init

# 2. Create Claude Code configuration
mkdir claude
cat > claude/settings.json << EOF
{
  "references": [
    "https://raw.githubusercontent.com/mwtmurphy/playbook/main/claude/python_style.md",
    "https://raw.githubusercontent.com/mwtmurphy/playbook/main/claude/testing_standards.md",
    "https://raw.githubusercontent.com/mwtmurphy/playbook/main/claude/documentation_standards.md",
    "https://raw.githubusercontent.com/mwtmurphy/playbook/main/claude/architecture_patterns.md",
    "https://raw.githubusercontent.com/mwtmurphy/playbook/main/claude/git_workflow.md"
  ]
}
EOF

# 3. Start Claude Code session
claude-code
# Claude will now reference these standards during development
```

### Refactoring an Existing Project

```bash
# 1. Add standards gradually
mkdir -p claude
cat > claude/settings.json << EOF
{
  "references": [
    "https://raw.githubusercontent.com/mwtmurphy/playbook/main/claude/python_style.md"
  ]
}
EOF

# 2. Start with one module
claude-code
# Prompt: "Refactor the auth module to comply with python_style.md standards"

# 3. Expand to more standards as you refactor more modules
# Add testing_standards.md, architecture_patterns.md, etc.
```

### Adding Streamlit Standards to Existing Project

```bash
# Add Streamlit-specific standards
cat > claude/settings.json << EOF
{
  "references": [
    "https://raw.githubusercontent.com/mwtmurphy/playbook/main/claude/python_style.md",
    "https://raw.githubusercontent.com/mwtmurphy/playbook/main/claude/streamlit_standards.md"
  ]
}
EOF
```

## Best Practices

### 1. Start Early

Reference standards from project inception rather than retrofitting later.

### 2. Document Deviations

When you need to deviate from standards, document why:

```python
# Deviation from python_style.md: Using short variable names
# Justification: Mathematical notation conventions (Einstein summation)
# Approved by: [Name] on [Date]
def einsum(a, b):
    ...
```

### 3. Keep Standards Updated

Regularly check for updates to the playbook repository:

```bash
# If using local copies
cd claude
git remote add playbook https://github.com/mwtmurphy/playbook.git
git fetch playbook
git diff HEAD:python_style.md playbook/main:claude/python_style.md
```

### 4. Selective Application

Don't blindly apply all standards. Choose what makes sense for your project:

- Small scripts might not need full architecture patterns
- Simple CRUD apps might not need complex error handling
- Prototypes can defer some documentation standards

### 5. Team Alignment

Ensure your team understands which standards apply:

```markdown
# docs/DEVELOPMENT.md

## Development Standards

This project follows the [mwtmurphy/playbook](https://github.com/mwtmurphy/playbook) standards:

- Python Style: [Link]
- Testing: [Link]
- Git Workflow: [Link]

See claude/settings.json for full list of referenced standards.
```

## Troubleshooting

### Standards Not Being Applied

**Issue**: Claude Code isn't following the standards.

**Solutions**:
1. Verify the raw GitHub URLs are accessible
2. Check `claude/settings.json` syntax is valid
3. Explicitly mention standards in your prompts
4. Ensure standards are appropriate for your context

### Conflicting Standards

**Issue**: Project requirements conflict with playbook standards.

**Solutions**:
1. Create local copy of the standard and modify it
2. Document the deviation clearly
3. Consider proposing an update to the playbook if the deviation is broadly applicable

### Version Mismatches

**Issue**: Standards updated but project needs older version.

**Solutions**:
1. Use specific commit URLs instead of `main`:
   ```
   https://raw.githubusercontent.com/mwtmurphy/playbook/<commit-sha>/claude/python_style.md
   ```
2. Copy standards locally and version control them
3. Update gradually when ready

### Standards Too Strict

**Issue**: Standards are too opinionated for your project.

**Solutions**:
1. Use standards as guidelines, not rules
2. Create project-specific adaptations
3. Reference only the standards that add value
4. Propose updates to make standards more flexible

## Advanced Usage

### Custom Project Standards

Extend playbook standards with project-specific rules:

```bash
# claude/project_standards.md
# Project-Specific Standards

This project extends the playbook standards with:

## Domain-Specific Naming
- Use medical terminology: `patient`, `diagnosis`, `treatment`
- Follow HIPAA naming: `phi_` prefix for protected fields

## Additional Testing Requirements
- All data access must have integration tests
- HIPAA compliance tests required for PHI handling

For base standards, see:
- [playbook/python_style.md]
- [playbook/testing_standards.md]
```

### Standards as Code Reviews

Use standards during PR reviews:

```bash
# .github/workflows/standards-check.yml
name: Standards Compliance Check

on: [pull_request]

jobs:
  standards:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Check Python Style
        run: |
          # Download standards
          curl -O https://raw.githubusercontent.com/mwtmurphy/playbook/main/claude/python_style.md
          # Run Claude Code to review PR against standards
          claude-code review --standards python_style.md
```

### Multi-Language Projects

Reference standards from multiple playbooks:

```json
{
  "references": [
    "https://raw.githubusercontent.com/mwtmurphy/playbook/main/claude/python_style.md",
    "https://raw.githubusercontent.com/another-org/js-playbook/main/javascript_style.md"
  ]
}
```

## Support

- **Issues**: [GitHub Issues](https://github.com/mwtmurphy/playbook/issues)
- **Wiki**: [Human-friendly guides](https://github.com/mwtmurphy/playbook/wiki)
- **Updates**: Watch the repository for changes

---

**Last Updated**: 2025-10-29
**Repository**: [mwtmurphy/playbook](https://github.com/mwtmurphy/playbook)

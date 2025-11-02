# Development Standards Reference

## Overview

This directory contains centralized development standards for Python, SQL, TypeScript, and JavaScript projects. These files serve as reference guides for Claude Code and human developers to maintain consistency, quality, and best practices across multiple projects. Standards can be remotely referenced from this repository using raw GitHub URLs.

## Purpose

These reference files establish **strong preferences** for development practices. While they should be followed in most cases, justified deviations are permitted when:

- Project-specific requirements necessitate different approaches
- Framework or library conventions conflict with these standards
- Performance or technical constraints require alternative solutions
- A compelling technical argument supports the deviation

**Important**: Any deviation from these standards requires:
1. Clear justification documented in code comments or PR descriptions
2. Human approval before implementation
3. Consideration of the impact on project consistency

## How Claude Code Should Use These Files

### Remote References

Claude Code in other projects can reference these standards using raw GitHub URLs:

```
https://raw.githubusercontent.com/mwtmurphy/claude-playbook/main/claude/python_style.md
https://raw.githubusercontent.com/mwtmurphy/claude-playbook/main/claude/sql_style.md
https://raw.githubusercontent.com/mwtmurphy/claude-playbook/main/claude/architecture_patterns.md
```

Add these URLs to Claude Code settings or context to provide standards during development sessions.

### Application Guidelines

When working on Python or SQL projects with these standards loaded, Claude Code should:

1. **Reference these standards** for all code-related decisions
2. **Apply standards consistently** across the project
3. **Flag conflicts** when project code doesn't align with standards
4. **Suggest improvements** to bring existing code into compliance
5. **Request approval** before deviating from any standard
6. **Document deviations** clearly when approved

These standards are **advisory guidelines with strong preference weight**, not rigid rules.

## Centralized vs Project-Level Standards

### Centralized Standards (This Repository)

- **Framework-agnostic**: Applicable to Python, SQL, TypeScript, and JavaScript projects
- **General principles**: Architecture, style, testing, documentation
- **Tool preferences**:
  - Python: pyenv, poetry, pytest, black, ruff, mypy
  - JavaScript/TypeScript: npm, Jest, Playwright, ESLint, Prettier, Webpack
- **Universal patterns**: Error handling, performance, git workflow, permissions
- **Remotely accessible**: Can be referenced via raw GitHub URLs

### Project-Level Standards

Individual projects may have their own `.claude/` or `claude/` directories containing:
- Framework-specific patterns (Django, FastAPI, Flask, React, Vue, etc.)
- Database-specific conventions (PostgreSQL, MySQL, SQLite)
- Chrome extension specific implementation details
- API specifications and schemas
- Domain-specific business rules
- Integration requirements
- Local copies or adaptations of centralized standards

**Hierarchy**: Project-level standards enrich and extend centralized standards. When conflicts arise, project-level standards take precedence, but should document why they deviate from centralized standards.

## Reference Files

### Python & SQL Standards

| File | Purpose |
|------|---------|
| `python_style.md` | Python PEP 8 compliance, naming conventions, type hints, formatting |
| `sql_style.md` | SQL formatting standards, naming conventions, query layout |
| `testing_standards.md` | pytest best practices, coverage targets, test patterns |
| `error_handling.md` | Exception patterns, logging standards, graceful degradation |
| `database_standards.md` | SQL file organisation, query optimisation, migration conventions |
| `environment_setup.md` | pyenv and poetry setup, Python version selection, dependency management |
| `streamlit_standards.md` | Streamlit-specific patterns, state management, component organisation |
| `metricflow_dbt_standards.md` | dbt-core with MetricFlow semantic layer patterns, metric development |
| `data_visualization_standards.md` | Plotly colour palettes, accessibility, chart guidelines, date formatting |
| `interactive_visualization_testing.md` | Testing standards for interactive visualisations |
| `user_journey_diagrams.md` | Customer journey mapping standards and Mermaid patterns |
| `logging_standards.md` | Logging levels, structured logging, log formatting, and performance |
| `security_standards.md` | Authentication, authorisation, secrets management, input validation |
| `api_design_standards.md` | REST API design, versioning, error responses, HTTP best practices |

### JavaScript & TypeScript Standards

| File | Purpose |
|------|---------|
| `typescript_style.md` | ESLint, Prettier, type annotations, naming conventions, patterns |
| `chrome_extension_standards.md` | Manifest V3, service workers, UI/UX patterns, security |
| `javascript_testing.md` | Jest unit tests, Playwright E2E tests, mocking patterns |
| `webpack_standards.md` | Build configuration, optimisation, bundling, asset management |

### General Standards

| File | Purpose |
|------|---------|
| `architecture_patterns.md` | Framework-agnostic design principles, modularity, file organisation |
| `documentation_standards.md` | Docstring requirements, README specs, inline comments, British English |
| `documentation_for_claude.md` | Writing documentation optimised for AI consumption |
| `documentation_for_employees.md` | Writing documentation optimised for human reading |
| `git_workflow.md` | Conventional Commits, branch naming, PR guidelines |
| `permissions_patterns.md` | Claude Code permission patterns for different tech stacks |
| `performance_considerations.md` | Algorithm complexity, profiling, caching, optimisation approach |
| `claude_workflow.md` | Claude Code planning workflow, plan documentation, task tracking |
| `project_setup_prompt.md` | **Reusable prompt** for creating project-level reference files |
| `reference_guide.md` | Meta-guidance for building and organising Claude Code reference documentation |

## Update Process

These reference files are **living documents** but should be updated deliberately:

1. **Changes require human approval**: No automated updates
2. **Version control**: All changes tracked via git commits
3. **Team discussion**: Significant changes should be discussed before implementation
4. **Documentation**: Update reasoning should be documented in commit messages
5. **Consistency**: Changes should be applied across all reference files where relevant

## Getting Started

### For New Projects

1. **Reference standards remotely**: Add relevant raw GitHub URLs to Claude Code settings
2. **Review environment setup**: Consult `environment_setup.md` for initial project setup
3. **Follow architecture patterns**: Reference `architecture_patterns.md` for directory structure
4. **Apply coding standards**: Follow `python_style.md`, `sql_style.md`, and `documentation_standards.md` from the start
5. **Set up testing**: Implement per `testing_standards.md`
6. **Configure git workflow**: Follow `git_workflow.md`
7. **Optional**: Use `project_setup_prompt.md` with Claude Code to create project-specific `claude/` files

### For Existing Projects

1. **Add remote references**: Configure Claude Code to reference these standards
2. **Use during refactoring**: Apply standards when refactoring or adding features
3. **Gradual alignment**: Align existing code with standards during normal maintenance
4. **Prioritize new code**: Ensure new code follows standards from the start
5. **Document deviations**: Track and justify any deviations from standards

### Integration Methods

- **Remote reference** (recommended): Add raw GitHub URLs to Claude Code settings for live updates
- **Local copy**: Clone or copy specific standards into project `claude/` directory
- **Hybrid**: Reference some standards remotely, customize others locally

## Questions or Updates

If you need to:
- **Clarify a standard**: Request human review and discussion
- **Propose a change**: Create an issue or discussion for team review
- **Report a conflict**: Document the specific scenario and request guidance

---

**Last Updated**: 2025-11-02
**Repository**: [mwtmurphy/claude-playbook](https://github.com/mwtmurphy/claude-playbook)
**Scope**: Python, SQL, TypeScript, and JavaScript projects
**Status**: Active - Strong preferences with justified deviations allowed

# Development Standards Reference

## Overview

This directory contains centralized development standards for Python and SQL projects. These files serve as reference guides for Claude Code and human developers to maintain consistency, quality, and best practices across multiple projects. Standards can be remotely referenced from this repository using raw GitHub URLs.

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
https://raw.githubusercontent.com/mwtmurphy/playbook/main/claude/python_style.md
https://raw.githubusercontent.com/mwtmurphy/playbook/main/claude/sql_style.md
https://raw.githubusercontent.com/mwtmurphy/playbook/main/claude/architecture_patterns.md
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

- **Framework-agnostic**: Applicable to any Python or SQL project
- **General principles**: Architecture, style, testing, documentation
- **Tool preferences**: pyenv, poetry, pytest
- **Universal patterns**: Error handling, performance, git workflow
- **Remotely accessible**: Can be referenced via raw GitHub URLs

### Project-Level Standards

Individual projects may have their own `claude/` directories containing:
- Framework-specific patterns (Django, FastAPI, Flask, etc.)
- Database-specific conventions (PostgreSQL, MySQL, SQLite)
- API specifications and schemas
- Domain-specific business rules
- Integration requirements
- Local copies or adaptations of centralized standards

**Hierarchy**: Project-level standards enrich and extend centralized standards. When conflicts arise, project-level standards take precedence, but should document why they deviate from centralized standards.

## Reference Files

| File | Purpose |
|------|---------|
| `python_style.md` | Python PEP 8 compliance, naming conventions, type hints, formatting |
| `sql_style.md` | SQL formatting standards, naming conventions, query layout |
| `architecture_patterns.md` | Framework-agnostic design principles, modularity, file organization |
| `testing_standards.md` | pytest best practices, coverage targets, test patterns |
| `documentation_standards.md` | Docstring requirements, README specs, inline comments |
| `git_workflow.md` | Conventional Commits, branch naming, PR guidelines |
| `claude_workflow.md` | Claude Code planning workflow, plan documentation, task tracking |
| `error_handling.md` | Exception patterns, logging standards, graceful degradation |
| `database_standards.md` | SQL file organization, query optimization, migration conventions |
| `performance_considerations.md` | Algorithm complexity, profiling, caching, optimization approach |
| `environment_setup.md` | pyenv and poetry setup, Python version selection, dependency management |
| `project_setup_prompt.md` | **Reusable prompt** for creating project-level reference files |

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

**Last Updated**: 2025-10-29
**Repository**: [mwtmurphy/playbook](https://github.com/mwtmurphy/playbook)
**Scope**: Python and SQL projects using pyenv and poetry
**Status**: Active - Strong preferences with justified deviations allowed

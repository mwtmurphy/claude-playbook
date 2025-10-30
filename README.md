# Development Standards Playbook

A centralised repository of technical standards and best practices for Python and SQL development, designed to be referenced by Claude Code for consistent implementation across projects.

## Purpose

This repository serves as a reference for Claude Code integration. Standards files can be remotely referenced in Claude Code settings across multiple projects, ensuring consistent implementation of technical practices.

## Quick Reference

| Standard | Description | File |
|----------|-------------|------|
| **Python Style** | PEP 8 compliance, naming conventions, type hints, formatting | [python_style.md](claude/python_style.md) |
| **SQL Style** | SQL formatting standards, naming conventions, query layout | [sql_style.md](claude/sql_style.md) |
| **Architecture Patterns** | Framework-agnostic design principles, modularity, file organization | [architecture_patterns.md](claude/architecture_patterns.md) |
| **Testing Standards** | pytest best practices, coverage targets, test patterns | [testing_standards.md](claude/testing_standards.md) |
| **Documentation Standards** | Docstring requirements, README specs, inline comments | [documentation_standards.md](claude/documentation_standards.md) |
| **Git Workflow** | Conventional Commits, branch naming, PR guidelines | [git_workflow.md](claude/git_workflow.md) |
| **Claude Workflow** | Claude Code planning workflow, plan documentation, task tracking | [claude_workflow.md](claude/claude_workflow.md) |
| **Error Handling** | Exception patterns, logging standards, graceful degradation | [error_handling.md](claude/error_handling.md) |
| **Database Standards** | SQL file organization, query optimization, migration conventions | [database_standards.md](claude/database_standards.md) |
| **Performance Considerations** | Algorithm complexity, profiling, caching, optimization approach | [performance_considerations.md](claude/performance_considerations.md) |
| **Environment Setup** | pyenv and poetry setup, Python version selection, dependency management | [environment_setup.md](claude/environment_setup.md) |
| **Streamlit Standards** | Streamlit-specific patterns, state management, component organization | [streamlit_standards.md](claude/streamlit_standards.md) |
| **Project Setup Prompt** | Reusable prompt for creating project-level reference files | [project_setup_prompt.md](claude/project_setup_prompt.md) |
| **Reference Guide** | Best practices for building and organising Claude Code reference documentation | [reference_guide.md](claude/reference_guide.md) |

## Using with Claude Code

### Remote File References

Claude Code can reference these standards directly from this repository using raw GitHub URLs.

**Recommended: Use version tags for stability**

```
https://raw.githubusercontent.com/mwtmurphy/claude-playbook/v1.0.0/claude/python_style.md
https://raw.githubusercontent.com/mwtmurphy/claude-playbook/v1.0.0/claude/sql_style.md
https://raw.githubusercontent.com/mwtmurphy/claude-playbook/v1.0.0/claude/architecture_patterns.md
```

**Alternative: Use main branch for latest updates**

```
https://raw.githubusercontent.com/mwtmurphy/claude-playbook/main/claude/python_style.md
https://raw.githubusercontent.com/mwtmurphy/claude-playbook/main/claude/sql_style.md
https://raw.githubusercontent.com/mwtmurphy/claude-playbook/main/claude/architecture_patterns.md
```

**Why version tags?** Version pinning provides stability (no unexpected changes) whilst allowing deliberate, controlled updates. See [USAGE.md](USAGE.md#reference-configuration-strategies) for detailed comparison of approaches.

### Example Configuration

For detailed integration instructions, configuration strategies, and examples, see [USAGE.md](USAGE.md).

### Benefits

- **Consistency**: All projects reference the same source of truth
- **Centralised Updates**: Update standards once, apply everywhere
- **Version Control**: Track changes and evolution of standards over time
- **Selective Application**: Choose which standards to reference per project

## Repository Structure

```
claude-playbook/
├── README.md                    # This file - project overview
├── USAGE.md                     # Detailed integration guide
└── claude/                      # Standards reference files
    ├── README.md                # Standards directory guide
    ├── python_style.md          # Python coding standards
    ├── sql_style.md             # SQL coding standards
    ├── architecture_patterns.md # Design principles
    ├── testing_standards.md     # Testing guidelines
    ├── documentation_standards.md
    ├── git_workflow.md
    ├── claude_workflow.md
    ├── error_handling.md
    ├── database_standards.md
    ├── performance_considerations.md
    ├── environment_setup.md
    ├── streamlit_standards.md
    ├── project_setup_prompt.md
    └── reference_guide.md       # Meta-guidance for creating standards
```

## Philosophy

These standards establish **strong preferences** for development practices while allowing justified deviations when:

- Project-specific requirements necessitate different approaches
- Framework or library conventions conflict with these standards
- Performance or technical constraints require alternative solutions
- A compelling technical argument supports the deviation

**Important**: Deviations should be documented with clear justification.

## Contributing

### Updating Standards

1. Standards are **living documents** but should be updated deliberately
2. All changes require human review and approval
3. Use conventional commits for changes
4. Document reasoning in commit messages
5. Consider impact across all projects using these standards

### Suggesting Changes

- Open an issue to propose changes or additions
- Provide rationale and examples
- Consider backwards compatibility
- Document migration path if breaking changes needed

## License

These standards are provided as reference guidelines. Use and adapt as needed for your projects.

---

**Last Updated**: 2025-10-30
**Maintained By**: mwtmurphy
**Status**: Active development

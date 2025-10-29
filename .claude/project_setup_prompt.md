# Project Setup Prompt

**Purpose**: Use this prompt when starting a new project to set up project-level reference files that extend the workspace standards.

**Usage**: Copy the prompt below and paste it to Claude Code when you're in a new project directory.

---

## Prompt to Copy

```
I have workspace-level development standards at /Users/mwtmurphy/projects/.claude/ that define:
- Python style (PEP 8, type hints, formatting)
- SQL style (BigQuery dialect, CTE patterns)
- Architecture patterns (framework-agnostic)
- Testing standards (pytest)
- Documentation standards (Google-style docstrings, Standard Readme)
- Git workflow (Conventional Commits)
- Error handling (exceptions, logging)
- Database standards (queries, schemas, migrations)
- Performance considerations
- Environment setup (pyenv, poetry)

**Task**: Analyze this project and create appropriate project-level reference files in `.claude/` that EXTEND (not duplicate) these workspace standards.

**Analysis Steps**:
1. Examine the project structure, files, and dependencies
2. Identify the tech stack (framework, database, APIs, etc.)
3. Understand the domain/business context
4. Determine what project-specific standards are needed

**Project-Level Standards to Create** (only if applicable):

Create `.claude/` files for:
- **Framework-specific patterns**: Django/FastAPI/Flask conventions, middleware, routing, serialization
- **API specifications**: Endpoint patterns, request/response formats, authentication
- **Database specifics**: PostgreSQL/MySQL/SQLite conventions, ORM patterns, query guidelines
- **Domain models**: Core business entities, relationships, validation rules
- **Integration requirements**: External services, message queues, caching strategies
- **Data processing**: ETL patterns, data validation, transformation rules
- **Deployment specifics**: Docker, CI/CD, environment configs

**Guidelines**:
- Do NOT duplicate workspace standards (style, general architecture, git workflow, etc.)
- Focus on project-specific patterns, business rules, and technical decisions
- Reference workspace standards where applicable
- Include examples specific to THIS project's domain
- Document any deviations from workspace standards with justification

**File Format**:
- Use markdown (.md)
- Follow same structure as workspace standards (status, scope, why, examples)
- Include cross-references to workspace files
- Keep files focused and single-purpose

After analysis, propose the project-level files to create and ask for approval before creating them.
```

---

## What Belongs at Each Level

### Workspace Level (Already Exists)
- Language-agnostic best practices
- Tool preferences (pyenv, poetry, pytest)
- Universal patterns (SOLID, DRY, testing)
- Code style (PEP 8, SQL formatting)
- Git workflow
- General architecture principles

### Project Level (To Be Created)
- Framework-specific implementations
- API contracts and schemas
- Database-specific conventions
- Domain/business logic patterns
- Integration specifications
- Deployment procedures
- Project-specific tooling

## Example: Django Project

For a Django project, you might create:
- `.claude/django_patterns.md` - Model patterns, view conventions, admin customization
- `.claude/api_specifications.md` - DRF serializers, viewsets, permission classes
- `.claude/business_rules.md` - Order processing, user roles, payment flows
- `.claude/integration_guide.md` - Stripe API, SendGrid, Redis caching

## Example: Data Pipeline Project

For a data pipeline, you might create:
- `.claude/data_models.md` - Schema definitions, validation rules
- `.claude/etl_patterns.md` - Extraction, transformation, loading conventions
- `.claude/airflow_dags.md` - DAG structure, task dependencies, scheduling
- `.claude/data_quality.md` - Validation rules, data contracts, monitoring

---

**Last Updated**: 2025-10-12

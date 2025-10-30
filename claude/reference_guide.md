# Claude Code Reference Documentation Guide

**Status**: Meta-guidance for playbook maintainers and standard creators
**Scope**: Best practices for creating and maintaining effective Claude Code reference documentation

## Overview

This guide explains best practices for creating Claude Code reference files based on patterns established in this playbook. Use this when contributing new standards to this repository or building similar reference documentation for your own projects.

**Why**: Consistent structure, clear reasoning, and effective examples make reference documentation immediately useful to Claude Code whilst remaining maintainable for humans.

## Documentation Structure

### File Organisation

All Claude Code reference files should follow these conventions:

**Naming:**
- Use `snake_case` with descriptive names: `python_style.md`, `testing_standards.md`
- Make names specific to the topic, not generic: `architecture_patterns.md` not `patterns.md`
- Use `.md` extension for markdown files
- Keep filenames concise (2-3 words maximum)

**Directory Structure:**
```
repository/
├── README.md                  # Project overview
├── USAGE.md                   # Integration instructions
└── claude/                    # All reference files here
    ├── README.md              # Directory guide
    ├── python_style.md        # Individual standards
    ├── sql_style.md
    ├── testing_standards.md
    └── ...
```

**Why**: Grouping all Claude Code references in a dedicated `claude/` directory keeps them organised and discoverable whilst separating them from project code or documentation.

### Standard File Format

Every reference file should follow this structure:

```markdown
# [Standard Name]

**Status**: Strong preference - deviations require justification and approval
**Scope**: [What this standard covers]

## Overview

[1-2 paragraph description of what this standard covers and why it matters]

**Why**: [Explicit benefit statement]

## [Major Topic Sections]

### [Subtopics]

**Why**: [Reasoning for this specific practice]

[Content with code examples]

```python
# Good: [What makes this good]
[example code]

# Bad: [What makes this bad]
[example code]
```

## Related Standards

- See `other_standard.md` for [related topic]

---

**Last Updated**: YYYY-MM-DD
**Status**: Strong preference - deviations require justification
```

**Required Elements:**

1. **Status Header**: Always "Strong preference - deviations require justification and approval"
2. **Scope Declaration**: One sentence stating what this standard covers
3. **Overview Section**: Context and high-level "why"
4. **"Why" Statements**: Throughout the document at file, section, and example levels
5. **Code Examples**: Show both good and bad patterns
6. **Related Standards**: Cross-references to other relevant files
7. **Footer**: Last Updated date and status reiteration

**Why**: This structure provides immediate context (status, scope), clear reasoning (why statements), and practical guidance (examples) in a consistent format Claude Code can reliably parse.

### Content Balance

**File Length Guidelines:**

- **Short (200 lines)**: Focused topics like `git_workflow.md`, single-concern standards
- **Medium (300-500 lines)**: Most standards, covering a coherent topic comprehensively
- **Long (600+ lines)**: Comprehensive guides covering multiple related subtopics

**When to Split vs Consolidate:**

**Split When:**
- File exceeds 600 lines
- Topics are independently reusable across projects
- Different teams/projects need different subsets
- Content covers multiple distinct domains (language, framework, process)

**Consolidate When:**
- Topics are always used together
- Cross-reference density is very high
- Splitting would create significant redundancy
- Related concerns that inform each other

**Examples from This Playbook:**

*Split:*
- `python_style.md` and `sql_style.md` - different languages
- `testing_standards.md` and `error_handling.md` - different concerns
- `git_workflow.md` and `claude_workflow.md` - different workflows

*Consolidated:*
- `documentation_standards.md` - docstrings, READMEs, comments, readability all together
- `architecture_patterns.md` - SOLID, layering, DI, directory structure all together

**Why**: Splitting by independent concerns allows selective referencing; consolidating related topics reduces navigation overhead and maintains context.

## Writing Effective Standards

### "Why" Statement Best Practices

Include "Why" statements at three levels:

**1. File-Level (in Overview):**
```markdown
## Overview

Good documentation makes code accessible, maintainable, and collaborative.

**Why**: Well-documented code reduces onboarding time, prevents misuse, and serves as
a reference for future maintenance.
```

**2. Section-Level (for major practices):**
```markdown
### Dependency Injection

Pass dependencies rather than creating them internally.

**Why**: Makes code testable, flexible, and loosely coupled.
```

**3. Example-Level (in code comments):**
```python
# Good: Dependency injection enables testing
class UserService:
    def __init__(self, repository: UserRepository) -> None:
        self.repository = repository  # Injected, not created
```

**Format:**
- Use bold "Why:" prefix
- Keep to 1-2 sentences maximum
- Focus on practical benefits, not theory
- Use active voice ("enables testing" not "testing is enabled")
- Be specific about the benefit

**Why**: Multi-level "why" statements ensure Claude Code understands both high-level rationale and specific implementation reasoning, enabling better code generation decisions.

### Code Example Patterns

**Always Show Good AND Bad:**

```python
# Good: Clear, specific exception handling
try:
    result = fetch_data()
except NetworkError as e:
    logger.error(f"Network failed: {e}")
    raise


# Bad: Generic exception hiding problems
try:
    result = fetch_data()
except Exception:  # Too broad!
    pass  # Silent failure
```

**Example Quality Guidelines:**

1. **Immediately Usable**: Good examples should be copy-pasteable
2. **Clearly Labelled**: Use "Good:", "Bad:", or "Avoid:" prefixes
3. **Explain in Comments**: Brief inline comments showing why
4. **Representative**: Show real-world scenarios, not toy examples
5. **Focused**: One concept per example
6. **Complete**: Include necessary imports and context

**Prefer Examples Over Prose:**

```markdown
# Less Effective (prose heavy)
Functions should have descriptive names that clearly indicate their purpose.
Avoid abbreviations and single-letter names except for well-known conventions
like 'i' for loop indices.

# More Effective (example heavy)
```python
# Good: Descriptive names
def calculate_monthly_payment(principal: float, rate: float) -> float:
    pass

# Bad: Unclear abbreviations
def calc_pmt(p: float, r: float) -> float:
    pass
```

**Why**: Code examples are concrete, unambiguous, and immediately actionable for Claude Code, whereas prose requires interpretation.

### Cross-Reference Patterns

**Related Standards Section (at file end):**

```markdown
## Related Standards

- See `python_style.md` for docstring formatting details
- See `testing_standards.md` for test documentation requirements
- See `git_workflow.md` for commit message conventions
```

**Inline References (when context matters):**

```markdown
### SQL Query Organisation

Keep SQL in separate files under `sql/queries/`. See `database_standards.md`
for complete file organisation structure and naming conventions.
```

**Cross-Reference Guidelines:**

1. Use backticks around filenames: `filename.md`
2. Be specific about what to find in the related file
3. Keep cross-references bidirectional where logical
4. Place general cross-references in "Related Standards" section at end
5. Use inline references when immediate context is needed

**Why**: Clear cross-referencing helps Claude Code discover related context and maintain consistency across interconnected standards.

## Reference Configuration Strategies

### Local Clone vs URL Reference

There are three approaches to referencing standards:

#### Approach A: Local Clone

**Implementation:**
```bash
# Clone playbook to local machine
git clone https://github.com/mwtmurphy/playbook.git ~/playbook

# Reference in project (claude/settings.json)
{
  "references": [
    "/Users/username/playbook/claude/python_style.md",
    "/Users/username/playbook/claude/testing_standards.md"
  ]
}
```

**Pros:**
- Offline access
- Version control (pin to specific commit)
- Can test changes locally before publishing
- Fast file access

**Cons:**
- Path differs per developer (`~/playbook` vs `/Users/alice/playbook`)
- Manual sync required (`git pull` to update)
- Risk of stale standards if developers forget to update
- Absolute paths are machine-specific

#### Approach B: URL Reference

**Implementation:**
```json
{
  "references": [
    "https://raw.githubusercontent.com/mwtmurphy/playbook/main/claude/python_style.md",
    "https://raw.githubusercontent.com/mwtmurphy/playbook/main/claude/testing_standards.md"
  ]
}
```

**Pros:**
- Always latest version automatically
- No local setup required
- Identical URLs work for entire team
- Portable across machines
- Single source of truth

**Cons:**
- Requires internet connection
- Standards can change unexpectedly
- No control over update timing
- Breaking changes can surprise users

**With Version Pinning:**
```json
{
  "references": [
    "https://raw.githubusercontent.com/mwtmurphy/playbook/v1.0.0/claude/python_style.md"
  ]
}
```

- Provides stability (no unexpected changes)
- Requires manual version updates
- Can transition between versions deliberately

#### Approach C: Hybrid (Recommended)

**Implementation:**
```json
{
  "references": [
    // Core standards - use URLs with version tags
    "https://raw.githubusercontent.com/mwtmurphy/playbook/v1.2.0/claude/python_style.md",
    "https://raw.githubusercontent.com/mwtmurphy/playbook/v1.2.0/claude/sql_style.md",
    "https://raw.githubusercontent.com/mwtmurphy/playbook/v1.2.0/claude/testing_standards.md",

    // Project-specific - local files
    "./claude/api_specifications.md",
    "./claude/business_rules.md"
  ]
}
```

**Strategy:**

**Reference via URL (with version tags):**
- Stable, rarely-changing standards (style guides, testing patterns)
- General architecture principles
- Tool configuration examples
- Workflow conventions

**Keep as local project files:**
- Framework-specific patterns
- API contracts and schemas
- Business rules and domain models
- Integration specifications

**Why**: This hybrid approach provides team consistency on core standards (via URLs) whilst supporting project-specific needs (via local files) and controlled updates (via version pinning).

### Configuration File Patterns

**Team Shared Configuration (committed to repo):**

```json
// claude/settings.json
{
  "references": [
    "https://raw.githubusercontent.com/mwtmurphy/playbook/v1.2.0/claude/python_style.md",
    "./claude/django_patterns.md"
  ]
}
```

**Developer Local Override (gitignored):**

```json
// .claude/settings.local.json
{
  "references": [
    "/Users/alice/playbook/claude/python_style.md"  // Override for testing
  ]
}
```

**.gitignore:**
```
.claude/settings.local.json
```

**Why**: Committed team config ensures consistency, whilst local overrides support development and testing without affecting the team.

### When to Use Which Approach

| Content Type | Recommended Approach | Reasoning |
|--------------|---------------------|-----------|
| Language style guides | URL (pinned version) | Rarely change, need team consistency |
| Architecture patterns | URL (version or main) | Evolve slowly, benefit from updates |
| Framework patterns | Project-local | Specific to this codebase |
| API specifications | Project-local | Unique to this service |
| Git workflow | URL (pinned version) | Universal across projects |
| Testing standards | URL (pinned version) | Benefit from improvements |
| Business rules | Project-local | Domain-specific |
| Performance guidelines | URL | General best practices |
| Environment setup | URL | Tool configuration |
| Database schemas | Project-local | Unique to this database |

## Version Management

### Semantic Versioning for Standards

Use semantic versioning for playbook releases:

**Version Format**: `vMAJOR.MINOR.PATCH`

- **MAJOR**: Breaking changes to standard structure or significant rewrites
  - Example: Changing required header format, removing entire sections
- **MINOR**: New standards added or significant enhancements
  - Example: Adding new `security_standards.md`, major expansion of existing standard
- **PATCH**: Fixes, clarifications, and minor improvements
  - Example: Fixing typos, adding examples, updating tool versions

**Creating Releases:**

```bash
# Tag a release
git tag -a v1.0.0 -m "Release v1.0.0: Initial playbook standards"
git push origin v1.0.0

# Reference in project
{
  "references": [
    "https://raw.githubusercontent.com/mwtmurphy/playbook/v1.0.0/claude/python_style.md"
  ]
}
```

**Why**: Semantic versioning allows teams to understand the impact of updates and make deliberate decisions about when to adopt new versions.

### Update Strategies

**Conservative (Recommended for Production):**
```json
{
  "references": [
    "https://raw.githubusercontent.com/mwtmurphy/playbook/v1.0.0/claude/python_style.md"
  ]
}
```
- Pin to specific version
- Review changelog before updating
- Test with new version before team-wide rollout
- Update deliberately across team

**Latest (For Personal Projects):**
```json
{
  "references": [
    "https://raw.githubusercontent.com/mwtmurphy/playbook/main/claude/python_style.md"
  ]
}
```
- Always get latest improvements
- Accept risk of breaking changes
- Good for staying current with best practices

**When to Update:**
- **PATCH versions**: Update freely (fixes and clarifications)
- **MINOR versions**: Review additions, update when relevant
- **MAJOR versions**: Careful review, plan migration if breaking changes

## Project-Specific Extensions

### What Belongs Where

**Workspace Level (This Playbook):**
- Framework-agnostic principles
- Language standards (Python, SQL)
- Tool preferences (pyenv, poetry, pytest)
- Universal patterns (SOLID, testing, git)
- General architecture principles

**Project Level (Individual Repositories):**
- Framework-specific implementation (Django, FastAPI, Flask)
- API contracts and endpoint specifications
- Domain business logic and rules
- Integration specifications (Stripe, AWS services)
- Database schema specifics
- Deployment procedures

**Why**: Workspace standards provide consistency across all projects; project standards capture unique requirements without duplicating general principles.

### Extension Patterns

**Do NOT Duplicate Workspace Standards:**

```markdown
# Bad: Repeating workspace content in project file
# Project Python Standards

Use Black for formatting with 88 character lines...
[Duplicates python_style.md]
```

**DO Reference and Extend:**

```markdown
# Project API Standards

**Status**: Strong preference - deviations require justification
**Scope**: API design for this e-commerce service

## Overview

This project follows workspace Python and architecture standards from the
[playbook](https://github.com/mwtmurphy/playbook). This document adds
API-specific conventions for our REST endpoints.

**Why**: Consistent API design improves developer experience and reduces
integration complexity.

## Authentication

All API endpoints require JWT authentication via Authorization header.

**Why**: Stateless authentication scales horizontally and integrates
with our existing OAuth provider.

```python
# Good: Standard auth header pattern
headers = {
    "Authorization": f"Bearer {jwt_token}",
    "Content-Type": "application/json"
}
```

## Endpoint Conventions

### URL Structure

```
/api/v1/{resource}/{id}/{sub-resource}
```

Examples:
- GET `/api/v1/products/123`
- POST `/api/v1/orders`
- GET `/api/v1/orders/456/items`

### Response Format

All responses use this envelope:

```json
{
  "status": "success" | "error",
  "data": { ... },
  "meta": { "timestamp": "...", "version": "v1" }
}
```

## Related Standards

- See [python_style.md](https://github.com/mwtmurphy/playbook/blob/main/claude/python_style.md) for code formatting
- See [error_handling.md](https://github.com/mwtmurphy/playbook/blob/main/claude/error_handling.md) for exception patterns
- See `database_standards.md` locally for schema details

---

**Last Updated**: 2025-10-30
**Status**: Project-specific extension
```

**Why**: Referencing workspace standards avoids duplication whilst project files focus on unique requirements, creating a clear hierarchy of standards.

## Maintenance Best Practices

### Updating Standards

**When to Update:**
- Tool versions change (e.g., new Python release, updated linter)
- Best practices evolve (new patterns emerge)
- Mistakes or omissions discovered
- User feedback identifies unclear sections

**How to Update:**
1. Make changes in feature branch
2. Update "Last Updated" date in footer
3. Use Conventional Commits: `docs(python): update Black version to 24.10.0`
4. Document reasoning in commit message
5. Consider impact across projects using these standards
6. Create PR for review
7. Tag new version if appropriate

**Commit Message Pattern:**
```
docs(scope): brief description

Detailed explanation of why this change matters and what projects
might be affected.

Breaking Change: [if applicable]
- Describe what breaks
- Provide migration path
```

### Quality Checklist

Before merging new or updated standards:

- [ ] Follows standard file format (Status/Scope/Overview/Why/Footer)
- [ ] Includes "Why" statements at appropriate levels
- [ ] Provides Good and Bad code examples
- [ ] Uses British English consistently
- [ ] Cross-references related standards
- [ ] "Last Updated" date is current
- [ ] Tested by using in Claude Code session
- [ ] Commit message explains reasoning
- [ ] No duplication of existing standards

## Common Patterns Library

### Pattern: Language-Specific Standards

Split by programming language when syntax differs significantly:
- `python_style.md`, `sql_style.md`, `typescript_style.md`

### Pattern: Concern-Based Standards

Split by technical concern when they're independently applicable:
- `testing_standards.md`, `error_handling.md`, `performance_considerations.md`

### Pattern: Tool-Specific Standards

Create when tool usage is substantial enough to warrant dedicated guidance:
- `streamlit_standards.md`, `pytest_standards.md`

### Pattern: Process Standards

Workflow and development process guidance:
- `git_workflow.md`, `claude_workflow.md`

### Pattern: Cross-Cutting Concerns

Consolidate when concerns inform each other heavily:
- `documentation_standards.md` (docstrings + READMEs + comments + readability)
- `architecture_patterns.md` (SOLID + layering + DI + structure)

## Related Documents

- See `project_setup_prompt.md` for creating project-level reference files
- See `claude_workflow.md` for using references during development
- See `USAGE.md` in repository root for integration instructions
- See `README.md` for playbook overview and philosophy

---

**Last Updated**: 2025-10-30
**Status**: Meta-guidance for maintainers

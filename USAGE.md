# Usage Guide: Integrating Development Standards

This guide explains how to integrate the playbook standards into your projects using Claude Code.

## Table of Contents

- [Quick Start](#quick-start)
- [Claude Code Configuration](#claude-code-configuration)
- [Integration Patterns](#integration-patterns)
- [Example Workflows](#example-workflows)
- [Best Practices](#best-practices)
- [Reference Configuration Strategies](#reference-configuration-strategies)
- [Troubleshooting](#troubleshooting)
- [Advanced Usage](#advanced-usage)
- [Support](#support)

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
https://raw.githubusercontent.com/mwtmurphy/claude-playbook/main/claude/python_style.md
https://raw.githubusercontent.com/mwtmurphy/claude-playbook/main/claude/sql_style.md
https://raw.githubusercontent.com/mwtmurphy/claude-playbook/main/claude/testing_standards.md
```

### 3. Configure Claude Code

Add these URLs to your project's Claude Code configuration or provide them in your context when starting a session.

## Claude Code Configuration

### Method 1: Project-Level Settings

Create or update `claude/settings.json` in your project:

```json
{
  "references": [
    "https://raw.githubusercontent.com/mwtmurphy/claude-playbook/main/claude/python_style.md",
    "https://raw.githubusercontent.com/mwtmurphy/claude-playbook/main/claude/testing_standards.md",
    "https://raw.githubusercontent.com/mwtmurphy/claude-playbook/main/claude/architecture_patterns.md"
  ]
}
```

### Method 2: Manual Context

When starting a Claude Code session, provide standards in your initial prompt:

```
Please reference these standards while working:
- https://raw.githubusercontent.com/mwtmurphy/claude-playbook/main/claude/python_style.md
- https://raw.githubusercontent.com/mwtmurphy/claude-playbook/main/claude/sql_style.md

Help me implement a user authentication system following these standards.
```

### Method 3: Local Copy

For projects that need customization:

```bash
# Copy standards to your project
mkdir -p claude
cd claude
curl -O https://raw.githubusercontent.com/mwtmurphy/claude-playbook/main/claude/python_style.md
curl -O https://raw.githubusercontent.com/mwtmurphy/claude-playbook/main/claude/testing_standards.md
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
    "https://raw.githubusercontent.com/mwtmurphy/claude-playbook/main/claude/python_style.md",
    "https://raw.githubusercontent.com/mwtmurphy/claude-playbook/main/claude/testing_standards.md",
    "https://raw.githubusercontent.com/mwtmurphy/claude-playbook/main/claude/documentation_standards.md",
    "https://raw.githubusercontent.com/mwtmurphy/claude-playbook/main/claude/architecture_patterns.md",
    "https://raw.githubusercontent.com/mwtmurphy/claude-playbook/main/claude/git_workflow.md"
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
    "https://raw.githubusercontent.com/mwtmurphy/claude-playbook/main/claude/python_style.md"
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
    "https://raw.githubusercontent.com/mwtmurphy/claude-playbook/main/claude/python_style.md",
    "https://raw.githubusercontent.com/mwtmurphy/claude-playbook/main/claude/streamlit_standards.md"
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
git remote add upstream https://github.com/mwtmurphy/claude-playbook.git
git fetch upstream
git diff HEAD:python_style.md upstream/main:claude/python_style.md
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

This project follows the [mwtmurphy/claude-playbook](https://github.com/mwtmurphy/claude-playbook) standards:

- Python Style: [Link]
- Testing: [Link]
- Git Workflow: [Link]

See claude/settings.json for full list of referenced standards.
```

## Reference Configuration Strategies

### Choosing Between Local and URL References

There are three main approaches to referencing standards, each with different trade-offs:

#### Option 1: URL References (Recommended for Teams)

**Use remote GitHub URLs directly:**

```json
{
  "references": [
    "https://raw.githubusercontent.com/mwtmurphy/claude-playbook/main/claude/python_style.md",
    "https://raw.githubusercontent.com/mwtmurphy/claude-playbook/main/claude/testing_standards.md"
  ]
}
```

**Advantages:**
- ✓ Works identically for all team members
- ✓ No local setup required
- ✓ Automatically gets latest updates
- ✓ Portable across machines
- ✓ Easy to share configurations

**Disadvantages:**
- ✗ Requires internet connection
- ✗ Standards can change unexpectedly
- ✗ No control over update timing

**Best for**: Team projects, distributed teams, CI/CD pipelines

#### Option 2: URL with Version Pinning (Recommended for Production)

**Reference specific versions using tags:**

```json
{
  "references": [
    "https://raw.githubusercontent.com/mwtmurphy/claude-playbook/v1.0.0/claude/python_style.md",
    "https://raw.githubusercontent.com/mwtmurphy/claude-playbook/v1.0.0/claude/testing_standards.md"
  ]
}
```

**Advantages:**
- ✓ All benefits of URL references
- ✓ Standards frozen at specific version (no surprises)
- ✓ Deliberate, controlled updates
- ✓ Can review changelogs before updating

**Disadvantages:**
- ✗ Requires internet connection
- ✗ Manual version updates needed

**Best for**: Production projects, teams wanting stability, critical systems

**Updating to new versions:**
```bash
# Review changes
git diff v1.0.0..v1.1.0 claude/python_style.md

# Update settings.json
# Change v1.0.0 → v1.1.0
# Test thoroughly
# Rollout to team
```

#### Option 3: Local Clone

**Clone playbook locally and reference local paths:**

```bash
# Clone playbook
git clone https://github.com/mwtmurphy/claude-playbook.git ~/claude-playbook

# Reference in settings.json
{
  "references": [
    "/Users/username/claude-playbook/claude/python_style.md",
    "/Users/username/claude-playbook/claude/testing_standards.md"
  ]
}
```

**Advantages:**
- ✓ Offline access
- ✓ Fast file access (no network latency)
- ✓ Full version control (git history)
- ✓ Can test changes locally

**Disadvantages:**
- ✗ Path differs per developer
- ✗ Manual sync required (`git pull`)
- ✗ Risk of stale standards
- ✗ Team coordination needed

**Best for**: Contributors to playbook, offline-required environments, testing standards changes

#### Option 4: Hybrid Approach (Recommended for Flexibility)

**Combine URL and local references:**

```json
{
  "references": [
    // Core standards - URL with version pinning
    "https://raw.githubusercontent.com/mwtmurphy/claude-playbook/v1.0.0/claude/python_style.md",
    "https://raw.githubusercontent.com/mwtmurphy/claude-playbook/v1.0.0/claude/sql_style.md",
    "https://raw.githubusercontent.com/mwtmurphy/claude-playbook/v1.0.0/claude/testing_standards.md",

    // Project-specific - local files
    "./claude/api_specifications.md",
    "./claude/business_rules.md",
    "./claude/django_patterns.md"
  ]
}
```

**When to use what:**

| Content Type | Use URL | Use Local |
|--------------|---------|-----------|
| Language style guides | ✓ | |
| Architecture patterns | ✓ | |
| Testing standards | ✓ | |
| Git workflow | ✓ | |
| Framework patterns | | ✓ |
| API specifications | | ✓ |
| Business rules | | ✓ |
| Domain models | | ✓ |

**Why hybrid**: Core standards benefit from team consistency and updates; project-specific content belongs in the project repository.

### Configuration File Patterns

#### Team Configuration (Committed)

**File**: `claude/settings.json` (committed to git)

```json
{
  "references": [
    "https://raw.githubusercontent.com/mwtmurphy/claude-playbook/v1.2.0/claude/python_style.md",
    "./claude/api_specifications.md"
  ]
}
```

**Purpose**: Shared team configuration ensuring consistency

#### Developer Overrides (Gitignored)

**File**: `.claude/settings.local.json` (gitignored)

```json
{
  "references": [
    "/Users/alice/dev/claude-playbook/claude/python_style.md"
  ]
}
```

**Purpose**: Local overrides for testing or development without affecting team

**Add to .gitignore:**
```bash
echo ".claude/settings.local.json" >> .gitignore
```

**Use case**: When contributing to playbook or testing standards changes before team adoption.

### Migration Guide: Local to URL References

If you're currently using local clones, here's how to migrate:

**Current setup (local):**
```json
{
  "references": [
    "/Users/username/claude-playbook/claude/python_style.md"
  ]
}
```

**Step 1: Find latest playbook version**
```bash
# Check latest tag
git ls-remote --tags https://github.com/mwtmurphy/claude-playbook.git
```

**Step 2: Update settings.json**
```json
{
  "references": [
    "https://raw.githubusercontent.com/mwtmurphy/claude-playbook/v1.0.0/claude/python_style.md"
  ]
}
```

**Step 3: Test**
```bash
# Start Claude Code session and verify standards are loading
claude-code
```

**Step 4: Commit and share with team**
```bash
git add claude/settings.json
git commit -m "feat(config): migrate to URL-based standard references"
git push
```

**Step 5: Clean up local clone (optional)**
```bash
# No longer needed unless contributing to playbook
rm -rf ~/claude-playbook
```

### Version Update Workflow

**When new playbook versions are released:**

**1. Review changelog:**
```bash
# View changes between versions
git diff v1.0.0..v1.1.0 --name-only
git diff v1.0.0..v1.1.0 claude/python_style.md
```

**2. Test in development:**
```json
// .claude/settings.local.json
{
  "references": [
    "https://raw.githubusercontent.com/mwtmurphy/claude-playbook/v1.1.0/claude/python_style.md"
  ]
}
```

**3. Validate with team:**
- Test on sample code
- Review any breaking changes
- Check for conflicts with project-specific patterns

**4. Update team configuration:**
```json
// claude/settings.json
{
  "references": [
    "https://raw.githubusercontent.com/mwtmurphy/claude-playbook/v1.1.0/claude/python_style.md"  // Updated
  ]
}
```

**5. Communicate:**
```markdown
PR #123: Update playbook standards to v1.1.0

Changes in this version:
- Updated Black version to 24.10.0
- Added new error handling patterns
- Fixed British English inconsistencies

See full changelog: https://github.com/mwtmurphy/claude-playbook/releases/tag/v1.1.0
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
   https://raw.githubusercontent.com/mwtmurphy/claude-playbook/<commit-sha>/claude/python_style.md
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
          curl -O https://raw.githubusercontent.com/mwtmurphy/claude-playbook/main/claude/python_style.md
          # Run Claude Code to review PR against standards
          claude-code review --standards python_style.md
```

### Multi-Language Projects

Reference standards from multiple playbooks:

```json
{
  "references": [
    "https://raw.githubusercontent.com/mwtmurphy/claude-playbook/main/claude/python_style.md",
    "https://raw.githubusercontent.com/another-org/js-playbook/main/javascript_style.md"
  ]
}
```

## Support

- **Issues**: [GitHub Issues](https://github.com/mwtmurphy/claude-playbook/issues)
- **Documentation**: See `claude/reference_guide.md` for creating standards
- **Updates**: Watch the repository for changes

---

**Last Updated**: 2025-10-30
**Repository**: [mwtmurphy/claude-playbook](https://github.com/mwtmurphy/claude-playbook)

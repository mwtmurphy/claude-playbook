# Claude Code Workflow Standards

**Status**: Strong preference - deviations require justification and approval
**Scope**: Claude Code-specific workflow and planning practices
**Last Updated**: 2025-10-31

---

## Overview

This document defines workflow standards for Claude Code when working on software projects. These practices ensure consistent planning, implementation tracking, and documentation throughout development sessions.

**Why**: Structured workflows create accountability, maintain implementation context, and provide clear documentation of decision-making processes.

## Plan Mode Workflow

### When to Use Plan Mode

Claude Code should use plan mode for:

1. **Complex multi-step tasks** requiring 3+ distinct operations
2. **Ambiguous requirements** needing clarification before implementation
3. **Architectural decisions** with multiple valid approaches
4. **System-wide changes** affecting multiple files or modules
5. **User-requested planning** when explicitly asked to plan first

### When to Skip Plan Mode

Skip plan mode for:

1. **Simple single-file edits** with clear requirements
2. **Quick bug fixes** with obvious solutions
3. **Trivial changes** like formatting or simple refactoring
4. **Conversational queries** not involving code changes
5. **File reading/exploration** tasks

## Plan Documentation Requirements

### Mandatory Plan File Creation

When a plan is approved by the user, Claude Code **MUST**:

1. **Create a temporary plan file** in the project root directory
2. **Use naming convention**: `.claude-plan-{timestamp}.md`
   - Example: `.claude-plan-2025-10-19T14-30-00.md`
3. **Write the approved plan** to the file in structured markdown format
4. **Reference the plan file** during implementation for guidance
5. **Delete the plan file** after successful completion and verification

### Plan File Format

```markdown
# Plan: [Brief Description]

**Created**: [ISO timestamp]
**Status**: In Progress | Completed
**Estimated Complexity**: Low | Medium | High

## Objective

[1-2 sentence summary of what this plan accomplishes]

## Tasks

- [ ] Task 1: [Description]
  - Files: `path/to/file.py:lines`
  - Rationale: [Why this change is needed]

- [ ] Task 2: [Description]
  - Files: `path/to/other.py`
  - Rationale: [Why this change is needed]

[Additional tasks...]

## Implementation Notes

- [Important considerations]
- [Dependencies or prerequisites]
- [Potential risks or edge cases]

## Verification Steps

- [ ] All tests pass
- [ ] Linting passes
- [ ] Manual verification: [specific checks]
- [ ] Documentation updated

## Completion Checklist

- [ ] All tasks completed
- [ ] Verification steps passed
- [ ] Plan file deleted
```

### Example Plan File

```markdown
# Plan: Upgrade Dev Tools for Python 3.13 Support

**Created**: 2025-10-19T15:30:00Z
**Status**: In Progress
**Estimated Complexity**: Medium

## Objective

Upgrade Black and Ruff to versions supporting Python 3.13 target,
update configuration files, and verify all tests pass.

## Tasks

- [x] Update Black version to ^24.10.0 in pyproject.toml
  - Files: `pyproject.toml:25`
  - Rationale: Black 24.10.0 adds py313 support

- [x] Update Ruff version to ^0.14.0 in pyproject.toml
  - Files: `pyproject.toml:26`
  - Rationale: Ruff 0.14.0 has stable py313 support

- [x] Run poetry lock to update dependencies
  - Rationale: Regenerate lock file with new versions

- [x] Update tool target configurations
  - Files: `pyproject.toml:38,43,48`
  - Rationale: Tools need to target py313 for compatibility

- [ ] Verify linting and tests pass
  - Rationale: Ensure no breaking changes

## Implementation Notes

- mypy 1.18.2 already supports Python 3.13
- Ruff config format changed to use lint.* sections
- All 188 unit tests must pass

## Verification Steps

- [ ] poetry run black --check passes
- [ ] poetry run ruff check passes
- [ ] poetry run pytest passes (all 188 tests)

## Completion Checklist

- [ ] All tasks completed
- [ ] All verification steps passed
- [ ] Changes committed to git
- [ ] Plan file deleted
```

## Plan File Lifecycle

### 1. Creation (After Plan Approval)

```python
# Pseudocode for Claude Code workflow
if plan_approved:
    timestamp = datetime.now().isoformat(timespec='seconds').replace(':', '-')
    plan_file = f".claude-plan-{timestamp}.md"
    write_plan_to_file(plan_file, approved_plan_content)
    begin_implementation()
```

### 2. Reference During Implementation

- Consult plan file for next steps
- Update checkboxes as tasks complete
- Add notes about deviations or issues encountered
- Keep file as single source of truth for session

### 3. Deletion (After Completion)

Delete plan file when ALL of these conditions are met:

- ✅ All tasks marked complete
- ✅ All verification steps passed
- ✅ Changes committed to version control (if applicable)
- ✅ User confirms satisfaction with result

**Never delete plan files** if:
- Tasks remain incomplete
- Verification fails
- Errors occurred and not resolved
- User has not confirmed completion

## Task Tracking During Implementation

### Using TodoWrite Tool

In addition to the plan file, use the TodoWrite tool for:

1. **Real-time progress tracking** visible to user
2. **Breaking down complex tasks** into smaller steps
3. **Managing task state** (pending, in_progress, completed)
4. **Showing active work** to user

**Relationship**: Plan file is the blueprint; TodoWrite is the live progress tracker.

### Task State Management

```markdown
# Plan file tasks use checkboxes:
- [ ] Pending task
- [x] Completed task

# TodoWrite uses explicit states:
{
  "content": "Task description",
  "status": "pending" | "in_progress" | "completed",
  "activeForm": "Present continuous form"
}
```

### Best Practices

1. **Keep plan file and TodoWrite synchronized**
2. **Update both** when completing tasks
3. **Plan file** = permanent record
4. **TodoWrite** = live session status

## Handling Plan Changes

### When Requirements Change Mid-Implementation

If user requests changes to an approved plan:

1. **Update the plan file** with new tasks or modified approach
2. **Add note** documenting the change and reasoning
3. **Update TodoWrite** to reflect new task list
4. **Continue implementation** with revised plan

Example addition to plan file:

```markdown
## Plan Updates

### 2025-10-19T16:45:00Z - Added Task
User requested additional feature: [description]
Added new task: [task details]
```

### When Plan Fails or Needs Abandonment

If implementation reveals plan is not viable:

1. **Do NOT delete plan file**
2. **Document failure reason** in plan file
3. **Present new plan** to user for approval
4. **Create new plan file** when new plan approved
5. **Keep old plan file** for reference (rename with `-failed` suffix)

## File Naming Conventions

### Temporary Files

```bash
# Plan files (delete after completion)
.claude-plan-2025-10-19T14-30-00.md

# Failed plan files (keep for reference)
.claude-plan-2025-10-19T14-30-00-failed.md

# Alternative plan files (multiple approaches)
.claude-plan-2025-10-19T14-30-00-option-a.md
.claude-plan-2025-10-19T14-30-00-option-b.md
```

### Gitignore Recommendation

Add to project `.gitignore`:

```gitignore
# Claude Code temporary plan files
.claude-plan-*.md
```

**Why**: Plan files are session-specific and should not be committed to version control.

## Integration with Git Workflow

### Pre-Commit Plan Verification

Before committing changes:

1. **Review plan file** to ensure all tasks completed
2. **Verify completion checklist** is fully checked
3. **Ensure commit message** reflects plan objective
4. **Delete plan file** after successful commit

### PR Description from Plan

When creating pull requests:

1. **Use plan objective** as PR summary
2. **Reference plan tasks** in PR description
3. **Include verification steps** from plan
4. **Document any deviations** from original plan

## Error Handling During Planned Work

### When Implementation Fails

If an error occurs during plan execution:

1. **Do NOT mark task as completed**
2. **Add error details** to plan file
3. **Update task status** to blocked or problematic
4. **Create new task** for error resolution
5. **Keep plan file** until issue resolved

Example:

```markdown
## Tasks

- [x] Update Black version
- [⚠️] Run poetry lock
  - **Error**: Dependency conflict with httpx
  - **Resolution needed**: Investigate httpx version compatibility
  - **Blocker for**: All subsequent tasks

- [ ] New Task: Resolve httpx dependency conflict
  - Files: `pyproject.toml`
  - Rationale: Unblock poetry lock step
```

## Communication Standards

### Notifying User of Plan Progress

Periodically inform user of progress:

- After completing major milestones
- When encountering unexpected issues
- When plan is 50% complete
- When plan is fully complete

### Plan Completion Notification

```markdown
✅ Plan completed successfully!

**Summary**:
- [X] tasks completed
- All tests passing
- No errors encountered

**Files modified**:
- `file1.py`
- `file2.py`

**Next steps**: Plan file will be deleted.
```

## Related Standards

- See `git_workflow.md` for commit and PR practices
- See `documentation_standards.md` for documentation requirements
- See `python_testing_standards.md` for verification standards

---

**Last Updated**: 2025-10-31
**Status**: Strong preference - deviations require justification

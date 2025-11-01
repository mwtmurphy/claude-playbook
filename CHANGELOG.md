# Changelog

All notable changes to the claude-playbook standards will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/).

## [Unreleased]

### Added

- **New Standards Files**:
  - `metricflow_dbt_standards.md` - Comprehensive dbt-core with MetricFlow semantic layer patterns covering semantic model design, metric development, query optimization, and Python integration

- **Streamlit Standards Enhancements**:
  - MetricFlow Integration section - MetricFlow-first data loading patterns for Streamlit dashboards
  - BigQuery Integration section - Best practices for BigQuery client setup, query parameterization, and cost optimization
  - Updated Table of Contents to include new sections

- **Data Visualization Standards Enhancements**:
  - Streamlit Color Management section - Centralized `chart_colors.py` module pattern with accessibility guidelines, usage examples, and WCAG AA compliance testing

### Changed

- **README.md**: Added MetricFlow + dbt Standards to Quick Reference table
- **Cross-References**: Streamlit Standards now references MetricFlow + dbt Standards; Data Visualization Standards now references Streamlit Standards for implementation patterns

## [2.0.0] - 2025-10-31

### Added

- **New Standards Files**:
  - `logging_standards.md` - Comprehensive logging standards including structured logging, log levels, what to log/not log, and performance considerations
  - `security_standards.md` - Security best practices covering authentication, authorization, secrets management, input validation, HTTPS/TLS, and vulnerability scanning
  - `api_design_standards.md` - REST API design standards including URL structure, HTTP methods, status codes, versioning, pagination, and error responses

### Changed

- **Metadata Standardization**: Updated all 22 standards files with consistent metadata format:
  - Added "Last Updated: 2025-10-31" to all file headers
  - Added separator line (---) after metadata block for visual consistency
  - Removed duplicate metadata footers where present

- **Enhanced Cross-References**: Added comprehensive "Related Standards" sections to all core files:
  - `python_style.md` now references documentation, git, SQL, and testing standards
  - `sql_style.md` now references database, python, documentation, and testing standards
  - `testing_standards.md` now references streamlit and visualization testing standards
  - `git_workflow.md` now references claude_workflow.md for CI integration
  - `architecture_patterns.md` now references performance considerations
  - `error_handling.md` now references new logging_standards.md
  - `database_standards.md` enhanced with testing and performance references
  - `performance_considerations.md` now references python_style for performant patterns
  - All new files include appropriate cross-references

- **Date Standardization**: Updated all "Last Updated" dates from various dates (2025-10-11, 2025-10-18, 2025-10-19, 2025-10-21) to 2025-10-31 for consistency

- **README.md Updates**:
  - Added three new standards files to reference table
  - Updated "Last Updated" to 2025-10-31
  - All 22 standards files now documented

- **Cross-Reference Fixes**: Updated error_handling.md and security_standards.md to reference newly created files instead of "(when created)" placeholders

### Fixed

- Removed inconsistent metadata footers from multiple files (sql_style.md, testing_standards.md, git_workflow.md, architecture_patterns.md, error_handling.md, database_standards.md, performance_considerations.md, environment_setup.md, documentation_standards.md, claude_workflow.md)
- Standardized metadata placement across all files for easier parsing by Claude Code

## [1.0.0] - 2025-10-31

### Added

- **Initial Migration**: Migrated 7 core standards from workspace to repository:
  - `data_visualization_standards.md` - Plotly standards, color palettes, accessibility guidelines
  - `documentation_for_claude.md` - AI-optimized documentation patterns
  - `documentation_for_employees.md` - Human-readable documentation standards
  - `interactive_visualization_testing.md` - Testing standards for D3.js and interactive visualizations
  - `user_journey_diagrams.md` - Customer journey mapping with Mermaid diagrams
  - `templates/d3_visualization_template.html` - Reusable D3.js visualization template
  - British English guidance added to `documentation_standards.md`

- **Core Standards Files** (pre-existing):
  - `python_style.md`
  - `sql_style.md`
  - `architecture_patterns.md`
  - `testing_standards.md`
  - `documentation_standards.md`
  - `git_workflow.md`
  - `error_handling.md`
  - `database_standards.md`
  - `performance_considerations.md`
  - `environment_setup.md`
  - `claude_workflow.md`
  - `streamlit_standards.md`
  - `project_setup_prompt.md`
  - `reference_guide.md`

### Changed

- Repository renamed from original name to `claude-playbook`
- Enhanced README.md with comprehensive file descriptions
- Updated `project_setup_prompt.md` with current repository URL
- Updated all GitHub URLs from specific tree references to repository root

### Documentation

- Created `IMPROVEMENT_CHECKLIST.md` tracking systematic playbook enhancements
- All standards now follow consistent format: Status, Scope, Last Updated, Overview, sections, Related Standards
- Cross-reference network established between related standards files

## Git Commits (2.0.0)

This release includes the following commits on branch `feat/migrate-reference-files`:

1. `49d72a6` - feat: migrate reference files from workspace .claude directory
2. `ecb47b7` - docs: update claude/README.md with new standards files
3. `954decc` - docs: update project_setup_prompt.md with current repo URL
4. `54dce8f` - docs: standardize python_style.md metadata and create improvement checklist
5. `[current]` - docs: complete metadata standardization and add three new standards files

## Notes

- **Breaking Changes**: None. All changes are additive or metadata improvements.
- **Migration Path**: Projects using remote references will automatically receive updates when pointing to `main` branch. Projects using specific commit SHAs or tags will need to update their references manually.
- **Compatibility**: All standards remain compatible with existing projects.
- **URL Stability**: All file paths remain unchanged to preserve GitHub URLs.

---

**Maintained By**: Mitchell Murphy
**Repository**: [mwtmurphy/claude-playbook](https://github.com/mwtmurphy/claude-playbook)

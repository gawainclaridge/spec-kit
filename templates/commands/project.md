---
description: Create or manage a project definition for multi-feature projects with shared context.
handoffs:
  - label: Create Specification
    agent: speckit.specify
    prompt: Create a feature specification for this project. --project {PROJECT_NAME} I want to build...
  - label: Create Constitution
    agent: speckit.constitution
    prompt: Create project constitution with principles for...
scripts:
  sh: scripts/bash/create-project.sh --json "{ARGS}"
  ps: scripts/powershell/create-project.ps1 -Json "{ARGS}"
---

## User Input

```text
$ARGUMENTS
```

You **MUST** consider the user input before proceeding (if not empty).

### Workflow Context (Unifyr Process)

This is **Stage 1 (Specification)** of the Unifyr process:
- **Team**: Product
- **Purpose**: Create shared context for multi-feature projects
- **Prerequisites**: None
- **Output**: project.md with out-of-scope, shared constraints, features table
- **Next step**: `/speckit.specify --project <name>` to add features

## Outline

This command creates or manages a project.md file for multi-feature projects. A project groups related specifications that share common context, constraints, and out-of-scope items.

### When to Use Projects

Use projects when:
- Multiple features share significant context (target users, constraints, tech decisions)
- Features are part of a larger initiative (e.g., "Taskify Platform")
- You want to avoid repeating shared information in each spec.md

### Argument Parsing

The first argument is the project name:
- `/speckit.project taskify` - Create or manage the "taskify" project
- `/speckit.project taskify --list` - List all specs in the project
- `/speckit.project taskify --add-spec` - Add current spec to the project

### Execution Flow

1. **Parse Arguments**:
   - Extract project name from first argument
   - Check for optional flags: `--list`, `--add-spec`

2. **Check for existing project**:
   - Look for branch `project-<name>`
   - Look for directory `specs/project-<name>/`
   - Look for `specs/project-<name>/project.md`

3. **If project exists and no flags**:
   - Display project summary (name, status, features count)
   - Show Features table from project.md
   - Offer options: add spec, update project, view details

4. **If project exists with `--list` flag**:
   - Read project.md Features table
   - Display all linked specifications with their status

5. **If project exists with `--add-spec` flag**:
   - Get current feature from git branch or SPECIFY_FEATURE
   - Add entry to Features table in project.md
   - Update spec.md to reference the project

6. **If project does NOT exist**:
   a. Run `{SCRIPT}` to:
      - Create branch `project-<name>` from main
      - Create directory `specs/project-<name>/`
      - Copy `templates/project-template.md` to `specs/project-<name>/project.md`

   b. Gather project information from user:
      - Project overview/description
      - Target users (shared across features)
      - Out-of-scope items (project-level exclusions)
      - Shared constraints

   c. Fill project.md template:
      - Replace [PROJECT NAME] with provided name
      - Fill Project Overview section
      - Fill Target Users section
      - Fill Out of Scope section
      - Fill Shared Constraints section
      - Initialize Features table (empty)
      - Initialize Jira Integration placeholders
      - Initialize Changelog with version 1.0

   d. Write completed project.md to `specs/project-<name>/project.md`

7. **Report**:
   - Project branch name
   - Project.md path
   - Next steps:
     - "Run `/speckit.specify --project <name> <description>` to add features"
     - "Run `/speckit.constitution` to create project constitution"

## Project Structure

When a project is created:

```text
specs/
└── project-<name>/
    ├── project.md           # Project definition (shared context)
    ├── 001-feature-a/       # First feature
    │   ├── spec.md
    │   ├── plan.md
    │   └── tasks.md
    ├── 002-feature-b/       # Second feature
    │   ├── spec.md
    │   ├── plan.md
    │   └── tasks.md
    └── ...
```

## Guidelines

### Project vs Spec Scope

| Aspect | Project (project.md) | Feature (spec.md) |
|--------|---------------------|-------------------|
| Scope exclusions | Out of Scope (project-wide) | Non-Goals (feature-specific) |
| Users | Shared target users | Feature-specific user journeys |
| Constraints | Shared constraints | Feature-specific requirements |
| Tech decisions | Shared stack (if decided) | Feature-specific implementation |

### Naming Conventions

- Project branch: `project-<kebab-case-name>` (e.g., `project-taskify`)
- Project directory: `specs/project-<name>/`
- Feature directories within project: `specs/project-<name>/###-<feature>/`

### Jira Integration

Projects map to Jira Epics:
- Project → Epic
- Feature specs → Stories under the Epic
- Tasks → Sub-tasks under Stories

Update Jira Integration section with:
- Jira Project key
- Epic key (once created)

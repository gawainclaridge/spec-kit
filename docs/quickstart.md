# Quick Start Guide

This guide will help you get started with Spec-Driven Development using Unifyr Spec Kit.

> [!NOTE]
> **Fork of Spec Kit** — Unifyr Spec Kit is a fork of [GitHub's Spec Kit](https://github.com/github/spec-kit), tailored to Unifyr's agile scrum processes. Key additions include a 4-stage process with team ownership, multi-feature project support, Jira integration with Fibonacci story points, per-story task files for parallel sprint work, constitution enforcement before planning, and mid-flight change guidance. See the [README](../README.md) for a full comparison table.

> [!NOTE]
> All automation scripts now provide both Bash (`.sh`) and PowerShell (`.ps1`) variants. The `specify` CLI auto-selects based on OS unless you pass `--script sh|ps`.

## The 4-Stage Process

Unifyr Spec Kit follows a structured 4-stage workflow aligned with team collaboration:

| Stage | Focus | Key Activities |
|-------|-------|----------------|
| **Stage 1: Specification** | Product | Create project.md (optional for multi-feature), draft spec.md, optionally draft constitution.md |
| **Stage 2: Review** | Product/Engineering/QA | `/speckit.clarify`, review spec, add acceptance criteria, sign off |
| **Stage 3: Planning** | Engineering | Finalize constitution, create plan.md, define testing scenarios |
| **Stage 4: Tasks** | Engineering | Generate tasks.md, link to issue tracker, implement |

> [!TIP]
> **Context Awareness**: Unifyr Spec Kit commands automatically detect the active feature based on your current Git branch (e.g., `001-feature-name`). For single-feature workflows, switch Git branches to switch between specifications. For multi-feature projects using `/speckit.project`, multiple specs live on the same project branch (e.g., `project-taskify`) and are managed through the project directory structure.

### Step 1: Install Specify

**In your terminal**, run the `specify` CLI command to initialize your project:

```bash
# Create a new project directory
uvx --from git+https://github.com/gawainclaridge/spec-kit.git@unifyr-spec-kit specify init <PROJECT_NAME>

# OR initialize in the current directory
uvx --from git+https://github.com/gawainclaridge/spec-kit.git@unifyr-spec-kit specify init .
```

Pick script type explicitly (optional):

```bash
uvx --from git+https://github.com/gawainclaridge/spec-kit.git@unifyr-spec-kit specify init <PROJECT_NAME> --script ps  # Force PowerShell
uvx --from git+https://github.com/gawainclaridge/spec-kit.git@unifyr-spec-kit specify init <PROJECT_NAME> --script sh  # Force POSIX shell
```

### Step 1b: Set Up Your Agent File (Recommended)

Your agent file (e.g., `CLAUDE.md` for Claude Code) should contain your **product-level architecture** and universal engineering truths. This is the permanent, high-level "who we are as a product" document that rarely changes.

Think of it as the product's architectural identity:
- "We use a microservices architecture deployed on AWS"
- "All APIs follow REST conventions with OpenAPI specs"
- "We use PostgreSQL for persistent storage"

> [!TIP]
> The agent file is auto-generated from plan.md files, but includes a `Product Architecture` section at the top that you maintain manually. This section persists across regenerations.

#### Understanding the Three Context Files

| File | Owner | About | Scope |
|------|-------|-------|-------|
| **Agent file** (CLAUDE.md etc.) | Engineering | Universal product truths | Entire product/repo |
| **Constitution** | Engineering | **The HOW** - principles for this initiative | Project/epic set |
| **Project.md** | Product | **The WHAT** - shared feature context | Multi-feature project |

- **Agent file**: High-level product architecture, universal engineering truths. Rarely changes. Engineering-maintained.
- **Constitution**: Initiative-specific principles, quality gates, development approach. Guides Stage 3 planning. Engineering-managed.
- **Project.md**: Feature list, shared constraints, out-of-scope items, Jira Epic. Product-managed.

### Step 2: Create Specifications (Stage 1: Specification)

**In your AI Agent's chat interface**, launch your agent in the project directory. The `/speckit.*` commands are available in the assistant.

For multi-feature projects, use **`/speckit.project`** first to create shared context across features:

```markdown
/speckit.project my-project-name
```

Use **`/speckit.specify`** to describe what you want to build. Focus on the **what** and **why**, not the tech stack:

```markdown
/speckit.specify Build an application that can help me organize my photos in separate photo albums. Albums are grouped by date and can be re-organized by dragging and dropping on the main page. Albums are never in other nested albums. Within each album, photos are previewed in a tile-like interface.
```

Optionally, use **`/speckit.constitution`** to draft guiding principles for this initiative. The constitution is optional at this stage but must be finalized before Stage 3 (Planning):

```markdown
/speckit.constitution This project follows a "Library-First" approach. All features must be implemented as standalone libraries first. We use TDD strictly. We prefer functional programming patterns.
```

#### Practical Guidance for the Constitution

- Expect to invest 2-3 days of senior engineering time on the constitution
- Clean up existing repo documentation first (README, architecture notes) before drafting
- Document YOUR architecture decisions, not generic best practices
- Start with 5-7 focused principles that you actually enforce today

> [!NOTE]
> The constitution is optional in Stage 1 but **must be finalized before** `/speckit.plan` (Stage 3). If you skip this step, `/speckit.specify` will warn you but continue.

### Step 3: Review and Refine (Stage 2: Review)

Use **`/speckit.clarify`** to identify and resolve ambiguities in your specification. You can provide specific focus areas as arguments:

```bash
/speckit.clarify Focus on security and performance requirements.
```

### Step 4: Create Implementation Plan (Stage 3: Planning)

Use the **`/speckit.plan`** slash command to provide your tech stack and architecture choices. The constitution must be finalized before this step.

```markdown
/speckit.plan The application uses Vite with minimal number of libraries. Use vanilla HTML, CSS, and JavaScript as much as possible. Images are not uploaded anywhere and metadata is stored in a local SQLite database.
```

### Step 5: Generate Tasks and Implement (Stage 4: Tasks)

Use **`/speckit.tasks`** to create an actionable task list:

```markdown
/speckit.tasks
```

For larger teams, you can generate separate task files per user story:

```markdown
/speckit.tasks --per-story
```

Optionally, create Jira tickets or GitHub issues from your tasks:

```markdown
# Create GitHub issues (default)
/speckit.taskstoissues
# or explicitly:
/speckit.taskstoissues --github

# Create Jira tickets
/speckit.taskstoissues --jira PROJ
```

Then, use the **`/speckit.implement`** slash command to execute the plan:

```markdown
/speckit.implement
```

## Detailed Example: Building Taskify

Here's a complete example of building a team productivity platform, following the 4-stage process:

### Stage 1: Specification

Define requirements with `/speckit.specify`:

```text
Develop Taskify, a team productivity platform. It should allow users to create projects, add team members,
assign tasks, comment and move tasks between boards in Kanban style. In this initial phase for this feature,
let's call it "Create Taskify," let's have multiple users but the users will be declared ahead of time, predefined.
I want five users in two different categories, one product manager and four engineers. Let's create three
different sample projects. Let's have the standard Kanban columns for the status of each task, such as "To Do,"
"In Progress," "In Review," and "Done." There will be no login for this application as this is just the very
first testing thing to ensure that our basic features are set up.
```

Optionally, draft the project's constitution to set ground rules:

```markdown
/speckit.constitution Taskify is a "Security-First" application. All user inputs must be validated. We use a microservices architecture. Code must be fully documented.
```

### Stage 2: Review

Use the `/speckit.clarify` command to interactively resolve any ambiguities in your specification. You can also provide specific details you want to ensure are included.

```bash
/speckit.clarify I want to clarify the task card details. For each task in the UI for a task card, you should be able to change the current status of the task between the different columns in the Kanban work board. You should be able to leave an unlimited number of comments for a particular card. You should be able to, from that task card, assign one of the valid users.
```

You can continue to refine the spec with more details using `/speckit.clarify`:

```bash
/speckit.clarify When you first launch Taskify, it's going to give you a list of the five users to pick from. There will be no password required. When you click on a user, you go into the main view, which displays the list of projects. When you click on a project, you open the Kanban board for that project. You're going to see the columns. You'll be able to drag and drop cards back and forth between different columns. You will see any cards that are assigned to you, the currently logged in user, in a different color from all the other ones, so you can quickly see yours. You can edit any comments that you make, but you can't edit comments that other people made. You can delete any comments that you made, but you can't delete comments anybody else made.
```

Validate the specification checklist using the `/speckit.checklist` command:

```bash
/speckit.checklist
```

### Stage 3: Planning

Finalize the constitution if you drafted it in Stage 1. Then generate a technical plan — be specific about your tech stack and requirements:

```bash
/speckit.plan We are going to generate this using .NET Aspire, using Postgres as the database. The frontend should use Blazor server with drag-and-drop task boards, real-time updates. There should be a REST API created with a projects API, tasks API, and a notifications API.
```

### Stage 4: Tasks

Generate a task breakdown:

```bash
/speckit.tasks
```

Optionally, audit the implementation plan using `/speckit.analyze`:

```bash
/speckit.analyze
```

Then implement the solution:

```bash
/speckit.implement
```

## Multi-Feature Projects

For larger initiatives with multiple related features, use the `/speckit.project` command to create a shared project context:

```markdown
# Create a new project
/speckit.project taskify

# Add a spec to an existing project
/speckit.specify --project taskify Add user authentication

# List all specs in a project
/speckit.project taskify --list

# Add current spec to a project
/speckit.project taskify --add-spec
```

This creates a project structure with shared context:

```text
specs/
└── project-taskify/
    ├── project.md           # Shared context, out-of-scope items
    ├── 001-user-auth/       # First feature
    │   ├── spec.md
    │   ├── plan.md
    │   └── tasks.md
    └── 002-task-boards/     # Second feature
        ├── spec.md
        ├── plan.md
        └── tasks.md
```

The `project.md` file contains:
- **Out of Scope**: Project-level exclusions shared across all features
- **Shared Constraints**: Technical and business constraints
- **Features Table**: Links to all specs in the project
- **Jira Integration**: Project-level Epic key

## Jira Integration

Unifyr Spec Kit includes Jira placeholders in generated artifacts:

| Artifact | Placeholder | Maps To |
|----------|-------------|---------|
| project.md | `[JIRA-EPIC-KEY]` | Epic |
| tasks.md | `[JIRA-XXX]` | Story/Sub-task |
| tasks-us*.md | `[JIRA-STORY-KEY]` | Story |

Use `/speckit.taskstoissues --jira PROJ` to create actual tickets and update placeholders with real keys.

## Key Principles

- **Be explicit** about what you're building and why
- **Don't focus on tech stack** during specification phase
- **Iterate and refine** your specifications before implementation
- **Validate** the plan before coding begins
- **Let the AI agent handle** the implementation details
- **Use projects** for multi-feature initiatives with shared context
- **Handle changes gracefully** - see [Managing Changes Mid-Flight](../README.md#managing-changes-mid-flight) for when to amend vs restart

## Next Steps

- Read the [complete methodology](../spec-driven.md) for in-depth guidance
- Check out [more examples](../templates) in the repository
- Explore the [source code on GitHub](https://github.com/gawainclaridge/spec-kit)

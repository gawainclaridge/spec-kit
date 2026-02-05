---
description: Execute the implementation planning workflow using the plan template to generate design artifacts.
handoffs:
  - label: Create Tasks (Stage 4)
    agent: speckit.tasks
    prompt: Break the plan into tasks
    send: true
  - label: Create Checklist
    agent: speckit.checklist
    prompt: Create a checklist for the following domain...
scripts:
  sh: scripts/bash/setup-plan.sh --json
  ps: scripts/powershell/setup-plan.ps1 -Json
agent_scripts:
  sh: scripts/bash/update-agent-context.sh __AGENT__
  ps: scripts/powershell/update-agent-context.ps1 -AgentType __AGENT__
---

## User Input

```text
$ARGUMENTS
```

You **MUST** consider the user input before proceeding (if not empty).

### Workflow Context (Unifyr Process)

This is **Stage 3 (Planning)** of the Unifyr process:
- **Team**: Engineering only
- **Prerequisites**:
  - spec.md exists and ideally clarified (Stage 2)
  - constitution.md MUST be finalized (run `/speckit.constitution` first if not done)
- **Output**: plan.md with technical architecture, testing scenarios, design documents
- **Next step**: `/speckit.tasks` (Stage 4 - Engineering)

## Outline

1. **Setup**: Run `{SCRIPT}` from repo root and parse JSON for FEATURE_SPEC, IMPL_PLAN, SPECS_DIR, BRANCH. For single quotes in args like "I'm Groot", use escape syntax: e.g 'I'\''m Groot' (or double-quote if possible: "I'm Groot").

2. **Verify prerequisites**:
   - **Constitution finalized**: Check if `.specify/memory/constitution.md` exists and is finalized
     - If NOT found: ERROR "Constitution must be finalized before planning. Run /speckit.constitution first."
     - If found but Sign-Off section shows "Pending": WARN "Constitution not yet signed off - consider finalizing before proceeding"
   - **Spec sign-off check** (advisory): Check spec Sign-Off table status
     - If not signed off: WARN "Spec not yet signed off - plan may need revision after sign-off"
   - **Check for project context**: Determine if feature is part of a project
     - Check if current directory is under `specs/project-<name>/`
     - If project.md exists, load and apply project context (see step 2b)

2b. **Load project.md** (if feature is part of a project):
   - Read `specs/project-<name>/project.md`:
     - **Shared Constraints** → Validate plan doesn't violate these constraints
     - **Shared Tech Decisions** → Inherit stack choices (if defined at project level)
     - **Out of Scope** → Ensure plan doesn't include any excluded items
     - **Jira Integration** → Use Epic key for linking
   - Add "Project Alignment" check to plan validation:
     - Plan MUST NOT include features listed in project Out of Scope
     - Plan MUST respect Shared Constraints
     - Plan SHOULD inherit Shared Tech Decisions unless spec explicitly overrides

3. **Load context**: Read FEATURE_SPEC and `/memory/constitution.md`. Load IMPL_PLAN template (already copied).

4. **Scope alignment check**: Before proceeding with planning:
   - Extract all user stories from spec.md
   - Verify plan will address each story
   - If plan would add features beyond spec scope:
     - WARN "Plan would exceed spec scope"
     - List the additional features
     - Recommend: "Amend spec.md first and get Product sign-off before continuing"
     - Allow user to confirm they want to proceed anyway

5. **Execute plan workflow**: Follow the structure in IMPL_PLAN template to:
   - Fill Technical Context (mark unknowns as "NEEDS CLARIFICATION")
   - Fill Constitution Check section from constitution
   - Evaluate gates (ERROR if violations unjustified)
   - Phase 0: Generate research.md (resolve all NEEDS CLARIFICATION)
   - Phase 1: Generate data-model.md, contracts/, quickstart.md
   - Phase 1: Update agent context by running the agent script
   - Re-evaluate Constitution Check post-design

6. **Generate Testing Scenarios**: For each user story:
   - Extract acceptance criteria from spec.md
   - Generate happy path test scenarios
   - Generate edge case test scenarios
   - Generate error handling test scenarios
   - Add to plan.md "Testing Scenarios" section

7. **Initialize Sign-Off table** in plan.md with "Pending" statuses

8. **Fill Spec Reference section** in plan.md:
   - Link to source spec.md
   - Record spec version from changelog
   - Update Scope Alignment Check checkboxes

9. **Stop and report**: Command ends after Phase 2 planning. Report branch, IMPL_PLAN path, and generated artifacts.
   - Include summary of Testing Scenarios generated
   - Note Sign-Off status (all Pending for new plans)

## Phases

### Phase 0: Outline & Research

1. **Extract unknowns from Technical Context** above:
   - For each NEEDS CLARIFICATION → research task
   - For each dependency → best practices task
   - For each integration → patterns task

2. **Generate and dispatch research agents**:

   ```text
   For each unknown in Technical Context:
     Task: "Research {unknown} for {feature context}"
   For each technology choice:
     Task: "Find best practices for {tech} in {domain}"
   ```

3. **Consolidate findings** in `research.md` using format:
   - Decision: [what was chosen]
   - Rationale: [why chosen]
   - Alternatives considered: [what else evaluated]

**Output**: research.md with all NEEDS CLARIFICATION resolved

### Phase 1: Design & Contracts

**Prerequisites:** `research.md` complete

1. **Extract entities from feature spec** → `data-model.md`:
   - Entity name, fields, relationships
   - Validation rules from requirements
   - State transitions if applicable

2. **Generate API contracts** from functional requirements:
   - For each user action → endpoint
   - Use standard REST/GraphQL patterns
   - Output OpenAPI/GraphQL schema to `/contracts/`

3. **Agent context update**:
   - Run `{AGENT_SCRIPT}`
   - These scripts detect which AI agent is in use
   - Update the appropriate agent-specific context file
   - Add only new technology from current plan
   - Preserve manual additions between markers

**Output**: data-model.md, /contracts/*, quickstart.md, agent-specific file

## Key rules

- Use absolute paths
- ERROR on gate failures or unresolved clarifications

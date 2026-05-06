---
description: Implementation driver that reads project specs, progress, and test docs to plan and execute the next development phase.
allowed-tools: [Read, Write, Edit, Glob, Grep, Bash, Agent, EnterPlanMode, ExitPlanMode, AskUserQuestion, TaskCreate, TaskUpdate, TaskList, TaskGet]
---

# Coder — Spec-Driven Implementation

You are a software development implementation driver. Your job is to read the project's specification, understand current progress, and plan + implement the next development phase.

## Step 1: Discover Project Context

Scan the current working directory for these document categories. Use glob patterns and search broadly — file names vary across projects.

### Functional Specification
Search for:
- `*Functional*Spec*.md`, `*FSD*.md`, `*spec*.md`, `*requirements*.md`
- `*PRD*.md`, `*product*.md`, `*design*.md`
- `docs/*spec*`, `docs/*design*`

### Progress Tracking
Search for:
- `*Progress*.md`, `*progress*.md`, `Development_Progress.md`
- `*TODO*.md`, `*STATUS*.md`, `*Roadmap*.md`
- `changelogs/*.md`, `*changelog*.md`, `CHANGELOG.md`
- `docs/changelogs/*.md`

### Testing Documents
Search for:
- `*Test*Plan*.md`, `*test*.md`, `*testing*.md`
- `*Acceptance*.md`, `*QA*.md`, `*validation*.md`
- `tests/**/*.md`, `test/**/*.md`

### Architecture / Configuration
Search for:
- `CLAUDE.md`, `README.md`, `ARCHITECTURE.md`
- `*.config.*`, `package.json`, `pyproject.toml`, `Cargo.toml`, `Makefile`

**Use parallel agents** to discover and read these documents. Launch up to 3 agents simultaneously:
- **Spec Agent** — Find and read the functional specification and any design documents
- **Progress Agent** — Find and read progress tracking, changelogs, and status documents
- **Test Agent** — Find and read testing plans, acceptance criteria, and QA documents

If a document category is not found, note it and continue. Do not halt if some documents are missing.

## Step 2: Analyze and Determine Scope

After context is gathered, analyze what you've found:

1. **Identify the project structure** — What kind of project is this? What language, framework, platform?
2. **Map the phases** — From the spec or progress doc, list all development phases/milestones
3. **Determine current state** — What has been completed? What is next?
4. **Identify the next phase** — Select the next unstarted or in-progress phase to work on
5. **Extract acceptance criteria** — From the spec and test docs, what must be true when this phase is done?

If the user provided an argument (e.g., `/coder phase 3` or `/coder authentication`), target that specific phase or feature instead of the automatic "next" one.

## Step 3: Enter Plan Mode

**Always enter plan mode before writing any code.** This ensures the user can review and approve the implementation approach.

Write a plan that includes:

1. **Phase Summary** — Name, number, and description of the phase being implemented
2. **Requirements** — What the spec says this phase must deliver (with section references)
3. **Files to Create/Modify** — Specific file paths and what changes each needs
4. **Implementation Order** — Ordered list of tasks, noting dependencies between them
5. **Testing Strategy** — How to verify the implementation (unit tests, integration tests, manual checks)
6. **Risks / Open Questions** — Anything ambiguous in the spec or that needs clarification

Exit plan mode and wait for user approval before proceeding.

## Step 3b: Save Implementation Plan

After the plan is approved, persist it to an implementation plan file before writing any code. This creates a durable record of what was planned and serves as a reference during implementation and review.

### File Naming
Save the plan to the project root (or a `docs/` directory if one exists) using this naming convention:
- `Implementation_Plan_Phase_{N}.md` — for numbered phases (e.g., `Implementation_Plan_Phase_3.md`)
- `Implementation_Plan_{Feature}.md` — for feature-targeted runs (e.g., `Implementation_Plan_Authentication.md`)
- If neither a phase number nor feature name is available, use the next sequential number based on existing plan files (e.g., check for `Implementation_Plan_Phase_*.md` and increment)

### File Contents
The implementation plan file should include all sections from the approved plan:

```markdown
# Implementation Plan: {Phase/Feature Name}

**Date:** {current date}
**Status:** Approved

## Phase Summary
{name, number, and description}

## Requirements
{what the spec says this phase must deliver, with section references}

## Files to Create/Modify
{specific file paths and what changes each needs}

## Implementation Order
{ordered list of tasks with dependencies}

## Testing Strategy
{how to verify the implementation}

## Risks / Open Questions
{anything ambiguous or needing clarification}
```

### When to Update the File
- If the implementation deviates from the plan in a significant way, update the plan file to reflect the change and note the reason
- After implementation is complete, update the status field to `Completed` and append a brief summary of deviations (if any)

## Step 4: Implement

After the plan is approved, create tasks for each implementation step and execute them.

### Task Management
- Create a task for each distinct piece of work from the approved plan
- Mark tasks in_progress as you start them, completed as you finish them
- Use TaskList to track overall progress

### Execution Strategy
- **Follow the implementation order** from the approved plan
- **Use parallel agents** when multiple independent files need to be created or modified simultaneously
- **Read before writing** — always read existing files before editing them
- **Write tests** — implement tests as specified in the test documentation, or alongside the code if no test doc exists
- **Run tests** — execute the test suite after each significant change to catch regressions early

### Code Standards
- Follow patterns and conventions already established in the project
- Match the existing code style (naming, formatting, structure)
- Don't add unnecessary abstractions, comments, or features beyond what the spec requires
- Keep implementations focused on the current phase's requirements

## Step 5: Verify and Report

After implementation is complete:

1. **Run the full test suite** — ensure all existing and new tests pass
2. **Check acceptance criteria** — verify each criterion from the spec/test docs is met
3. **Run a lint/type check** if the project has one configured
4. **Report results** — summarize:
   - What was implemented
   - Which files were created/modified
   - Test results
   - Any deviations from the spec (and why)
   - Any remaining work or follow-up items

Then proceed to Step 6 to persist this report.

## Step 6: Write Run Summary File

After reporting results inline, persist a detailed summary to a Markdown file. Write it to `docs/` if that directory exists, otherwise to the project root.

### Filename
- `Coder_Summary_Phase{N}_{YYYYMMDD}_{HHMM}.md` for numbered phases (e.g., `Coder_Summary_Phase3_20260506_1814.md`)
- `Coder_Summary_{Feature}_{YYYYMMDD}_{HHMM}.md` for feature-targeted runs (e.g., `Coder_Summary_Authentication_20260506_1814.md`)
- Determine the timestamp once at the start of the run with `date +%Y%m%d_%H%M`. A fresh file is written per invocation; never overwrite prior summaries.

### Contents
The summary file should include:

```markdown
# Coder Run Summary: {Phase/Feature Name}

**Date:** {YYYY-MM-DD HH:MM}
**Phase / Feature:** {name and number}
**Plan File:** {path to Implementation_Plan_*.md, if any}

## What Was Implemented
{bulleted list of features / behaviours delivered this run}

## Files Created / Modified
| File | Change |
|------|--------|
| ... | ... |

## Test Results
{full suite output summary, lint/type-check status}

## Deviations from the Plan
{anything that differs from the approved Implementation_Plan_*.md, with reasons. Write "None" if there were none.}

## Outstanding Work / Follow-ups
{anything not finished, deferred, or surfaced as new work during implementation}
```

This file is the durable record of a single coder invocation — it complements (does not replace) `Implementation_Plan_*.md`, which remains the canonical evolving plan.

## Handling Missing Documents

If the project has no specification document:
- Ask the user what they want to build before proceeding
- Offer to help create a basic spec

If there is no progress tracking document:
- Infer progress from the codebase (existing files, git history)
- Offer to create a progress tracking document

If there are no test documents:
- Propose a testing approach in the plan based on project conventions
- Write tests alongside implementation

## Arguments

The user may pass arguments to target specific work:
- `/coder` — automatically determine and implement the next phase
- `/coder phase 3` — implement a specific phase number
- `/coder authentication` — implement a specific feature or component
- `/coder --plan-only` — stop after writing the plan, don't implement

Respect these arguments when determining scope.

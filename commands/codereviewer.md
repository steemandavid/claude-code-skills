---
description: LLM-powered code review against project spec and changelog
allowed-tools: [Read, Glob, Grep, Bash, Agent, EnterPlanMode]
---

# Code Reviewer

Perform a thorough code review of the latest development phase, evaluated against the project's specification and recent development history.

## Execution Strategy

**Switch to plan mode first.** Before reading any files, enter plan mode to outline your review approach. This allows the user to confirm the scope and strategy before the full review begins.

**Use parallel agents for independent review tracks.** Once the plan is approved, launch multiple agents in parallel to maximize efficiency:

- **Context Agent** — Gathers and reads all context documents (FSD, changelogs, prior reviews, architecture docs)
- **Coverage Agent** — Maps FSD requirements to source files and identifies which files belong to the phase under review
- **Review Agents** — When the phase spans many files (5+), split them across 2-3 parallel review agents, each focusing on a subset of files. Each agent evaluates all review criteria (spec conformance, correctness, safety, etc.) for its assigned files.

After all agents complete, synthesize their findings into the unified review output document.

For small phases (under 5 files), a single agent is sufficient — no need to parallelize.

## Context Gathering

Before reviewing any code, **read these documents** in order:

1. **Functional Specification (FSD)** — Search the project root for files matching these patterns (use the most recent version):
   - `*Functional*Spec*.md`, `*FSD*.md`, `*spec*.md`, `*requirements*.md`
   - If multiple versions exist in an `archive/` folder, use the one in the project root or the highest version number

2. **Development Progress / Changelog** — Search for:
   - `Development_Progress.md`, `*Progress*.md`
   - Changelog files in `changelogs/` or `docs/changelogs/` within the project root — read the most recent 2-3 entries
   - Also check `~/claudecode/changelogs/` if it exists (user-level centralized changelogs)
   - `CHANGELOG.md`, `*changelog*.md`

3. **Previous Review (if any)** — Search for:
   - `*Code_Review*.md`, `*review*.md`

4. **Architecture / Configuration** — Search for:
   - `CLAUDE.md`, `README.md`, `ARCHITECTURE.md`, `*design*.md`
   - Any config headers or protocol definition files that define constants, pin assignments, message formats, etc.

## Identify the Phase to Review

Determine the scope of the review:

1. Read the development progress document to find the **most recently completed phase**
2. If the user provided an argument (e.g., `/codereviewer phase 3`), use that phase instead
3. From the FSD or progress document, identify:
   - The phase name and number
   - Which tasks/features it includes
   - Which source files implement those features
   - Any test specifications or acceptance criteria

## Review Execution

Review **only** the files that belong to the identified phase.

**For phases with 5+ source files:** Split the files across 2-3 parallel review agents. Each agent reviews its assigned files against all criteria below, then returns findings. Synthesize the results into the final review document.

**For smaller phases:** Review all files directly without spawning agents.

Evaluate against these criteria:

### 1. Spec Conformance
- Does the implementation match what the FSD requires for this phase?
- Are there requirements that are missing, partially implemented, or deviate from the spec?
- Note the specific FSD section references for each finding

### 2. Correctness
- Does the logic do what it's supposed to?
- Are there edge cases, race conditions, or off-by-one errors?
- Are state transitions valid and complete?

### 3. Safety & Robustness
- Are there code paths that could lead to hazardous states?
- Are fail-safes and defaults robust?
- Can the system recover gracefully from communication failures, sensor errors, or power issues?

### 4. Concurrency & Real-Time
- Are tasks, threads, queues, mutexes, and interrupts used correctly?
- Any deadlock, starvation, or priority inversion risks?
- Are shared resources properly protected?

### 5. Platform & Framework Best Practices
- Proper use of the target platform's APIs (GPIO, ADC, wireless, storage, etc.)?
- Correct initialization and teardown sequences?
- Memory management — leaks, fragmentation, stack sizing?

### 6. Error Handling
- Are error codes checked and propagated?
- Can failures leave the system in an inconsistent state?
- Are recovery paths realistic?

### 7. Code Quality
- Clear naming and structure
- Separation of concerns
- No unnecessary complexity
- Consistent with patterns established in earlier phases

## Output Format

Write the review as a Markdown file named `Phase{N}_Code_Review.md` in the project root (e.g., `Phase2_Code_Review.md`). Use this structure:

```markdown
# Phase {N} Code Review — {Phase Name}

**Document ID:** {PROJECT}-REVIEW-P{N}-{SEQ}
**Reviewer:** Code Review Agent
**Date:** {today's date}
**Scope:** Phase {N} — {Phase Name}
**FSD Reference:** {FSD filename}
**Commit Reviewed:** {current HEAD short hash}

---

## Verdict: {PASS | PASS WITH NOTES | MAYBE | FAIL}

{One-paragraph summary of the overall assessment}

---

## Table of Contents

1. [Coverage Analysis](#1-coverage-analysis)
2. [Deviation Report](#2-deviation-report)
3. [Edge Cases & Safety](#3-edge-cases--safety)
4. [Concurrency & Platform Issues](#4-concurrency--platform-issues)
5. [Error Handling](#5-error-handling)
6. [Code Quality](#6-code-quality)
7. [Summary](#7-summary)
8. [Recommendation](#8-recommendation)

---

## Files Reviewed

| File | Purpose |
|------|---------|
| ... | ... |

---

## 1. Coverage Analysis

{Map each FSD requirement for this phase to its implementation status: DONE, PARTIAL, MISSING}

## 2. Deviation Report

{List each instance where implementation differs from the FSD, with severity: CRITICAL / MAJOR / MINOR}

## 3. Edge Cases & Safety

{Enumerate edge cases and safety concerns, with risk assessment}

## 4. Concurrency & Platform Issues

{Issues with threading, interrupts, hardware API usage}

## 5. Error Handling

{Gaps in error handling and recovery}

## 6. Code Quality

{Naming, structure, complexity observations — only substantive issues}

## 7. Summary

| Category | Critical | Major | Minor | Info |
|----------|----------|-------|-------|------|
| Spec conformance | ... | ... | ... | ... |
| Correctness | ... | ... | ... | ... |
| Safety | ... | ... | ... | ... |
| Concurrency | ... | ... | ... | ... |
| Error handling | ... | ... | ... | ... |
| Code quality | ... | ... | ... | ... |

## 8. Recommendation

{Clear go/no-go recommendation for proceeding to the next phase, with conditions if applicable}
```

## Important Guidelines

- **Focus on substance** — bugs, safety risks, design flaws, spec deviations. Not style nits.
- **Prioritize by severity** — Critical (will cause failure or harm) > Major (spec deviation, potential bug) > Minor (improvement) > Info (observation)
- **Be specific** — cite file paths, line numbers, function names, and FSD section references
- **Be honest** — if the code is solid, say so. Don't manufacture issues.
- **Reference prior reviews** — if a previous phase review exists, check whether earlier findings were addressed
- **Do not modify any code** — this is a read-only review. Report findings only.

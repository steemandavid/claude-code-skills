---
description: Initialize the session with context about the current project — reads relevant changelogs and the project's own .md files so Claude starts fresh sessions already knowing what was built and why.
allowed-tools: [Read, Glob, Grep, Bash]
---

# Project Context Loader

Load context about the current project so this session starts informed about prior work, decisions, and the project's structure. Run this at the start of a fresh chat session.

## Step 1: Identify the Current Project

The user's projects all live under `/home/john/claudecode/projects/<project>/`. The changelogs for ALL projects (and for unrelated system work) live in a single shared folder: `/home/john/claudecode/changelogs/`.

1. Run `pwd` to confirm the working directory.
2. If the working directory is inside `/home/john/claudecode/projects/<project>/...`, the project slug is `<project>` (the first path segment under `projects/`).
3. If the working directory is **not** under `/home/john/claudecode/projects/`, stop and tell the user: this skill is for project directories. Ask whether they want to point at a specific project folder instead.

Capture:
- `PROJECT_DIR` — absolute path of the project root (e.g. `/home/john/claudecode/projects/whisper`)
- `PROJECT_SLUG` — directory name (e.g. `whisper`)

## Step 2: Build a Keyword Set for Changelog Filtering

A single project can be referenced under several names in changelog filenames (e.g. `whisper` ↔ `whisper-transcription`, `MonthlyBudget` ↔ `monthlybudget`, `RS41spoofer` ↔ `rs41-spoofer`). Build a small keyword set:

1. Start with `PROJECT_SLUG` lowercased.
2. Split on common separators (`-`, `_`, camelCase boundaries) and add each component longer than 3 chars (e.g. `RS41spoofer` → `rs41`, `spoofer`).
3. Read `README.md`, `CLAUDE.md`, and any obvious top-level `.md` in `PROJECT_DIR` to pick up alternate names the project uses for itself (project title, product name, key acronyms). Add those to the keyword set.
4. Drop generic words that would over-match (`code`, `app`, `tool`, `script`, `setup`, `test`, `docs`).

## Step 3: Filter the Changelog Folder

Changelogs are flat files named `YYYY-MM-DD-<topic>.md` in `/home/john/claudecode/changelogs/`. Most topics include the project name in the filename, but not always.

1. **Filename pass** — list the changelog folder and pick files whose name contains any keyword from Step 2 (case-insensitive).
2. **Content pass** — for each keyword, run a Grep over the changelog folder for that keyword and add the matching files. This catches changelogs that mention the project without naming it in the filename.
3. Deduplicate, then sort the final list by filename (which sorts chronologically because of the date prefix). Read newest first if you need to budget time, but include them all in the summary so older foundational decisions aren't lost.
4. If the filtered list is empty, say so and continue with Step 4 only.

**Do not cap the list.** Older changelogs often contain foundational design decisions, the original spec rationale, and known issues that are still load-bearing — silently dropping them defeats the purpose of this skill. The keyword filter already narrows the set; trust it. If a match looks like a false positive on closer reading (a generic word collision rather than the actual project), drop that single entry, not a whole date range.

If the filtered set is unusually large (say 30+ entries), read the most recent in full and skim older ones (heading + first paragraph is enough to know whether they still matter) — but still surface them in the timeline.

## Step 4: Read Project-Local Documentation

Inside `PROJECT_DIR`, read the documentation files that explain the project itself. Prefer these in order:

1. `CLAUDE.md` (if present — auto-loaded already, but skim for context)
2. `README.md`
3. Any `*Spec*.md`, `*FSD*.md`, `*requirements*.md`, `*design*.md`, `*architecture*.md`
4. `Implementation_Plan_*.md`, `*Progress*.md`, `*STATUS*.md`, `*Roadmap*.md`
5. Anything in `docs/` if that directory exists

Skim — don't deeply read every file. The goal is orientation, not exhaustive review.

## Step 5: Glance at Code Structure (Light Touch)

To ground the docs in what actually exists:

1. List the top-level entries of `PROJECT_DIR` (via Glob or `ls`) so you know which files/dirs are present.
2. If there is an obvious entrypoint (`main.py`, `index.js`, `Dockerfile`, `Cargo.toml`, `package.json`, `pyproject.toml`, a single shell script), note it but do not read it in full unless something in the docs is unclear.

Avoid recursive reads of `node_modules`, `.git`, `venv`, `dist`, `build`, vendored ESP-IDF, or other large dependency trees.

## Step 6: Summarize for the User

Produce a tight briefing — this is what the user sees, so make it useful at a glance. Aim for under ~25 lines:

- **Project:** name + one-line description (from README/spec)
- **Current state:** what's built, what's in progress (inferred from the most recent 1–3 changelogs and any progress doc)
- **Recent activity:** bullet list of the last few changelog entries with date + one-line summary each. If there were significantly older but still-relevant changelogs (foundational decisions, ongoing known issues), include them in a separate "earlier context" sub-list rather than dropping them.
- **Key files / entrypoints:** the handful worth knowing about
- **Open items / known issues:** anything flagged as TODO, follow-up, or "known issue" in the docs or recent changelogs

End with a short prompt asking the user what they want to work on. Do **not** dump the full contents of changelogs or docs back into the chat — the user already has them on disk; the value here is the synthesis.

## Arguments

- `/projectcontext` — auto-detect the project from the working directory (default behavior above)
- `/projectcontext <slug>` — force a specific project, e.g. `/projectcontext whisper`. Use this when the working directory isn't a project folder, or when the user wants context for a different project.
- `/projectcontext --full` — explicit "read everything in full, skip any skim shortcuts" mode. The default already includes all matching changelogs; this flag just disables the skim-older-entries optimization in Step 3 for unusually large match sets.

## Notes

- This skill is read-only — it should never modify files in the project or the changelog folder.
- If a changelog filename matches a keyword but the content turns out to be about something unrelated (e.g. a generic word collision), drop that single entry rather than including a misleading one — but do not drop entries just because they are old.
- If no project documentation and no matching changelogs are found, say so plainly and offer to help create a CLAUDE.md or initial spec — don't fabricate context.

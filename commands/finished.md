---
description: End-of-session workflow. Updates documentation, writes a changelog, then commits/pushes project changes (only for existing project repos with uncommitted changes).
allowed-tools: [Read, Write, Glob, Grep, Bash, AskUserQuestion, Skill]
---

# Finished — End of Session

Run this workflow to wrap up a session cleanly.

## Steps

### Step 1: Documentation

Update all project documentation to reflect changes made during this session.

1. Identify the project's documentation files (e.g., `README.md`, `CLAUDE.md`, `docs/` directory, API docs, architecture docs, etc.).
2. Review the session's changes (git diff, recent edits, conversation context) to determine what was modified, added, or removed.
3. Update documentation to accurately reflect the current state of the project:
   - Add entries for new features, commands, or configuration options.
   - Update descriptions of changed behavior.
   - Remove documentation for deleted or deprecated functionality.
   - Ensure code examples and references are up to date.
4. If no documentation files exist or no updates are needed, skip this step.
5. Report what documentation was updated (if anything).

### Step 2: Changelog

Run the `/changelog` skill to write or update the session changelog.

**Always run this step**, regardless of whether the session was project work, system administration, infrastructure setup, or any other type of task. The changelog skill already handles both project changelogs and machine-local changelogs — do not skip it.

### Step 3: Check for Project Repository

After the changelog is written, determine if this is a project that should be committed to GitHub:

1. Check if a valid git repository exists: `git rev-parse --git-dir 2>/dev/null`
2. Check if a GitHub remote is already configured: `git remote -v | grep github.com`

**Skip to Step 4** if ANY of these are true:
- No valid git repository exists
- Git repository is corrupted (e.g., `git status` fails with errors)
- No GitHub remote is configured (this indicates a non-project directory)

**Only proceed to Step 4** if:
- Valid git repository exists AND
- GitHub remote is already configured AND
- There are uncommitted changes

### Step 4: Git & GitHub (Only for Existing Project Repos)

1. Run `git status` and `git diff` to check for uncommitted changes.
2. If there are **no changes**, you're done.
3. If there **are changes**, automatically commit and push:
   - Stage the changes: `git add <files>`
   - Commit with appropriate message: `git commit -m "..."` (include Co-Authored-By)
   - Push to GitHub: `git push origin <branch>`
4. Report what was committed and pushed.

**Note:** Never offer to create a new GitHub repository. This skill only handles existing project repos.

## Completion

When the workflow is complete, output exactly this single line as the final line of your response:

```
=== /finished skill completed ===
```

This line is mandatory and must always be emitted, regardless of which steps were run or skipped. Output nothing else after it.

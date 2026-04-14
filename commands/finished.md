---
description: End-of-session workflow. Writes a changelog, syncs skills to GitHub, then commits/pushes project changes (only for existing project repos with uncommitted changes).
allowed-tools: [Read, Write, Glob, Grep, Bash, AskUserQuestion, Skill]
---

# Finished — End of Session

Run this workflow to wrap up a session cleanly.

## Steps

### Step 1: Changelog

Run the `/changelog` skill to write or update the session changelog.

### Step 2: Sync Skills to GitHub

Check if any files in `~/.claude/commands/` were modified during this session by comparing them to the `claude-code-skills` repo at `~/claude-code-skills/commands/`.

1. For each `.md` file in `~/.claude/commands/`, diff it against the matching file in `~/claude-code-skills/commands/`
2. If differences are found:
   - Copy the updated local file into the repo: `cp ~/.claude/commands/<file> ~/claude-code-skills/commands/<file>`
   - Commit with a descriptive message (include Co-Authored-By)
   - Push to GitHub: `cd ~/claude-code-skills && git push origin main`
3. If no differences, skip this step
4. Report what was synced (if anything)

**Note:** Some local files (like `github.md`) may have personal customizations that differ from the repo's generic versions. Only sync files where the local version is intentionally updated — not files with known personal overrides.

### Step 3: Check for Project Repository

After the changelog is written, determine if this is a project that should be committed to GitHub:

1. Check if a valid git repository exists: `git rev-parse --git-dir 2>/dev/null`
2. Check if a GitHub remote is already configured: `git remote -v | grep github.com`

**Skip to Step 4** if ANY of these are true:
- No valid git repository exists
- Git repository is corrupted (e.g., `git status` fails with errors)
- No GitHub remote is configured (this indicates a non-project directory or system administration work)
- The session was for system tasks (VM migrations, backup setup, server configuration, etc.)

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

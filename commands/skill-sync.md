---
description: Manually sync Claude Code skills from ~/.claude/commands/ to the claude-code-skills GitHub repo. Use when you have developed a new skill or changed an existing one and want to push it to GitHub.
allowed-tools: [Read, Write, Glob, Grep, Bash]
---

# Skill Sync — Sync Skills to GitHub

Sync skills from the local `~/.claude/commands/` directory to the `claude-code-skills` repo at `~/claude-code-skills/commands/`, then commit and push to GitHub.

Run this manually when you have created a new skill or modified an existing one. It is **not** part of the end-of-session `/finished` workflow.

## Steps

### Step 1: Verify the Repo Exists

Check that the skills repo is present:

```bash
ls -d ~/claude-code-skills
```

If it does not exist, stop and report that the repo is missing — do not attempt to create it.

### Step 2: Find Changed or New Skills

Compare each `.md` file in `~/.claude/commands/` against the matching file in `~/claude-code-skills/commands/`:

1. List local skills: `ls ~/.claude/commands/*.md`
2. For each file, diff it against the repo copy: `diff ~/.claude/commands/<file> ~/claude-code-skills/commands/<file>`
3. A file needs syncing if:
   - The diff shows differences, OR
   - The file exists locally but not in the repo (a new skill)

If no differences are found and no new files exist, report that everything is already in sync and stop.

### Step 3: Copy and Review

For each file that needs syncing:

1. Copy the local file into the repo: `cp ~/.claude/commands/<file> ~/claude-code-skills/commands/<file>`
2. Re-run the diff to confirm the copy is now identical.

**Note:** Some local files (like `github.md`) may carry personal customizations that differ from the repo's generic versions. Only sync files where the local version is intentionally meant to be canonical — skip files with known personal overrides unless explicitly asked.

### Step 4: Commit and Push

1. Check the working state: `git -C ~/claude-code-skills status` and `git -C ~/claude-code-skills branch --show-current`
2. Stage the changed skill files: `git -C ~/claude-code-skills add commands/<file> ...`
3. Commit with a descriptive message (include Co-Authored-By):
   ```bash
   git -C ~/claude-code-skills commit -m "<message>"
   ```
4. Push to GitHub: `git -C ~/claude-code-skills push origin main`

### Step 5: Report

List each skill that was synced, committed, and pushed, plus the resulting commit hash.

## Completion

When the workflow is complete, output exactly this single line as the final line of your response:

```
=== /skill-sync completed ===
```

This line is mandatory and must always be emitted, regardless of which steps were run or skipped. Output nothing else after it.

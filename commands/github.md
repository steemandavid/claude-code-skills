---
name: github
description: Create repositories, commit changes, and push to GitHub. Use when the user wants to create a new repo, commit code, push changes, or perform git operations with GitHub. Handles authentication via gh CLI with SSH keys.
allowed-tools: [Read, Write, Glob, Grep, Bash, AskUserQuestion]
---

# GitHub Repository Management

Manage GitHub repositories: create new repos, commit changes, and push to GitHub.

## Authentication

GitHub authentication uses the `gh` CLI. Verify authentication status first:

```bash
gh auth status
```

If not authenticated, run:

```bash
gh auth login
```

Select:
1. GitHub.com
2. SSH
3. Yes (upload existing SSH key)

## Git Identity

Before committing in a new repo, set the local git identity:

```bash
git config user.name "<user's name>"
git config user.email "<user's email>"
```

Use the values from the user's global git config if available (`git config --global user.name`, `git config --global user.email`). If not set, ask the user.

## Creating a New Repository

### 1. Initialize Local Repository

```bash
mkdir -p /path/to/repo-name
cd /path/to/repo-name
git init
git branch -m main
```

### 2. Configure Git Identity

```bash
git config user.name "<name>"
git config user.email "<email>"
```

### 3. Add Files and Commit

```bash
git add <files>
git commit -m "Initial commit: <description>

Co-Authored-By: Claude Opus 4.6 <noreply@anthropic.com>"
```

### 4. Create GitHub Repository and Push

Determine the GitHub username from `gh auth status`, then:

```bash
gh repo create <repo-name> --public --description "<description>" --source=. --push
```

For private repos, use `--private` instead of `--public`.

## Committing Changes to Existing Repository

### 1. Check Status

```bash
git status
git diff
git log --oneline -5
```

### 2. Stage and Commit

```bash
git add <files>
git commit -m "<message>

Co-Authored-By: Claude Opus 4.6 <noreply@anthropic.com>"
```

### 3. Push

```bash
git push origin main
```

Or for any branch:
```bash
git push origin <branch-name>
```

## Cloning Existing Repository

```bash
gh repo clone <owner>/<repo-name>
```

Or via SSH:
```bash
git clone git@github.com:<owner>/<repo-name>.git
```

## Creating Pull Requests

```bash
gh pr create --title "<title>" --body "<description>"
```

## Common Operations

### List Repositories
```bash
gh repo list
```

### View Repository Info
```bash
gh repo view <owner>/<repo-name>
```

## Workflow Summary

1. **New repo**: init → config → add → commit → `gh repo create --push`
2. **Existing repo**: status → add → commit → push
3. **Auth check**: `gh auth status` first, login if needed

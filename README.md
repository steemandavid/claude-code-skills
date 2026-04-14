# Claude Code Skills

Reusable custom commands (skills) for [Claude Code](https://docs.anthropic.com/en/docs/claude-code). Copy the `.md` files into `~/.claude/commands/` to make them available as slash commands.

## Installation

```bash
# Clone the repo
git clone https://github.com/steemandavid/claude-code-skills.git

# Copy individual skills to your commands directory
cp claude-code-skills/commands/changelog.md ~/.claude/commands/

# Or copy all of them
cp claude-code-skills/commands/*.md ~/.claude/commands/
```

## Available Skills

| Command | Description |
|---------|-------------|
| `/codereviewer` | LLM-powered code review against project spec and changelog. Auto-discovers FSD, progress docs, and previous reviews. Evaluates spec conformance, correctness, safety, concurrency, error handling, and code quality. |
| `/changelog` | Write a detailed summary of the session's work to a dated changelog file. Detects existing entries and updates rather than duplicating. |
| `/finished` | End-of-session workflow: writes a changelog, then commits and pushes to GitHub (only for existing project repos with uncommitted changes). |
| `/github` | Create repositories, commit changes, and push to GitHub. Handles authentication via `gh` CLI with SSH keys. |

## Usage

Skills are invoked as slash commands in Claude Code:

```
/codereviewer              # Review the latest completed phase
/codereviewer phase 3      # Review a specific phase
/changelog                 # Write session changelog
/finished                  # End-of-session wrap-up
/github                    # GitHub operations
```

## Writing Custom Skills

Each skill is a Markdown file with YAML frontmatter:

```markdown
---
description: Short description shown in the skill list
allowed-tools: [Read, Write, Glob, Grep, Bash]
---

# Skill Name

Instructions for Claude Code to follow when this skill is invoked.
```

Place `.md` files in `~/.claude/commands/` to make them globally available, or in `.claude/commands/` within a project directory for project-specific skills.

## License

MIT

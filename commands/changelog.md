---
description: Write a summary of the current session's work to a changelog .md file
allowed-tools: [Read, Write, Glob, Bash]
---

# Changelog Writer

Write a detailed summary of the work performed in this session to a new `.md` file.

## Changelog Location

Changelogs are split into two categories:

1. **Machine-local changelogs** — for infrastructure, sysadmin, and machine-specific work:
   - Path: `/home/john/claudecode/changelogs/<hostname>/` where `<hostname>` is the output of `hostname`
   - These are NOT synced between machines (each machine has its own directory)

2. **Project changelogs** — for project-specific development work:
   - Path: Inside the project directory, e.g. `/home/john/claudecode/projects/<project>/changelog.md`
   - These ARE synced between machines via Syncthing

**Which one to use:**
- If working on a specific project (code in `~/claudecode/projects/<project>/`), write to that project's `changelog.md`
- If working on machine infrastructure, system setup, or cross-cutting tasks, write to the machine-local directory
- When in doubt, write to the machine-local directory

Create directories if they do not exist.

## Instructions

1. **Review the conversation** to identify:
   - What was the goal / problem being solved
   - What steps were taken (commands run, configs changed, files created/modified)
   - Important details needed for future reference (IPs, paths, credentials locations, caveats, gotchas)
   - Any known issues or follow-up items

2. **Check if a changelog already exists for this session:**
   - List files in the changelog directory
   - Look for a file created during this session (today's date, matching the project/topic)
   - If one exists, read it and **update it** with the new actions rather than creating a new file
   - Add new sections or append to existing sections as appropriate — do not duplicate content already captured

3. **If no changelog exists for this session**, check existing changelogs for format reference by reading one or two recent files to match the style

4. **Name new files** as `YYYY-MM-DD-short-description.md` using today's date

5. **Write or update the file** following the established format:
   - Top-level heading with title and date
   - System Info section if relevant (hostnames, IPs, disk UUIDs, etc.)
   - Numbered sections for each major area of work
   - Tables for structured data (packages, retention policies, file paths, etc.)
   - Code blocks for commands, config snippets, and file contents
   - Notes section for caveats, gotchas, and things to watch out for

6. **Confirm** to the user with the filename and a brief summary of what was captured or updated.

---
description: The API rate limit has reset — reorient and resume the interrupted work
---

# Rate Limit Reset — Resume

The rate limit that was blocking progress has reset. Reorient to where things left off and continue the interrupted work.

## Instructions

1. **Re-establish state.** Before charging ahead, determine what was in progress:
   - **Read the checkpoint** at `~/.claude/checkpoint.json` (written automatically after every turn, and again on a rate-limit failure). Its `timestamp`, `last_assistant_message`, `last_tool_use`, and `git.dirty_files` are the freshest mechanical snapshot — start here. If `event` is `StopFailure` or `error` is `rate_limit`, that confirms a rate-limit interruption. If its `cwd` differs from the current project, treat the data cautiously. If the file is missing, skip silently — the steps below cover it.
   - Check the task list (`TaskList`) — note anything `in_progress`, `pending`, or `blocked`. The most reliable source of "what's left."
   - If inside a git repo, run `git status` and look at recent commits to spot uncommitted changes belonging to the in-progress work.
   - Review the most recent messages in this conversation for the immediate task and any partial work — a command about to run, a file half-edited, a step mid-way through.
   - If the project keeps one, check its changelog or `SESSION_CONTEXT.md`.

2. **Confirm briefly (2–3 lines).** State what was in progress and what the next concrete step is. If it's unclear where things left off, say so and ask — do **not** guess and barrel into the wrong task.

3. **Continue.** Execute the next step. Do not re-derive facts already established in this conversation or re-litigate decisions already made — just resume.

## Notes

- A rate-limit interruption does **not** lose context — conversation history and the task list are intact. The only goal is to reorient quickly and proceed.
- Prefer the task list first when resuming a multi-step change.
- If nothing was actually in progress (rate limit hit at the very start, or this is a fresh session), ask the user what they'd like to work on rather than inventing a task.

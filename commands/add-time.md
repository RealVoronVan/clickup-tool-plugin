---
description: Log time on a ClickUp task. Can analyze git history to determine
  duration and description automatically.
argument-hint: <TASK_ID> <DURATION> [--description TEXT] [--date YYYY-MM-DD]
---

Log a time tracking entry on a ClickUp task.

## Input

- `$ARGUMENTS` may contain task ID, duration, and optional flags.
- If arguments are incomplete, ask the user or infer from context.
- Duration formats: "2h 30m", "45m", "150" (minutes).

## Steps

1. Parse arguments from `$ARGUMENTS`.
2. If the user gives a general instruction (e.g., "figure out how long I worked on this today"),
   use git log, commit history, and other tools to estimate duration and compose a description.
3. Show the user a preview:
   - **Task**: task ID (and task name if known)
   - **Duration**: human-readable
   - **Description**: what will be logged
   - **Date**: which day (default: today)
4. Ask the user to confirm before logging.
5. Run: `clickup-tool add-time-entry TASK_ID "duration" --description "text" --date YYYY-MM-DD`
6. Report the result.

## Important

- ALWAYS show preview and get confirmation before logging.
- NEVER log time without user approval.
- If the user asks to log time across multiple tasks, handle each one separately with its own confirmation.

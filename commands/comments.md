---
description: Get comments and discussion on a ClickUp task
allowed-tools: Bash(clickup-tool *)
argument-hint: <TASK_ID>
context: fork
agent: clickup-executor
---

Run `clickup-tool get-comments $ARGUMENTS` and present the comments.

The argument must be a task ID string (e.g. `86a1b2c3d`). If the user provides a task name instead of ID, first run `clickup-tool get-tasks` to find the task ID. If $ARGUMENTS is empty, ask the user for the task ID or suggest running `/tasks` first to find it.

Present comments chronologically. For each comment:
- **user** (date): text
- Attachments below the text (if any)
- Replies indented under parent comment

Show total comment count at the end.

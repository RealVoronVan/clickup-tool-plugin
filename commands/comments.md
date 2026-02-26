---
description: Get comments and discussion on a ClickUp task
allowed-tools: Task(clickup-executor)
argument-hint: <TASK_ID>
---

Use the **clickup-executor** subagent to fetch and format comments. Do NOT run clickup-tool commands inline.

If `$ARGUMENTS` is empty, ask the user for the task ID or suggest running `/tasks` first to find it. Otherwise, dispatch the clickup-executor agent with the Task tool, providing the following prompt:

---

**Command:** `get-comments`
**Task ID:** $ARGUMENTS

## Instructions

1. If the argument looks like a task name (not an alphanumeric ID), first run `clickup-tool get-tasks` to find the task ID
2. Run `clickup-tool get-comments TASK_ID`
3. Format as threaded conversation: **username** (date): text, with replies indented
4. Show attachments below each comment if present
5. Show total comment count at the end

---

Present the subagent's summary to the user as-is. Do not re-run the commands yourself.

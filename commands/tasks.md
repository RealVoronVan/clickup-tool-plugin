---
description: List ClickUp tasks assigned to me (filter by status, tag, assignee)
allowed-tools: Task(clickup-executor)
argument-hint: "[--status STATUS] [--tag TAG] [--assignee ID] [--list-id ID]"
---

Use the **clickup-executor** subagent to fetch and format task data. Do NOT run clickup-tool commands inline.

Dispatch the clickup-executor agent with the Task tool, providing the following prompt:

---

**Command:** `get-tasks`
**User request:** $ARGUMENTS

## Available statuses and members

Statuses per space:
!`clickup-tool get-spaces 2>/dev/null || echo "[]"`

Workspace members:
!`clickup-tool get-members 2>/dev/null || echo "[]"`

## Instructions

1. Resolve any person names to numeric IDs using the members list above
2. Resolve any status names to exact strings using the spaces data above
3. If user mentions "sprint" or "iteration": run `clickup-tool get-spaces` → `clickup-tool get-folders SPACE_ID` → `clickup-tool get-lists FOLDER_ID` to find the list ID
4. If user mentions a tag: verify exact name with `clickup-tool get-tags SPACE_ID`
5. Run `clickup-tool get-tasks` with resolved `--flag VALUE` arguments
6. Format results as a markdown table and suggest `/clickup-tool:task TASK_ID` for details

---

Present the subagent's summary to the user as-is. Do not re-run the commands yourself.

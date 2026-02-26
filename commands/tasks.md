---
description: List ClickUp tasks assigned to me (filter by status, tag, assignee)
allowed-tools: Bash(clickup-tool *)
argument-hint: "[--status STATUS] [--tag TAG] [--assignee ID] [--list-id ID]"
context: fork
agent: clickup-executor
---

## Available statuses and members

!`clickup-tool get-spaces 2>/dev/null || echo "[]"`
!`clickup-tool get-members 2>/dev/null || echo "[]"`

## Syntax

```
clickup-tool get-tasks [--status STATUS] [--tag TAG] [--assignee ID] [--list-id ID] [--space-id ID] [--include-closed]
```

All flags use `--flag VALUE` syntax. NEVER use `key:value` or positional arguments.

## Before running

**If user mentions a person's name** (e.g. "Jane", "Alex"): find their numeric ID in the members list above. NEVER pass a name as assignee value.

**If user mentions a status** (e.g. "in progress", "review"): use the exact status name from the spaces data above. Do NOT guess — match exactly.

**If user mentions "sprint" or "iteration"**: these map to Lists. Run `clickup-tool get-spaces` → `clickup-tool get-folders SPACE_ID` → `clickup-tool get-lists FOLDER_ID` to find the list ID, then use `--list-id LIST_ID`.

**If user mentions a tag**: tag names must match exactly. If unsure, run `clickup-tool get-tags SPACE_ID` to check available tags.

## Wrong (NEVER do this)

```
clickup-tool get-tasks assignee:Jane status:"in progress"
clickup-tool get-tasks --assignee Jane
clickup-tool get-tasks 900000000003
```

## Correct

```
clickup-tool get-tasks --assignee 100000004 --status "in progress"
clickup-tool get-tasks --list-id 900000000003 --assignee 100000001
clickup-tool get-tasks --tag "frontend" --status "review"
```

## Execution

1. Resolve any names/statuses/sprints to numeric IDs or exact strings using the data above
2. Run `clickup-tool get-tasks` with the resolved `--flag VALUE` arguments
3. Present results as a table or compact list. For each task show:
   - name (with task ID in parentheses)
   - status
   - priority (if present)
   - assignees (usernames)
   - due_date (if present)
   - tags (if present)
   - list name (sprint)
4. After the list, suggest: `/task TASK_ID` for full details

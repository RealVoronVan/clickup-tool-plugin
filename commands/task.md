---
description: Get full details of a specific ClickUp task including description, checklists, and time tracking
allowed-tools: Bash(clickup-tool *)
argument-hint: <TASK_ID>
context: fork
agent: clickup-executor
---

Run `clickup-tool get-task $ARGUMENTS` and present full task details.

The argument must be a task ID string (e.g. `86a1b2c3d`). If the user provides a task name instead of ID, first run `clickup-tool get-tasks` to find the task ID. If $ARGUMENTS is empty, ask the user for the task ID or suggest running `/tasks` first to find it.

Present task details in this order:
1. **Header**: name — status — priority
2. **People**: assignees, creator
3. **Dates**: due_date, start_date, date_created
4. **Time**: time_estimate vs time_spent (if present)
5. **Description**: markdown_description (full text)
6. **Checklists**: name + resolved/total per checklist, then items
7. **Custom fields**: name: value pairs
8. **Attachments**: list with titles and URLs
9. **Tags**: inline list
10. **Link**: task URL

Suggest `/comments $ARGUMENTS` for task discussion.

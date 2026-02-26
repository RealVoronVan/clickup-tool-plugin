---
description: Get full details of a specific ClickUp task including description, checklists, and time tracking
allowed-tools: Task(clickup-executor)
argument-hint: <TASK_ID>
---

Use the **clickup-executor** subagent to fetch and format task details. Do NOT run clickup-tool commands inline.

If `$ARGUMENTS` is empty, ask the user for the task ID or suggest running `/clickup-tool:tasks` first to find it. Otherwise, dispatch the clickup-executor agent with the Task tool, providing the following prompt:

---

**Command:** `get-task`
**Task ID:** $ARGUMENTS

## Instructions

1. If the argument looks like a task name (not an alphanumeric ID), first run `clickup-tool get-tasks` to find the task ID
2. Run `clickup-tool get-task TASK_ID`
3. Format the result as structured sections: Header, People, Dates, Time, Description, Checklists, Custom fields, Attachments, Tags, Link
4. Skip sections where data is absent
5. Suggest `/clickup-tool:comments TASK_ID` for task discussion

---

Present the subagent's summary to the user as-is. Do not re-run the commands yourself.

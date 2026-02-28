---
description: Change the status of a ClickUp task. Understands free-form input like "в работу", "close it", "on hold".
allowed-tools: Task(clickup-executor)
argument-hint: <TASK_ID> <status in free form>
---

Use a **Haiku** clickup-executor subagent to change a task's status. Do NOT run clickup-tool commands inline.

If `$ARGUMENTS` is empty, ask the user for the task ID and desired status. Otherwise, dispatch the clickup-executor agent with the Task tool using **model: haiku**, providing the following prompt:

---

**Command:** `set-status`
**Arguments:** $ARGUMENTS

## Instructions

1. Parse the first word as TASK_ID. Everything after is the desired status in free form.
2. Run `clickup-tool get-statuses TASK_ID` to get the list of valid statuses.
3. Match the user's free-form text to exactly one status from the list.
   - If the match is **unambiguous** (one clear candidate), proceed immediately.
   - If the match is **ambiguous** (multiple candidates could fit), show the options and ask the user to pick one. Example: "Did you mean: 1) completed 2) done 3) cancelled?"
4. Run `clickup-tool set-status TASK_ID "exact status name"`
5. Report: "Status of TASK_ID changed to **exact status name**"

---

Present the subagent's result to the user as-is. Do not re-run the commands yourself.

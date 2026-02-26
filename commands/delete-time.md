---
description: Delete a time tracking entry from ClickUp.
argument-hint: <TIMER_ID>
---

Delete a time tracking entry.

## Input

- `$ARGUMENTS` should contain the timer ID.
- If no timer ID provided, suggest running `/clickup-tool:task TASK_ID` first to see time entries with their IDs.

## Steps

1. Parse timer ID from `$ARGUMENTS`.
2. If no timer ID, ask the user which entry to delete.
3. Show the user a confirmation:
   - **Timer ID**: the entry to delete
   - **Warning**: this action cannot be undone
4. Ask the user to confirm.
5. Run: `clickup-tool delete-time-entry TIMER_ID`
6. Report the result.

## Important

- ALWAYS ask for confirmation before deleting.
- NEVER delete without user approval.
- This action is irreversible.

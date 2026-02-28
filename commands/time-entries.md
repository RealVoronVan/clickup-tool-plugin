---
description: Show time tracking entries for a date range. Understands "this week", "last month", specific dates, etc.
allowed-tools: Task(clickup-executor)
argument-hint: [--start-date YYYY-MM-DD] [--end-date YYYY-MM-DD] [--assignee ID]
---

Use a **Haiku** clickup-executor subagent to fetch and format time entries. Do NOT run clickup-tool commands inline.

If `$ARGUMENTS` is empty, assume the user wants the default (last 30 days for their default user). Dispatch the clickup-executor agent with the Task tool using **model: haiku**, providing the following prompt:

---

**Command:** `get-time-entries`
**Arguments:** $ARGUMENTS

## Instructions

1. Parse the arguments for date range and assignee. If the user provided free-form dates (e.g., "this week", "last month", "february"), convert to YYYY-MM-DD format.
2. Build the command: `clickup-tool get-time-entries [--start-date YYYY-MM-DD] [--end-date YYYY-MM-DD] [--assignee ID]`
3. If no dates specified, omit the flags (API defaults to last 30 days).
4. If no assignee specified, omit the flag (CLI defaults to configured user).
5. Run the command and format the results.

## Output format

Group entries by task, show per-task subtotals and a grand total:

| Task | Duration | Date | Description |
|------|----------|------|-------------|
| **Task Name** (ID) | | | |
| | 2h 30m | 2026-02-20 | OAuth2 work |
| | 1h 30m | 2026-02-21 | Session mgmt |
| *Subtotal* | *4h* | | |
| **Another Task** (ID) | | | |
| | 30m | 2026-02-22 | Debugging |
| *Subtotal* | *30m* | | |

**Total: 4h 30m** (3 entries)

If no entries found, report: "No time entries found for the specified period."

---

Present the subagent's result to the user as-is. Do not re-run the commands yourself.

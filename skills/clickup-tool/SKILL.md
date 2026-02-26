---
name: clickup-tool
description: This skill should be used when the user asks to "check my tasks",
  "show ClickUp tasks", "get task details", "view task comments", "show project
  structure", "list sprints", "what am I working on", "show my ClickUp board",
  "task status", "add a comment", "post a note", "log time", "track time",
  "delete time entry", or mentions ClickUp, tasks, sprints, or project management
  queries. Provides read and write access to ClickUp workspace data through the
  clickup-tool CLI with normalized, compressed JSON output.
allowed-tools: Bash(clickup-tool *)
---

# ClickUp Task Management Skill

## Overview

clickup-tool is a CLI for extracting ClickUp task data with configurable output normalization. It connects to the ClickUp API v2 and returns structured JSON to stdout. Raw API responses (32--92 KB per request) are compressed to 1.8--2.5 KB through field filtering, value resolution, and format conversion. The tool handles pagination, deduplication, rate limiting, and automatic retries transparently.

The ClickUp data hierarchy is: **Workspace > Space > Folder > List > Task**. A workspace contains multiple spaces (e.g., "Engineering", "Design"). Spaces contain folders (project groupings). Folders contain lists, which are often used as sprints or iterations. Lists contain individual tasks. Understanding this hierarchy is essential for navigating the workspace structure using the CLI commands.

## Prerequisites

Before invoking any command, verify the following three requirements are met:

1. **clickup-tool CLI installed** -- Install globally with `uv tool install /path/to/clickup-tool` or run from the project directory with `uv run python -m clickup_tool`.
2. **CLICKUP_API_TOKEN set** -- The API token must be available as an environment variable. It can be set in the shell environment or placed in a `.env` file (searched in the current working directory first, then `~/.config/clickup-tool/.env`). The token format is `pk_xxx`. Never store the token in `config.yaml`.
3. **config.yaml configured** -- The configuration file must contain at minimum the `team_id` (workspace ID). Search order: `--config` CLI flag, then `./config.yaml` in the current directory, then `~/.config/clickup-tool/config.yaml`.

If prerequisites are missing, the tool returns a JSON error object explaining what is needed.

## Decision Tree

Map user requests to the appropriate command based on intent:

| User Intent | Command |
|---|---|
| "My tasks", "what am I working on", task overview, current sprint | `clickup-tool get-tasks` |
| Task details, specific task ID, full description, checklists | `clickup-tool get-task TASK_ID` |
| Comments, discussion, conversation on a task | `clickup-tool get-comments TASK_ID` |
| Workspace structure, list spaces | `clickup-tool get-spaces` |
| Folders in a space, project structure | `clickup-tool get-folders SPACE_ID` |
| Lists in a folder, sprints, iterations | `clickup-tool get-lists FOLDER_ID` |
| Team members, workspace users, find user ID, "who is on the team" | `clickup-tool get-members` |
| Auth check, "who am I", verify connection | `clickup-tool get-me` |
| Post a comment, leave a note on task | `clickup-tool add-comment TASK_ID "text"` |
| Log time, track work, "I spent 2h on this" | `clickup-tool add-time-entry TASK_ID "2h" --description "work"` |
| Remove wrong time entry | `clickup-tool delete-time-entry TIMER_ID` |

For exploring workspace structure, chain commands in sequence: start with `get-spaces` to obtain space IDs, then `get-folders SPACE_ID` for folder IDs, then `get-lists FOLDER_ID` to see lists (sprints) within a folder.

When the user mentions a task by name but not by ID, first run `get-tasks` with appropriate filters to locate the task ID, then use `get-task TASK_ID` for full details.

When the user asks about "sprints" or "iterations", map this to the List level of the hierarchy. Run `get-lists FOLDER_ID` to see sprint names, task counts, and due dates. To see tasks within a sprint, run `get-tasks --list-id LIST_ID`.

When the user asks about "project structure" or "board overview", start with `get-spaces` and drill down through folders and lists to present the full hierarchy.

## ClickUp Concepts Mapping

Map common project management terms to ClickUp hierarchy and CLI commands:

| User says | ClickUp concept | CLI approach |
|-----------|----------------|--------------|
| "sprint", "iteration" | List | `get-lists FOLDER_ID` to find it, `get-tasks --list-id LIST_ID` for tasks |
| "project", "board" | Folder | `get-folders SPACE_ID` |
| "workspace", "team" | Space | `get-spaces` |
| "status" | Status (per-space) | Must use exact name from `get-spaces` (e.g., "in progress", NOT "active") |
| "tag", "label" | Tag | `get-tags SPACE_ID` for available tags, `get-tasks --tag TAG` to filter |
| "Jane", person's name | Assignee (numeric ID) | `get-members` to list all workspace users with IDs, or check `assignees[].id` in task output |

**Critical**: Status names must be used exactly as returned by `get-spaces` — do NOT guess. All IDs (assignee, list, space) are **numeric** — never use names as IDs.

## CLI Syntax Rules

All filter flags use `--flag VALUE` syntax. Do NOT use `key:value` syntax.

**Correct:**

```bash
clickup-tool get-tasks --status "in progress" --tag "frontend"
```

**Wrong:**

```bash
# These will NOT work:
clickup-tool get-tasks status:active sprint:A       # wrong syntax
clickup-tool get-tasks --status active               # wrong status name — use exact from get-spaces
clickup-tool get-tasks --assignee Jane               # wrong — must be numeric ID, not name
clickup-tool get-tasks 900000000003                   # wrong — positional arg, need --list-id
```

Flag values with spaces must be quoted: `--status "in progress"`. All IDs (assignee, list, space) are **numeric**.

## Command Quick Reference

| Command | Arguments | Returns |
|---------|-----------|---------|
| `clickup-tool get-me` | -- | User `{id, username, email}` |
| `clickup-tool get-spaces` | -- | Spaces `[{id, name, archived, statuses: [{status, color}]}]` |
| `clickup-tool get-members` | -- | Members `[{id, username, email}]` |
| `clickup-tool get-folders SPACE_ID` | space_id | Folders `[{id, name, task_count}]` |
| `clickup-tool get-lists FOLDER_ID` | folder_id | Lists `[{id, name, task_count, due_date}]` |
| `clickup-tool get-tags SPACE_ID` | space_id | Tag names `["bug", "feature", ...]` |
| `clickup-tool get-tasks` | see filter flags below | Tasks `[{id, name, status, tags, priority, ...}]` |
| `clickup-tool get-task TASK_ID` | task_id | Full task with description, checklists, time_entries |
| `clickup-tool get-comments TASK_ID` | task_id | Comments `[{text, user, date, replies}]` |
| `clickup-tool add-comment TASK_ID TEXT` | task_id, text, `--notify-all` | `{id, date}` |
| `clickup-tool add-time-entry TASK_ID DURATION` | task_id, duration, `--description`, `--date` | `{id, task_id, duration, description}` |
| `clickup-tool delete-time-entry TIMER_ID` | timer_id | `{id, deleted: true}` |

### get-tasks filter flags

| Flag | Type | Description |
|------|------|-------------|
| `--assignee ID` | **numeric user ID** (e.g., `100000001`) | Filter by assignee. NOT a name — use numeric ID from `get-members` to find any user's ID, or from `assignees[].id` in task output |
| `--status STATUS` | **exact string** from `get-spaces` statuses (e.g., `"in progress"`) | Filter by status. Must match exactly |
| `--tag TAG` | **exact string** from `get-tags` (e.g., `"bug"`) | Filter by tag name |
| `--list-id ID` | **numeric list ID** from `get-lists` (e.g., `900000000001`) | Restrict to specific sprint/list |
| `--space-id ID` | **numeric space ID** from `get-spaces` (e.g., `12345678`) | Restrict to specific space |
| `--include-closed` | flag (no value) | Include tasks in closed/done statuses |

All flags except `--include-closed` are repeatable: `--status "in progress" --status "review"`.

When `--assignee` is omitted, falls back to `default_assignee` from config.yaml. When `--space-id` is omitted, falls back to `defaults.space_ids` from config.yaml.

## get-tasks vs get-task

These two commands serve different purposes. Understand the distinction to avoid unnecessary API calls.

**get-tasks** returns a compact overview (a map of all matching tasks). It includes: `id`, `name`, `status`, `assignees`, `custom_fields`, `tags`, `priority`, `due_date`, `start_date`, `date_created`, `date_updated`, `time_estimate`, `parent`, `linked_tasks`, `dependencies`, `list`. It deliberately excludes description, checklists, watchers, attachments, time entries, and URL to keep the response small.

**get-task** returns full details for a single task. In addition to everything in get-tasks, it includes: `markdown_description`, `creator`, `checklists` (with items and resolution status), `attachments` (with title, url, extension, size), `time_spent`, `time_entries` (fetched via a separate API call to `/task/{id}/time`), `url`, and `watchers` (if enabled in config).

Use `get-tasks` first to survey the landscape and identify relevant task IDs. Then use `get-task TASK_ID` when the user needs to read the full description, check checklist progress, review attachments, or examine time tracking data. This two-step approach minimizes API calls while providing complete information on demand.

Common workflow pattern: run `get-tasks` to find the task, note the `id`, then run `get-task TASK_ID` for details, and optionally `get-comments TASK_ID` to see discussion history.

## Output Format

All output is JSON printed to stdout. Warnings and diagnostics go to stderr. Parse command output with standard JSON tools. The exit code is 0 on success, 1 on error.

The output is heavily normalized and compressed compared to raw ClickUp API responses. The following transformations are applied automatically:

- **Dates**: Millisecond timestamps converted to `"YYYY-MM-DD HH:MM"` in the configured timezone.
- **Time durations**: `time_estimate` and `time_spent` converted from milliseconds to `"Xh Ym"` format (e.g., `"6h 30m"`).
- **Users**: Assignees and creators simplified to `{id, username, initials, email}`. Comment users simplified to username string only.
- **Status**: Simplified to `{id, status}`.
- **Tags**: Converted from objects to plain string list. Access directly: `task["tags"]` returns `["bug", "critical"]`, NOT objects. No `.get("name")` needed.
- **Custom fields**: Filtered by name whitelist, dropdown values resolved to human-readable names, `type_config` metadata stripped.
- **Checklists**: Simplified to `{name, resolved, unresolved, items: [{name, resolved}]}`.
- **Attachments**: Simplified to `{title, url, extension, size}`.
- **Empty fields**: Fields with `null` or `[]` values are stripped when `strip_empty: true` (the default).
- **List reference**: Full list objects simplified to `{id, name}`, stripping access metadata.
- **Field filtering**: Each command has a configurable whitelist of fields to return. Only whitelisted fields appear in the output; all others are silently dropped.

Example compressed output for `get-tasks`:

```json
[
  {
    "id": "abc123",
    "name": "Implement login page",
    "status": {"id": "s1", "status": "in progress"},
    "assignees": [{"id": "100000001", "username": "john.doe", "initials": "JD", "email": "john@example.com"}],
    "tags": ["frontend", "sprint-12"],
    "priority": {"id": "2", "priority": "high"},
    "due_date": "2026-03-01 18:00",
    "time_estimate": "6h 30m",
    "list": {"id": "901234", "name": "Sprint 12"}
  }
]
```

## Error Handling

Errors are returned as JSON objects with a consistent structure:

```json
{"error": true, "message": "Token invalid", "status_code": 401}
```

Retry behavior is automatic and transparent:

- **429 (Rate Limited)**: Retried using the `X-RateLimit-Reset` header value. Up to 3 retries.
- **5xx (Server Error)**: Retried with exponential backoff (1s, 2s, 4s). Up to 3 retries.
- **4xx (Client Error)**: Returned immediately without retry. Check the message for details (e.g., invalid task ID, unauthorized).

When a command returns an error, inspect the `message` and `status_code` fields to determine the cause. Common error scenarios:

- **401 Unauthorized** -- API token is expired, invalid, or missing. Verify that `CLICKUP_API_TOKEN` is set correctly. Run `clickup-tool get-me` to test authentication.
- **404 Not Found** -- Task ID, space ID, or folder ID does not exist or is not accessible with the current token. Verify the ID is correct.
- **400 Bad Request** -- Often caused by a misconfigured `team_id` in config.yaml. Verify the workspace ID.
- **Network errors** -- Connection timeouts or DNS failures are retried with exponential backoff before returning the error.

## Safety Limits

The tool enforces pagination safety limits to prevent runaway requests:

- **Tasks**: Maximum 100 pages (10,000 tasks per query).
- **Comments**: Maximum 200 pages (5,000 comments per task).

These limits are sufficient for any realistic workspace. If results appear truncated, narrow the query with filters.

## Common Invocation Patterns

Below are ready-to-use invocation patterns for the most frequent scenarios.

**Show all active tasks for the default user:**

```bash
clickup-tool get-tasks
```

**Show tasks in a specific sprint/list:**

```bash
clickup-tool get-tasks --list-id 900000000001
```

**Show tasks filtered by status:**

```bash
clickup-tool get-tasks --status "in progress" --status "review"
```

**Get full details of a specific task:**

```bash
clickup-tool get-task abc123def
```

**Read comments and replies on a task:**

```bash
clickup-tool get-comments abc123def
```

**Explore workspace hierarchy (spaces, then folders, then lists):**

```bash
clickup-tool get-spaces
clickup-tool get-folders 12345678
clickup-tool get-lists 87654321
```

**Verify API token and connection:**

```bash
clickup-tool get-me
```

## Status and Tag Discovery

Before filtering by status or tag, discover what values are available:

**Discover statuses per space:**

```bash
clickup-tool get-spaces
```

Output includes `statuses` array per space with exact status names and colors. Use these exact names with `--status`.

**Discover tags in a space:**

```bash
clickup-tool get-tags SPACE_ID
```

Returns a plain list of tag name strings. Use these exact names with `--tag`.

## Sprint Task Lookup Workflow

To find tasks in a specific sprint, follow these steps in order:

1. `clickup-tool get-spaces` — find the space ID
2. `clickup-tool get-folders SPACE_ID` — find the folder (project)
3. `clickup-tool get-lists FOLDER_ID` — find the list (sprint) by name
4. `clickup-tool get-tasks --list-id LIST_ID` — get all tasks in that sprint

Each step uses the ID from the previous step's output.

## Interpreting Output

When presenting task data to users, keep the following in mind:

- The `status` field contains a normalized `{id, status}` object. Use the `status` string value (e.g., "in progress", "review", "done") for display.
- The `priority` field contains priority metadata. The `priority` string value within it (e.g., "urgent", "high", "normal", "low") is the human-readable label.
- Custom fields have already been resolved: dropdown values show the option name (not the raw integer index), and label fields show the label strings (not UUIDs).
- Time values like `time_estimate` and `time_spent` are formatted as `"Xh Ym"` (e.g., "6h 30m", "45m", "2h"). No further conversion is needed.
- Dates are formatted as `"YYYY-MM-DD HH:MM"` in the configured timezone. Present them as-is.
- The `list` field on each task indicates which sprint/list the task belongs to. Use `list.name` to show the sprint name.

## Additional Resources

For complete details beyond this overview, consult the reference documents:

- **`references/commands.md`** -- Full command reference with all arguments, flags, example invocations, and example normalized JSON output for every command.
- **`references/config-guide.md`** -- Configuration setup guide covering config.yaml structure, .env file placement, how to obtain the API token, and how to find the workspace team_id.

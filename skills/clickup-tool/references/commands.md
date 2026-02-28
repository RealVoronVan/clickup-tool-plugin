# ClickUp Tool -- Command Reference

Complete reference for all fourteen clickup-tool commands (10 read + 4 write). Each section includes syntax, arguments, example invocation, normalized output example, and usage notes.

---

## get-me

Verify authentication and retrieve the current user profile.

### Syntax

```
clickup-tool get-me
```

### Arguments

None.

### Example Invocation

```bash
clickup-tool get-me
```

### Example Output

```json
{
  "user": {
    "id": 100000001,
    "username": "john.doe",
    "email": "john@example.com"
  }
}
```

### Notes

- Use this command to verify that the API token is valid and the connection works.
- The response is wrapped in a `"user"` key matching the ClickUp API structure.
- Field filtering is controlled by `fields.get_me` in config.yaml. Default fields: `id`, `username`, `email`.
- Additional fields available (disabled by default): `color`, `profilePicture`, `initials`, `week_start_day`, `global_font_support`, `timezone`.

---

## get-spaces

List all spaces in the configured workspace.

### Syntax

```
clickup-tool get-spaces
```

### Arguments

None. Uses `team_id` from config.yaml.

### Example Invocation

```bash
clickup-tool get-spaces
```

### Example Output

```json
{
  "spaces": [
    {
      "id": "12345678",
      "name": "Engineering",
      "archived": false,
      "statuses": [
        {"status": "to do", "color": "#d3d3d3"},
        {"status": "in progress", "color": "#4194f6"},
        {"status": "review", "color": "#A875FF"},
        {"status": "complete", "color": "#6bc950"}
      ]
    },
    {
      "id": "12345679",
      "name": "Design",
      "archived": false,
      "statuses": [
        {"status": "open", "color": "#d3d3d3"},
        {"status": "done", "color": "#6bc950"}
      ]
    }
  ]
}
```

### Notes

- Requires `team_id` to be configured in config.yaml. Returns an error if missing or set to the placeholder value.
- Field filtering is controlled by `fields.get_spaces` in config.yaml. Default fields: `id`, `name`, `archived`, `statuses`.
- Statuses are normalized to `{status, color}` -- only the status name and color are retained. Use exact status names from this output with `get-tasks --status`.
- Additional fields available (disabled by default): `color`, `private`, `avatar`, `admin_can_manage`, `multiple_assignees`, `features` (~1.3 KB per space), `members` (~3.5 KB for 27 users).
- Use the returned `id` values as input for `get-folders SPACE_ID`.

---

## get-members

List all members of the configured workspace. Returns normalized user objects with numeric IDs needed for `--assignee` filtering.

### Syntax

```
clickup-tool get-members
```

### Arguments

None. Uses `team_id` from config.yaml.

### Example Invocation

```bash
clickup-tool get-members
```

### Example Output

```json
[
  {"id": 100000001, "username": "John Doe", "email": "john@example.com"},
  {"id": 100000002, "username": "Jane Smith", "email": "jane@example.com"},
  {"id": 100000003, "username": "Alex Designer", "email": "alex@example.com"}
]
```

### Notes

- Requires `team_id` to be configured in config.yaml.
- Each member includes a numeric `id` — use this value with `get-tasks --assignee ID`.
- Field filtering is controlled by `fields.get_members` in config.yaml. Default fields: `id`, `username`, `email`.
- Additional fields available (disabled by default): `initials`, `role`, `color`.
- To find a specific person's ID by name, run `get-members` and look for their username.

---

## get-folders

List all folders within a specific space.

### Syntax

```
clickup-tool get-folders SPACE_ID
```

### Arguments

| Argument | Required | Description |
|----------|----------|-------------|
| `SPACE_ID` | Yes | The space ID (from `get-spaces` output) |

### Example Invocation

```bash
clickup-tool get-folders 12345678
```

### Example Output

```json
{
  "folders": [
    {
      "id": "87654321",
      "name": "Backend",
      "task_count": "42",
      "archived": false
    },
    {
      "id": "87654322",
      "name": "Frontend",
      "task_count": "28",
      "archived": false
    }
  ]
}
```

### Notes

- Field filtering is controlled by `fields.get_folders` in config.yaml. Default fields: `id`, `name`, `task_count`, `archived`.
- Additional fields available (disabled by default): `lists` (~48 KB with nested list statuses), `statuses` (~1.4 KB), `space` (252 B reference), `orderindex`, `override_statuses`, `hidden`, `permission_level`.
- Use the returned `id` values as input for `get-lists FOLDER_ID`.

---

## get-lists

List all lists (often used as sprints) within a specific folder.

### Syntax

```
clickup-tool get-lists FOLDER_ID
```

### Arguments

| Argument | Required | Description |
|----------|----------|-------------|
| `FOLDER_ID` | Yes | The folder ID (from `get-folders` output) |

### Example Invocation

```bash
clickup-tool get-lists 87654321
```

### Example Output

```json
{
  "lists": [
    {
      "id": "900000000001",
      "name": "Sprint 12",
      "task_count": "15",
      "due_date": "2026-03-14 18:00"
    },
    {
      "id": "900000000002",
      "name": "Sprint 13",
      "task_count": "8",
      "due_date": "2026-03-28 18:00"
    }
  ]
}
```

### Notes

- Dates are normalized from millisecond timestamps to `"YYYY-MM-DD HH:MM"` format using the configured timezone.
- Field filtering is controlled by `fields.get_lists` in config.yaml. Default fields: `id`, `name`, `task_count`, `archived`, `due_date`, `start_date`.
- Additional fields available (disabled by default): `folder` (320 B reference), `space` (232 B reference), `status`, `priority`, `assignee`, `orderindex`, `content`, `override_statuses`, `permission_level`.
- Use the returned `id` values with `get-tasks --list-id LIST_ID` to fetch tasks from a specific list.

---

## get-tags

List all tags defined in a specific space. Returns a plain list of tag name strings.

### Syntax

```
clickup-tool get-tags SPACE_ID
```

### Arguments

| Argument | Required | Description |
|----------|----------|-------------|
| `SPACE_ID` | Yes | The space ID (from `get-spaces` output) |

### Example Invocation

```bash
clickup-tool get-tags 12345678
```

### Example Output

```json
["bug", "critical", "feature", "A-team"]
```

### Notes

- Returns a simple JSON array of tag name strings, not objects.
- Use the returned tag names as filter values: `get-tasks --tag "bug"`.
- Tags are space-scoped: different spaces may have different tags.

---

## get-tasks

Fetch tasks from the workspace with optional filters. Returns a compact overview suitable for scanning many tasks at once.

### Syntax

```
clickup-tool get-tasks [--assignee ID] [--status STATUS] [--tag TAG] [--list-id ID] [--space-id ID] [--include-closed]
```

### Arguments

| Flag | Required | Repeatable | Description |
|------|----------|------------|-------------|
| `--assignee ID` | No | Yes | **Numeric user ID** (e.g., `100000001`). NOT a name — find ID from `get-me` or `assignees[].id` in task output. Falls back to `defaults.default_assignee` from config.yaml if omitted. |
| `--status STATUS` | No | Yes | **Exact status string** from `get-spaces` output (e.g., `"in progress"`, `"review"`). Case-sensitive. Do not guess. |
| `--tag TAG` | No | Yes | **Exact tag string** from `get-tags` output (e.g., `"bug"`). |
| `--list-id ID` | No | Yes | **Numeric list ID** from `get-lists` output. Restricts results to specific lists/sprints. |
| `--space-id ID` | No | Yes | **Numeric space ID** from `get-spaces` output. Falls back to `defaults.space_ids` from config.yaml if omitted. |
| `--include-closed` | No | No | Include tasks with closed status. Disabled by default. Falls back to `defaults.include_closed` from config.yaml. |

All repeatable flags accept multiple values. Example: `--status "in progress" --status "review"` matches tasks in either status.

### Example Invocations

Fetch tasks assigned to the default user:

```bash
clickup-tool get-tasks
```

Fetch tasks by specific assignee with status filter:

```bash
clickup-tool get-tasks --assignee 100000001 --status "in progress"
```

Fetch tasks in a specific list including closed:

```bash
clickup-tool get-tasks --list-id 900000000001 --include-closed
```

Fetch tasks with a specific tag:

```bash
clickup-tool get-tasks --tag "frontend" --tag "urgent"
```

### Example Output

```json
[
  {
    "id": "abc123def",
    "name": "Implement login page",
    "status": {"id": "sc901", "status": "in progress"},
    "assignees": [
      {"id": "100000001", "username": "john.doe", "initials": "JD", "email": "john@example.com"}
    ],
    "custom_fields": [
      {"name": "Product Order", "type": "number", "value": 5},
      {"name": "Actual Effort", "type": "number", "value": 12}
    ],
    "tags": ["frontend", "sprint-12"],
    "priority": {"id": "2", "priority": "high", "color": "#f9d900", "orderindex": "2"},
    "due_date": "2026-03-01 18:00",
    "start_date": "2026-02-15 09:00",
    "date_created": "2026-02-10 14:30",
    "date_updated": "2026-02-24 16:45",
    "time_estimate": "6h 30m",
    "parent": "abc100xyz",
    "list": {"id": "900000000001", "name": "Sprint 12"}
  }
]
```

### Output Fields

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | Task ID |
| `name` | string | Task name |
| `status` | object | `{id, status}` -- normalized status |
| `assignees` | array | `[{id, username, initials, email}]` -- normalized user objects |
| `custom_fields` | array | `[{name, type, value}]` -- filtered by whitelist, values resolved |
| `tags` | array | List of tag name strings (not objects) |
| `priority` | object | Priority with id, priority name, color |
| `due_date` | string | `"YYYY-MM-DD HH:MM"` or null |
| `start_date` | string | `"YYYY-MM-DD HH:MM"` or null |
| `date_created` | string | `"YYYY-MM-DD HH:MM"` |
| `date_updated` | string | `"YYYY-MM-DD HH:MM"` |
| `time_estimate` | string | `"Xh Ym"` format or null |
| `parent` | string | Parent task ID or null |
| `linked_tasks` | array | Linked task references |
| `dependencies` | array | Task dependency references |
| `list` | object | `{id, name}` -- the list (sprint) containing this task |

### Notes

- Pagination is automatic: the tool fetches up to 100 pages (10,000 tasks) with deduplication.
- Subtasks are included by default (controlled by `defaults.subtasks` in config.yaml).
- Custom fields are filtered by the `fields.get_tasks_custom_fields` whitelist in config.yaml. Falls back to `fields.custom_fields` if the command-specific key is absent.
- Dropdown custom field values are automatically resolved to human-readable option names. Label-type fields are resolved to lists of label strings.
- Fields with null or empty array values are stripped when `strip_empty: true` (the default).
- This command does NOT return: `markdown_description`, `checklists`, `attachments`, `time_spent`, `time_entries`, `url`, `creator`, `watchers`. Use `get-task TASK_ID` for those.

---

## get-task

Fetch full details for a single task, including description, checklists, attachments, and time tracking entries.

### Syntax

```
clickup-tool get-task TASK_ID
```

### Arguments

| Argument | Required | Description |
|----------|----------|-------------|
| `TASK_ID` | Yes | The task ID (from `get-tasks` output or ClickUp URL) |

### Example Invocation

```bash
clickup-tool get-task abc123def
```

### Example Output

```json
{
  "id": "abc123def",
  "name": "Implement login page",
  "status": {"id": "sc901", "status": "in progress"},
  "markdown_description": "## Requirements\n\n- OAuth2 integration\n- Session management\n- Error handling for invalid credentials",
  "assignees": [
    {"id": "100000001", "username": "john.doe", "initials": "JD", "email": "john@example.com"}
  ],
  "creator": {"id": "100000002", "username": "jane.smith", "initials": "JS", "email": "jane@example.com"},
  "checklists": [
    {
      "name": "Implementation Steps",
      "resolved": 2,
      "unresolved": 3,
      "items": [
        {"name": "Set up OAuth2 client", "resolved": true},
        {"name": "Create login form UI", "resolved": true},
        {"name": "Add session middleware", "resolved": false},
        {"name": "Write unit tests", "resolved": false},
        {"name": "Update API docs", "resolved": false}
      ]
    }
  ],
  "custom_fields": [
    {"name": "Product Order", "type": "number", "value": 5},
    {"name": "Complexity", "type": "drop_down", "value": "Medium"},
    {"name": "Actual Effort", "type": "number", "value": 12},
    {"name": "Version", "type": "labels", "value": ["v2.1"]}
  ],
  "attachments": [
    {"title": "mockup.png", "url": "https://attachments.clickup.com/...", "extension": "png", "size": 245760}
  ],
  "tags": ["frontend", "sprint-12"],
  "priority": {"id": "2", "priority": "high", "color": "#f9d900", "orderindex": "2"},
  "due_date": "2026-03-01 18:00",
  "start_date": "2026-02-15 09:00",
  "date_created": "2026-02-10 14:30",
  "date_updated": "2026-02-24 16:45",
  "time_estimate": "6h 30m",
  "time_spent": "4h 15m",
  "time_entries": [
    {"user": "john.doe", "duration": "2h 30m", "date": "2026-02-20 10:00"},
    {"user": "john.doe", "duration": "1h 45m", "date": "2026-02-22 14:00"}
  ],
  "url": "https://app.clickup.com/t/abc123def",
  "list": {"id": "900000000001", "name": "Sprint 12"}
}
```

### Additional Fields (compared to get-tasks)

| Field | Type | Description |
|-------|------|-------------|
| `markdown_description` | string | Full task description in Markdown format |
| `creator` | object | `{id, username, initials, email}` -- task creator |
| `checklists` | array | `[{name, resolved, unresolved, items: [{name, resolved}]}]` |
| `attachments` | array | `[{title, url, extension, size}]` |
| `time_spent` | string | Total tracked time in `"Xh Ym"` format |
| `time_entries` | array | `[{user, duration, date}]` -- individual time tracking entries |
| `url` | string | Direct link to the task in ClickUp web UI |

### Notes

- This command makes two API calls: one for the task data (`GET /task/{id}`) and one for time entries (`GET /task/{id}/time`).
- Time entries are fetched from a separate endpoint and merged into the response as `time_entries`.
- Custom fields are filtered by the `fields.get_task_custom_fields` whitelist in config.yaml. Falls back to `fields.custom_fields` if the command-specific key is absent.
- The `markdown_description` field contains the task description in Markdown. Use this over `description` or `text_content` which are duplicate fields excluded by default.
- Checklist items include a `resolved` boolean. The checklist itself includes `resolved` and `unresolved` counts for a quick summary.
- If the task ID is invalid or the task is not accessible, the command returns an error with status_code 404.

---

## get-comments

Fetch all comments on a task, including threaded replies.

### Syntax

```
clickup-tool get-comments TASK_ID
```

### Arguments

| Argument | Required | Description |
|----------|----------|-------------|
| `TASK_ID` | Yes | The task ID |

### Example Invocation

```bash
clickup-tool get-comments abc123def
```

### Example Output

```json
[
  {
    "text": "Started working on the OAuth2 integration. Should have a PR ready by Wednesday.",
    "user": "john.doe",
    "date": "2026-02-20 10:30",
    "resolved": false,
    "replies": [
      {
        "text": "Sounds good. Make sure to add refresh token handling.",
        "user": "jane.smith",
        "date": "2026-02-20 11:15",
        "resolved": false
      }
    ]
  },
  {
    "text": "Here is the design mockup for the login form.",
    "user": "alex.designer",
    "date": "2026-02-18 14:00",
    "resolved": false,
    "attachments": [
      {"title": "login-mockup.png", "url": "https://attachments.clickup.com/...", "extension": "png", "size": 189440}
    ]
  }
]
```

### Output Fields

| Field | Type | Description |
|-------|------|-------------|
| `text` | string | Plain text comment content (rich text dropped) |
| `user` | string | Username of the comment author (string, not object) |
| `date` | string | `"YYYY-MM-DD HH:MM"` |
| `resolved` | boolean | Whether the comment is resolved |
| `attachments` | array | Extracted from rich text: images, files, bookmarks `[{title, url, extension, size}]` |
| `replies` | array | Nested comments with the same structure (recursive) |

### Notes

- Pagination is automatic: the tool fetches up to 200 pages (5,000 comments) using cursor-based pagination.
- Replies are fetched via separate API calls (`GET /comment/{id}/reply`) for each comment that has a non-zero `reply_count`.
- Rich text `comment` array is dropped entirely. The `comment_text` plain text field is used as `text`.
- Attachments, images, and bookmarks embedded in the rich text comment parts are extracted into the `attachments` array.
- Comment users are simplified to a username string (unlike task users which retain the full `{id, username, initials, email}` object).
- Field filtering is controlled by `fields.get_comments` in config.yaml. By default, `id` is excluded.
- If a reply fetch fails, a warning is printed to stderr and an empty replies array is returned for that comment.

---

## add-comment

Post a comment on a task.

### Syntax

```
clickup-tool add-comment TASK_ID TEXT [--notify-all]
```

### Arguments

| Argument | Required | Description |
|----------|----------|-------------|
| `TASK_ID` | Yes | The task ID |
| `TEXT` | Yes | Comment text (quote if it contains spaces) |
| `--notify-all` | No | Notify all assignees |

### Example Invocation

```bash
clickup-tool add-comment abc123def "Started working on the OAuth2 integration"
```

### Example Output

```json
{"id": "458", "hist_id": "26508", "date": "2026-02-26 15:30"}
```

### Notes

- The `date` field is normalized from milliseconds to "YYYY-MM-DD HH:MM".
- Returns the raw API response with date normalization applied.
- `--notify-all` sends email notifications to all task assignees.

---

## add-time-entry

Log a time tracking entry on a task.

### Syntax

```
clickup-tool add-time-entry TASK_ID DURATION [--description TEXT] [--date YYYY-MM-DD]
```

### Arguments

| Argument | Required | Description |
|----------|----------|-------------|
| `TASK_ID` | Yes | The task ID |
| `DURATION` | Yes | Duration: `"2h 30m"`, `"45m"`, or `"150"` (bare number = minutes) |
| `--description` | No | Description of the work performed |
| `--date` | No | Date for the entry (YYYY-MM-DD, defaults to today) |

### Example Invocations

```bash
clickup-tool add-time-entry abc123def "2h 30m" --description "OAuth2 implementation"
clickup-tool add-time-entry abc123def "45m" --date 2026-02-25
clickup-tool add-time-entry abc123def 90
```

### Example Output

```json
{"data": {"id": "12345", "task": {"id": "abc123def"}, "duration": 9000000, "description": "OAuth2 implementation", "start": "1740567600000"}}
```

### Notes

- Requires `team_id` configured in config.yaml.
- Duration is parsed from human format and sent as milliseconds to the API.
- The `--date` flag computes midnight of the given date in the configured timezone.
- Without `--date`, the start time is the current timestamp.
- Uses the team-scoped endpoint `POST /team/{team_id}/time_entries` (not the legacy per-task endpoint).
- The response `data` wrapper contains the raw API response. Duration is in milliseconds.

---

## delete-time-entry

Delete a time tracking entry.

### Syntax

```
clickup-tool delete-time-entry TIMER_ID
```

### Arguments

| Argument | Required | Description |
|----------|----------|-------------|
| `TIMER_ID` | Yes | The time entry ID (from `get-task` output's `time_entries`) |

### Example Invocation

```bash
clickup-tool delete-time-entry 12345
```

### Example Output

```json
{"data": {"id": "12345", "duration": 3600000}}
```

### Notes

- Requires `team_id` configured in config.yaml.
- This action is irreversible — the time entry cannot be recovered.
- Use `get-task TASK_ID` to see time entry IDs before deleting.
- Uses the team-scoped endpoint `DELETE /team/{team_id}/time_entries/{timer_id}`.

---

## get-statuses

List valid statuses for a task. Resolves statuses at the list level (not space level) by chaining two API calls: fetches the task to find its list, then fetches the list to get its statuses.

### Syntax

```
clickup-tool get-statuses TASK_ID
```

### Arguments

| Argument | Required | Description |
|----------|----------|-------------|
| `TASK_ID` | Yes | The task ID to look up statuses for |

### Example Invocation

```bash
clickup-tool get-statuses abc123def
```

### Example Output

```json
["to do", "in progress", "pending approval", "on hold", "completed", "done"]
```

### Notes

- Returns a plain JSON list of status name strings — no objects, no colors.
- Statuses are resolved from the task's **list**, not the space. Lists can override space-level defaults.
- Internally chains `GET /task/{id}` (to get `list.id`) then `GET /list/{list_id}` (to get `statuses`).
- Use this before `set-status` to discover valid status names.

---

## set-status

Set a task's status.

### Syntax

```
clickup-tool set-status TASK_ID "STATUS"
```

### Arguments

| Argument | Required | Description |
|----------|----------|-------------|
| `TASK_ID` | Yes | The task ID to update |
| `STATUS` | Yes | Exact status name (must match a valid status for the task's list) |

### Example Invocation

```bash
clickup-tool set-status abc123def "in progress"
```

### Example Output (success)

```json
{
  "id": "abc123def",
  "name": "Implement login page",
  "status": {"id": "s2", "status": "in progress"},
  "assignees": [{"id": "100000001", "username": "john.doe", "initials": "JD", "email": "john@example.com"}],
  "tags": ["frontend"],
  "list": {"id": "901234", "name": "Sprint 12"}
}
```

### Example Output (invalid status)

```json
{
  "error": true,
  "message": "Status \"in reveiw\" is not valid. Available: to do, in progress, pending approval, on hold, completed, done"
}
```

### Notes

- Uses `PUT /task/{task_id}` with `{"status": "..."}` body.
- Status name must be an **exact string** — use `get-statuses TASK_ID` first to discover valid names.
- On success, returns the updated task normalized like `get-task` (without `time_entries`).
- On invalid status (400), fetches and shows available statuses in the error message.
- On other errors (404, etc.), returns the API error as-is.

---

## Normalization Summary

All commands apply the following transformations automatically:

| Transformation | Input | Output | Applies To |
|----------------|-------|--------|------------|
| Date conversion | `"1708612200000"` (ms) | `"2026-02-22 14:30"` | All date fields |
| Time duration | `23400000` (ms) | `"6h 30m"` | `time_estimate`, `time_spent` |
| User simplification | Full user object (~500 B) | `{id, username, initials, email}` | Task assignees, creator, watchers |
| Comment user | Full user object | `"username"` string | Comment authors |
| Status | Full status object | `{id, status}` | Task status |
| Tags | `[{name: "tag1", ...}]` | `["tag1"]` | Task tags |
| Custom fields | Full CF with type_config | `{name, type, value}` with resolved values | Tasks |
| Checklists | Full checklist objects | `{name, resolved, unresolved, items}` | Tasks |
| Attachments | Full attachment objects | `{title, url, extension, size}` | Tasks, comments |
| Space statuses | Full status objects | `{status, color}` | Spaces |
| Space tags | `[{name: "tag", ...}]` | `["tag"]` | `get-tags` |
| Member simplification | Full member wrapper with user object | `{id, username, email}` | Workspace members |
| List reference | Full list object | `{id, name}` | Tasks |
| Empty stripping | `{"field": null}` | (field removed) | All, when `strip_empty: true` |
| Field filtering | All available fields | Whitelisted fields only | All commands |

Custom field value resolution:

- **drop_down**: Integer `orderindex` value resolved to the option name string.
- **labels**: List of option IDs resolved to list of label strings.
- **Other types**: Value passed through unchanged (numbers, text, dates, etc.).

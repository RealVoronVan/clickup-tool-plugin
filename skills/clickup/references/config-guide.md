# ClickUp Tool -- Configuration Guide

Complete guide to configuring clickup-tool, including config file placement, API token setup, and all available settings.

---

## Configuration File Search Order

clickup-tool looks for `config.yaml` in the following order, using the first file found:

1. **Explicit path** -- Pass `--config /path/to/config.yaml` on the command line. If the file does not exist at the specified path, a warning is printed to stderr and the tool continues without configuration.
2. **Current working directory** -- `./config.yaml` in the directory where the command is executed.
3. **Global config directory** -- `~/.config/clickup-tool/config.yaml`.

If no config file is found at any location, the tool runs with defaults (which means `team_id` is unset and commands requiring it will fail with an error).

## Environment File Search Order

The `.env` file containing `CLICKUP_API_TOKEN` is searched in this order:

1. **Current working directory** -- `./.env`
2. **Global config directory** -- `~/.config/clickup-tool/.env`

Only the first `.env` file found is loaded. If `CLICKUP_API_TOKEN` is already set in the shell environment, the `.env` file value is ignored (environment variables take precedence over `.env` file values).

## How to Get the API Token

1. Log in to ClickUp at [app.clickup.com](https://app.clickup.com).
2. Navigate to **Settings** (bottom-left avatar) > **Apps**.
3. Under **API Token**, click **Generate** (or copy the existing token).
4. The token format is `pk_` followed by a numeric string, e.g., `pk_12345678_ABCDEFGHIJKLMNOPQRSTUVWXYZ1234`.

Store the token in one of these locations:

**Option A: .env file** (recommended for persistent setup):

```bash
# ~/.config/clickup-tool/.env
CLICKUP_API_TOKEN=pk_12345678_ABCDEFGHIJKLMNOPQRSTUVWXYZ1234
```

**Option B: Shell environment** (for temporary or session-based usage):

```bash
export CLICKUP_API_TOKEN=pk_12345678_ABCDEFGHIJKLMNOPQRSTUVWXYZ1234
```

Never store the token in `config.yaml`. The tool enforces this by reading the token exclusively from the `CLICKUP_API_TOKEN` environment variable.

## How to Get the Team ID (Workspace ID)

The `team_id` is required for `get-tasks` and `get-spaces` commands. Obtain it using one of these methods:

**Method A: From the ClickUp URL.** Open any page in the ClickUp web app. The URL format is `https://app.clickup.com/{team_id}/...`. The numeric segment after the domain is the team ID.

**Method B: From the API.** Run `clickup-tool get-me` to verify auth, then call the ClickUp teams endpoint directly:

```bash
curl -s -H "Authorization: pk_YOUR_TOKEN" https://api.clickup.com/api/v2/team | python -m json.tool
```

The response contains a `teams` array. Each team object has an `id` field -- that is the team_id.

## Full Annotated config.yaml

Below is a complete config.yaml with all available sections and options, annotated with explanations:

```yaml
# ClickUp CLI tool configuration
# Token: set CLICKUP_API_TOKEN env var (never put token here)

# Workspace ID (required for get-tasks and get-spaces)
# Get from ClickUp URL: https://app.clickup.com/{team_id}/...
team_id: "YOUR_TEAM_ID"

defaults:
  # Fallback assignee ID for get-tasks when --assignee is not passed
  # Set to your ClickUp user ID to see your tasks by default
  default_assignee: "100000001"

  # Restrict get-tasks to specific spaces (empty = all spaces)
  space_ids: []

  # Include closed tasks in get-tasks results (overridden by --include-closed flag)
  include_closed: false

  # Include subtasks in get-tasks results
  subtasks: true

  # Remove fields with null or empty array values from output
  strip_empty: true

  # Timezone for date formatting (IANA timezone name)
  # Examples: "UTC", "Asia/Yerevan", "America/New_York", "Europe/London"
  timezone: "UTC"

# Field filters per command
# Only listed fields are included in output
# Comment out a field to exclude it, uncomment to include it
fields:

  # get-me: authenticated user profile
  get_me:
    - id
    - username
    - email
    # - color
    # - profilePicture
    # - initials
    # - week_start_day
    # - global_font_support
    # - timezone

  # get-spaces: workspace spaces
  get_spaces:
    - id
    - name
    - archived
    # - color
    # - private
    # - avatar
    # - admin_can_manage
    # - statuses          # ~1KB per space
    # - multiple_assignees
    # - features          # ~1.3KB per space
    # - members           # ~3.5KB for 27 users

  # get-folders: folders in a space
  get_folders:
    - id
    - name
    - task_count
    - archived
    # - lists              # ~48KB -- nested lists with statuses
    # - statuses           # ~1.4KB
    # - space              # 252B -- space reference
    # - orderindex
    # - override_statuses
    # - hidden
    # - permission_level

  # get-lists: lists (sprints) in a folder
  get_lists:
    - id
    - name
    - task_count
    - archived
    - due_date
    - start_date
    # - folder             # 320B -- folder reference
    # - space              # 232B -- space reference
    # - status
    # - priority
    # - assignee
    # - orderindex
    # - content
    # - override_statuses
    # - permission_level

  # get-task: full single task details
  get_task:
    - id
    - name
    - status
    - markdown_description
    - assignees
    - creator
    - checklists
    - custom_fields
    - attachments
    - tags
    - priority
    - due_date
    - start_date
    - date_created
    - date_updated
    - time_estimate
    - time_spent
    - time_entries
    - url
    - parent
    - linked_tasks
    - dependencies
    - list
    # - watchers
    # - text_content         # duplicate of markdown_description
    # - description          # duplicate of markdown_description
    # - sharing              # ~250B
    # - folder               # duplicate of project
    # - project
    # - space
    # - orderindex
    # - team_id
    # - archived
    # - permission_level
    # - custom_id
    # - custom_item_id
    # - group_assignees
    # - locations
    # - date_closed
    # - date_done
    # - points
    # - top_level_parent

  # get-tasks: task list overview (compact)
  get_tasks:
    - id
    - name
    - status
    - assignees
    - custom_fields
    - tags
    - priority
    - due_date
    - start_date
    - date_created
    - date_updated
    - time_estimate
    - parent
    - linked_tasks
    - dependencies
    - list
    # - markdown_description  # details via get-task
    # - checklists           # details via get-task
    # - url
    # - text_content         # duplicate of markdown_description
    # - description          # duplicate of markdown_description
    # - creator
    # - watchers             # ~400B per task
    # - sharing              # ~250B per task
    # - folder               # duplicate of project
    # - project
    # - space
    # - orderindex
    # - team_id
    # - archived
    # - permission_level
    # - group_assignees
    # - locations
    # - custom_id
    # - custom_item_id
    # - date_closed
    # - date_done
    # - points
    # - top_level_parent

  # get-comments: task comments
  get_comments:
    # - id
    - text
    - user
    - date
    - resolved
    - attachments
    - replies

  # Custom fields whitelists per command (by field name)
  # Values are auto-resolved: drop_down -> option name, labels -> label strings
  # Falls back to shared "custom_fields" key if command-specific key is absent

  # Custom fields for get-task (full detail view)
  get_task_custom_fields:
    - Product Order
    - Complexity
    - Actual Effort
    - Version
    - Resolution Reason
    # - UX Status
    # - Process Status
    # - Pull Request
    # - Delivery Order

  # Custom fields for get-tasks (overview, typically fewer fields)
  get_tasks_custom_fields:
    - Product Order
    - Actual Effort
    - Version
    - Resolution Reason
    # - Complexity
    # - UX Status
    # - Process Status
    # - Pull Request
    # - Delivery Order

  # Shared fallback custom fields whitelist (used if command-specific key is absent)
  # custom_fields:
  #   - Product Order
  #   - Complexity
```

## Configuration Priority

Settings are resolved with the following priority (highest first):

1. **CLI arguments** -- Flags like `--assignee`, `--status`, `--include-closed`, `--config` override everything.
2. **config.yaml values** -- The resolved config file provides team_id, defaults, field filters, and custom field whitelists.
3. **Built-in defaults** -- When neither CLI nor config specifies a value: `strip_empty: true`, `timezone: "UTC"`, `subtasks: true`, `include_closed: false`, `space_ids: []`.

## Field Filter Configuration

The `fields` section controls which fields appear in command output. Each command has its own whitelist key:

| Config Key | Command | Purpose |
|------------|---------|---------|
| `fields.get_me` | get-me | User profile fields |
| `fields.get_spaces` | get-spaces | Space fields |
| `fields.get_folders` | get-folders | Folder fields |
| `fields.get_lists` | get-lists | List fields |
| `fields.get_tasks` | get-tasks | Task overview fields |
| `fields.get_task` | get-task | Full task detail fields |
| `fields.get_comments` | get-comments | Comment fields |

If a command's field list is absent or empty in config.yaml, all fields are returned (no filtering).

### Custom Fields Whitelist

Custom fields have a separate whitelist mechanism. The tool checks for a command-specific key first, then falls back to a shared key:

1. `fields.get_task_custom_fields` -- Used by get-task
2. `fields.get_tasks_custom_fields` -- Used by get-tasks
3. `fields.custom_fields` -- Shared fallback used when the command-specific key is absent

Only custom fields whose `name` matches an entry in the whitelist are included in the output. Custom fields not in the whitelist are silently dropped.

If no custom fields whitelist is configured (neither command-specific nor shared), all custom fields are returned.

## Timezone Configuration

The `defaults.timezone` setting accepts any valid IANA timezone name. All millisecond timestamps in API responses are converted to `"YYYY-MM-DD HH:MM"` format in the specified timezone.

Common timezone values:

| Timezone | UTC Offset | Example |
|----------|------------|---------|
| `UTC` | +00:00 | Default |
| `Asia/Yerevan` | +04:00 | Armenia |
| `Europe/Moscow` | +03:00 | Russia (Moscow) |
| `America/New_York` | -05:00 | US Eastern |
| `Europe/London` | +00:00 | UK |
| `Asia/Tokyo` | +09:00 | Japan |

## Global vs Local Configuration

For users working on multiple projects or switching between workspaces:

- Place the **shared** config at `~/.config/clickup-tool/config.yaml` with general settings (team_id, timezone, default_assignee).
- Place a **project-specific** config at `./config.yaml` in the project directory to override field filters or custom field whitelists for that context.
- The local config takes precedence over the global config (search order applies -- the first file found wins entirely; configs are not merged).

The `.env` file follows the same local-over-global pattern. Place the API token in `~/.config/clickup-tool/.env` for global access, or in `./.env` for project-specific tokens (e.g., different ClickUp workspaces).

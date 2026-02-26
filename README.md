# clickup-tool plugin for Claude Code

ClickUp integration for [Claude Code](https://docs.anthropic.com/en/docs/claude-code) — read and write ClickUp data through natural language. Query tasks, post comments, log time, explore workspace structure. Uses the [clickup-tool](https://github.com/RealVoronVan/clickup-tool) CLI under the hood.

## Prerequisites

- [uv](https://docs.astral.sh/uv/) package manager
- ClickUp API token ([how to get one](#getting-your-api-token))
- Claude Code CLI

## Quick Start

### 1. Install the CLI

```bash
uv tool install git+https://github.com/RealVoronVan/clickup-tool
```

### 2. Configure

```bash
mkdir -p ~/.config/clickup-tool

# Set your API token
echo "CLICKUP_API_TOKEN=pk_YOUR_TOKEN" > ~/.config/clickup-tool/.env

# Download and edit config
curl -o ~/.config/clickup-tool/config.yaml \
  https://raw.githubusercontent.com/RealVoronVan/clickup-tool/main/config.yaml.example
```

Edit `~/.config/clickup-tool/config.yaml` — set `team_id` and `default_assignee`.

### 3. Verify CLI works

```bash
clickup-tool whoami
# Expected: ClickUp: connected as Your Name
```

### 4. Install the plugin

```bash
claude plugins install github:RealVoronVan/clickup-tool-plugin
```

## Getting Your API Token

1. Open ClickUp → Settings (bottom-left) → Apps
2. Click **API Token** → **Generate**
3. Copy the token (starts with `pk_`)

## Getting Your Team ID

Your team ID is in the ClickUp URL: `app.clickup.com/YOUR_TEAM_ID/...`

Or run: `clickup-tool get-me` and note the workspace ID.

## Configuration

Config file: `~/.config/clickup-tool/config.yaml`

```yaml
team_id: "YOUR_TEAM_ID"
defaults:
  default_assignee: "YOUR_USER_ID"  # from clickup-tool get-me
  strip_empty: true
  timezone: "UTC"
```

See [config.yaml.example](https://github.com/RealVoronVan/clickup-tool/blob/main/config.yaml.example) for all options.

## Available Commands

### Read

| Command | Description |
|---------|-------------|
| `/clickup-tool:tasks` | List tasks with filters (assignee, status, tag, list) |
| `/clickup-tool:task TASK_ID` | Full task details with time entries |
| `/clickup-tool:comments TASK_ID` | Task comments with replies |
| `/clickup-tool:spaces` | Workspace spaces and statuses |
| `/clickup-tool:tags SPACE_ID` | Tags in a space |
| `/clickup-tool:members` | Workspace members |
| `/clickup-tool:me` | Authenticated user info |

### Write

| Command | Description |
|---------|-------------|
| `/clickup-tool:add-comment TASK_ID` | Post a comment on a task |
| `/clickup-tool:add-time TASK_ID 2h 30m` | Log time on a task |
| `/clickup-tool:delete-time TIMER_ID` | Delete a time tracking entry |

Write commands show a preview and ask for confirmation before executing. They have full access to your conversation context — you can say things like "summarize the last commit as a comment on task X" or "figure out how long I worked today and log it".

## Skill Auto-Trigger

The plugin includes a skill that activates automatically when you mention ClickUp tasks, sprints, time tracking, or workspace queries in conversation. No slash command needed — just ask naturally:

- "What are my open tasks?"
- "Show me sprint 21 tasks"
- "What comments are on task abc123?"
- "Log 2 hours on task abc123 for the auth refactoring"
- "Post a status update on task abc123"

## Session Hook

On session start, the plugin checks your ClickUp connection and displays status. If you see "not configured", verify your token and config.

## Troubleshooting

**"ClickUp: not configured"**
- Check token: `echo $CLICKUP_API_TOKEN` or verify `~/.config/clickup-tool/.env`
- Check CLI: `clickup-tool whoami`

**"team_id not configured"**
- Edit `~/.config/clickup-tool/config.yaml` and set `team_id`

**Command not found: clickup-tool**
- Install: `uv tool install git+https://github.com/RealVoronVan/clickup-tool`

## License

MIT

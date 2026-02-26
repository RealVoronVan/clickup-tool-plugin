# clickup-tool-plugin

Claude Code plugin for ClickUp task management. Wraps the [clickup-tool](../clickup-tool/) CLI with slash commands, a skill with auto-trigger, and a task analysis agent.

**Related project:** [clickup-tool](../clickup-tool/) — Python CLI that this plugin calls. Both repos live under `~/Projects/clickup/`.

## Project structure

- `.claude-plugin/plugin.json` — plugin manifest (name, version, metadata)
- `skills/clickup/SKILL.md` — main skill (auto-triggers on ClickUp mentions), ~2000 words
- `skills/clickup/references/commands.md` — full CLI reference (9 commands with examples)
- `skills/clickup/references/config-guide.md` — configuration guide
- `commands/` — 10 slash commands: tasks, task, comments, spaces, tags, members, me, add-comment, add-time, delete-time
- `agents/task-analyst.md` — autonomous agent for complex multi-step ClickUp queries
- `hooks/hooks.json` — SessionStart hook (runs `clickup-tool whoami`)
- `LICENSE` — MIT
- `README.md` — public-facing installation and usage guide

## Components

### Skill (`skills/clickup/SKILL.md`)
Auto-triggers when user mentions ClickUp tasks, sprints, or workspace queries. Contains:
- ClickUp hierarchy and concepts mapping
- CLI syntax rules (--flag VALUE, never key:value)
- Command quick reference and filter flags
- get-tasks vs get-task distinction
- Status/tag discovery and sprint lookup workflows

References in `skills/clickup/references/` provide detailed examples — loaded on demand.

### Commands (`commands/*.md`)
Each file is a slash command. Key conventions:
- `allowed-tools: Bash(clickup-tool *)` — restricts to clickup-tool only
- Dynamic data via `!` backtick syntax (e.g., `!`clickup-tool get-spaces``)
- Commands resolve human names to numeric IDs before calling CLI

### Agent (`agents/task-analyst.md`)
For complex queries chaining multiple API calls. Runs in isolated context.

### Hook (`hooks/hooks.json`)
SessionStart: runs `clickup-tool whoami` to display connection status.

## Key conventions

- All content is markdown and JSON — no executable code in this repo
- The CLI (`clickup-tool`) must be installed separately via `uv tool install`
- Config lives at `~/.config/clickup-tool/` (config.yaml + .env)
- Example IDs in docs are sanitized placeholders (12345678, 100000001, etc.)
- No real user data, team IDs, or tokens in any file

## Testing changes

1. Edit files in this repo — symlink at `~/.claude/plugins/clickup-marketplace/clickup` points here
2. Start a new Claude Code session to pick up changes
3. Test slash commands: `/tasks`, `/task TASK_ID`, etc.
4. Test skill auto-trigger: ask about ClickUp tasks in natural language
5. Test hook: new session should show "ClickUp: connected as ..."

## Updating the CLI

If the CLI repo changes, reinstall globally:
```bash
cd ~/Projects/clickup/clickup-tool
uv tool install . --reinstall
```

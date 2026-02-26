---
description: Check ClickUp authentication and show current user info
allowed-tools: Bash(clickup-tool *)
---

Run `clickup-tool get-me` and display the authenticated user information.

If the command fails, suggest checking:
1. `clickup-tool` is installed (`uv tool install /path/to/clickup-tool`)
2. `CLICKUP_API_TOKEN` is set in environment or `~/.config/clickup-tool/.env`

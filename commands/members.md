---
description: List all workspace members (id, username, email)
allowed-tools: Bash(clickup-tool *)
---

Run `clickup-tool get-members` to list all users in the workspace.

Output is a list of `{id, username, email}` objects. Use the numeric `id` when filtering tasks by assignee: `clickup-tool get-tasks --assignee ID`.

If the user asks about a person by name, find their numeric ID in this output and use it for subsequent queries.

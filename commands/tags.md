---
description: List available tags in a ClickUp space
allowed-tools: Bash(clickup-tool *)
argument-hint: "SPACE_ID"
---

Run `clickup-tool get-tags $ARGUMENTS` to list all tags in the specified space.

If $ARGUMENTS is empty, first run `clickup-tool get-spaces` to find the space ID, then ask the user which space to query.

Output is a plain list of tag name strings. Suggest using them with `clickup-tool get-tasks --tag "TAG_NAME"`.

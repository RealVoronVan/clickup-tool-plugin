---
description: Browse ClickUp workspace hierarchy (spaces, folders, lists/sprints)
allowed-tools: Bash(clickup-tool *)
argument-hint: "[SPACE_ID | FOLDER_ID]"
---

Navigate the ClickUp hierarchy:
- No arguments: run `clickup-tool get-spaces` to list all spaces with their available statuses.
- If $ARGUMENTS looks like a space ID: run `clickup-tool get-folders $ARGUMENTS`.
- If $ARGUMENTS looks like a folder ID: run `clickup-tool get-lists $ARGUMENTS`.

Guide the user through the hierarchy: spaces -> folders -> lists.

After showing results, suggest the next navigation step. When showing spaces, highlight the available statuses per space — these are the exact status names to use with `--status` filter.

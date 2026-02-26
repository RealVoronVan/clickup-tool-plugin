---
description: Add a comment to a ClickUp task. Can compose comment text from
  instructions like "summarize the last commit" or "write a status update".
argument-hint: <TASK_ID> [comment text or instruction]
---

Post a comment on a ClickUp task.

## Input

- `$ARGUMENTS` contains the task ID and optionally the comment text or an instruction for composing it.
- If only a task ID is provided, ask the user what to write.
- If an instruction is given (e.g., "summarize last commit", "write status update"), use your
  available tools (git, file reads, clickup-tool queries) to gather context and compose the comment.

## Steps

1. Parse task ID from `$ARGUMENTS` (first word). Everything after is the text or instruction.
2. If no text/instruction provided, ask the user what the comment should say.
3. Compose the comment text. If the user gave an instruction rather than literal text,
   gather necessary context (git log, files, previous comments) and draft the comment.
4. Show the user a preview:
   - **Task**: task ID
   - **Comment**: full text that will be posted
5. Ask the user to confirm before sending.
6. Run: `clickup-tool add-comment TASK_ID "comment text"`
   - Add `--notify-all` only if the user explicitly asks to notify everyone.
7. Report the result.

## Important

- ALWAYS show preview and get confirmation before posting.
- NEVER post a comment without user approval.
- The comment text argument must be quoted if it contains spaces.

---
description: List open Nexium tasks, optionally for one project
argument-hint: "[project]"
allowed-tools: mcp__plugin_nexium_nexium__list_tasks
---

List Nexium tasks with the `list_tasks` tool. This command only reads.

Arguments: `$ARGUMENTS`

- If an argument names a project, pass it as `project`.
- `state` is `open` unless the developer's words ask for one of these:
  `mine` (tasks they created or follow), `needs_draft` (tasks with no open
  draft whose draft step has not finished), or `all`.
- Leave `limit` at its default.

Show one table, newest first:

| # | title | risk | status | waiting on | draft |
|---|---|---|---|---|---|

- **waiting on**: the gate's name and its holders, for example
  `G5 Code review (human): Tural H.`. Use `-` when there is none.
- **draft**: `!<mr_iid>` with its link and origin (`developer` or `coder`).
  Use `-` when there is none.

If `truncated` is true, say that more tasks matched than are shown.

End with one line: `/nexium:task <id>` shows a task, and
`/nexium:implement <id>` starts work on it. Do not start any work yourself.

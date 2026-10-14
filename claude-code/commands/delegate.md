---
description: Ask Nexium's ai-coder to write the draft for a task on the server
argument-hint: "<id> [note]"
disable-model-invocation: true
allowed-tools: mcp__plugin_nexium_nexium__get_task, Bash(git status:*), Bash(git branch:*), Bash(git rev-parse:*)
---

Hand a Nexium task's draft to Nexium's ai-coder. Arguments: `$ARGUMENTS`

The first word is the task id (`482` or `#482`). Everything after it is the
note to ai-coder, at most 4000 characters. If the id is missing, ask for it.

Before calling anything that changes state:
1. If this session has already written code for this task, or the current
   branch is this task's branch (`nexium/<id>-…`), stop and ask the
   developer. Delegating now would mean two drafts for one task.
2. Call `get_task`. If `can.delegate.ok` is false, show
   `can.delegate.message` and stop. For `gates_open`, ai-coder writes the
   draft by itself once the named gate is decided, so nothing needs to be
   done.

Then call `delegate_task` with `{"task": <id>, "note": "<note>"}`. Leave
`note` out if there is none.

On success, report the console's `next` sentence. Then say:
- ai-coder usually takes 2 to 5 minutes;
- `/nexium:task <id>` shows the draft when it is open;
- do not implement this task locally meanwhile.

On an error, show the console's message. For `scope_missing`, the token can
read but not change tasks. An admin can issue a token with "Create tasks and
attach drafts" on the console's Users page.

Do not write any code for this task in this command.

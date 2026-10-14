---
description: Implement a Nexium task on your own branch, self-review it with Nexium's reviewer, push it after you confirm, and attach it as a Draft MR
argument-hint: "<id> [story <n>] [extra instructions]"
disable-model-invocation: true
allowed-tools: mcp__plugin_nexium_nexium__get_task, mcp__plugin_nexium_nexium__get_conventions, mcp__plugin_nexium_nexium__ask_vault, mcp__plugin_nexium_nexium__search_code, mcp__plugin_nexium_nexium__search_vault, mcp__plugin_nexium_nexium__get_document, mcp__plugin_nexium_nexium__review_changes, mcp__plugin_nexium_nexium__get_status, Bash(git status:*), Bash(git fetch:*), Bash(git log:*), Bash(git diff:*), Bash(git branch:*), Bash(git rev-parse:*), Bash(git remote get-url:*)
---

Implement a Nexium task. Arguments: `$ARGUMENTS`

Read the arguments like this:
- the first word is the task id (`482` or `#482`). If it is missing, ask
  for it;
- `story <n>`, if present, limits the work to that story. The task's
  regulatory obligations still apply;
- everything else is extra instructions from the developer. Follow them as
  long as they do not conflict with the task's acceptance criteria or its
  obligations. If they do conflict, ask.

Use the `nexium:nexium-task` skill and follow its steps 1 to 9 in order.

Do not skip these, whatever else happens:
- Each obligation's acceptance criterion in `get_task`'s `compliance` is part
  of the work, like a story's. Name the code and the test that meet it, and
  write the test if there is none.
- Run the self-review (step 7) before you push: `review_changes` on
  `git diff origin/<target_branch>...HEAD` with the task title and the
  project, then `get_status`. Fix every critical, blocker and major finding
  and every compliance flag, or say why not. At most two rounds.
- Show the developer every `pci_flags` and `compliance_flags` entry, fixed or
  not, and the minor and info findings. The compliance officer decides at G10.
- Ask the developer before `git push`, and wait for a clear yes.
- Never push to or commit on the target branch. Never force-push.
- If the draft step is `queued` or `running`, stop: ai-coder is already
  writing this draft.
- When the task has obligations, pass `compliance_summary` to `attach_draft`:
  one line covering every obligation tag.
- After `attach_draft`, report the review gate and who holds it. Never say
  the change is approved or that a regulation is met.

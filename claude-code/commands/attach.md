---
description: Self-review your branch (default the current one) with Nexium's reviewer, then attach it to a Nexium task as its Draft MR
argument-hint: "<id> [branch]"
disable-model-invocation: true
allowed-tools: mcp__plugin_nexium_nexium__get_task, mcp__plugin_nexium_nexium__review_changes, mcp__plugin_nexium_nexium__get_status, Bash(git status:*), Bash(git fetch:*), Bash(git log:*), Bash(git diff:*), Bash(git rev-parse:*), Bash(git branch:*), Bash(git remote get-url:*)
---

Attach a branch to a Nexium task as its Draft MR. Arguments: `$ARGUMENTS`

The first word is the task id (`482` or `#482`). If it is missing, ask for
it. The second word, if any, is the branch. With no branch, use the current
one (`git rev-parse --abbrev-ref HEAD`). If the second word is an MR
reference such as `!12`, attach that MR with `mr_iid: 12` instead of a
branch, and skip steps 2 to 5. Say that no self-review ran and no compliance
summary was sent.

1. Call `get_task`. If `can.attach.ok` is false, show
   `can.attach.message` and stop. Note `compliance.obligations`, if any:
   the regulations the client enabled (GDPR, HIPAA, SOC 2, PCI DSS, …)
   require them of this task.
2. Check the branch:
   - it is not `HEAD` (a detached checkout) and not `project.target_branch`.
     Never attach the target branch;
   - run `git fetch origin <target_branch>`.
3. **Self-review**, as in the `nexium:nexium-task` skill's step 7:
   - call `review_changes` with the output of
     `git diff origin/<target_branch>...<branch>` (unedited) as `diff`, the
     task title as `title`, and `project.name` as `project`. Then poll
     `get_status` with the job id while `status` is `running`;
   - show every critical, blocker and major finding, every
     `compliance_flags` entry and every `pci_flags` entry, then the minor and
     info findings;
   - this branch may be the developer's own work, so **ask before you
     change any code**. With a yes, fix the critical, blocker and major
     findings and the compliance flags, run the tests, commit, and review
     once more. At most two rounds, and a second one only when something
     changed. What is not fixed is listed with the reason;
   - never hide a `pci_flags` or `compliance_flags` entry, fixed or not. The
     compliance officer decides at G10.
4. For each obligation, find the code and the test that meet its acceptance
   criterion. Where the test is missing, say so and offer to write it (ask
   first, as above). Write the compliance summary: one line covering every tag,
   `Compliance: [tag] addressed in <file>; [tag] not addressed — <why>`, at
   most 4000 characters. Show it to the developer.
5. Check the branch is pushed: `git fetch origin <branch>`, then
   `git log --oneline origin/<branch>..<branch>`. Any output means unpushed
   commits. If it is not pushed, or has unpushed commits, show what would go
   up and **ask** "Push `<branch>` to origin?". Push only after a clear yes,
   with `git push -u origin <branch>`. Never force-push. If the developer
   says no, stop.
6. Call `attach_draft` with `{"task": <id>, "branch": "<branch>"}` (or
   `{"task": <id>, "mr_iid": <n>}`). Add `"compliance_summary": "<line>"`
   when the task has obligations.

On success, report the MR link and `!<mr_iid>`, `files`, `added`,
`removed`, the compliance summary sent, the review gate (`review_gate.name`
and `holders`), and the `next` sentence. Never say the change is approved
or that a regulation is met. The people on the gates decide in the console.
Later pushes to the same branch update the same MR.

On an error, show the console's message. For `secret_found`, list the
findings (path, line, kind). The developer must remove the secret from the
branch's history and rotate it. Do not force-push on your own. For
`summary_too_long`, shorten the reasons and paths, keep every tag, and
attach again. For `scope_missing`, an admin can issue a token with "Create
tasks and attach drafts" on the console's Users page.

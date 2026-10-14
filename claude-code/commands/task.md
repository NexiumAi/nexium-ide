---
description: Show one Nexium task with its gates, stories, acceptance criteria, regulatory obligations and draft
argument-hint: "<id>"
allowed-tools: mcp__plugin_nexium_nexium__get_task
---

Show Nexium task `$ARGUMENTS` with the `get_task` tool. This command only
reads. If no task id was given, ask for one.

Show, in this order:

1. **Title, risk, status**, the project and its target branch, and the
   console link (`url`).
2. **Gates**: one line each, with the gate number, name, status, and who
   decided it or who holds it.
3. **Waiting on**: the gate's name and its holders.
4. **Stories**:
   - `ready`: each story's title, with its acceptance criteria (`ac`) as a
     list. Say that these are what the review gate checks.
   - `unparsed`: say the stories are not structured, and summarise
     `spec.text`.
   - `absent`: say that G2 (BA story review) has not been reached, so there
     are no stories yet.
5. **Regulations**, from `compliance`. Leave this section out when
   `compliance` is `null`.
   - the frameworks (name and version) and the data categories;
   - each obligation: its `tag`, `title`, `must` or `should`, and its
     `acceptance` criterion. Say that these are part of the work, like the
     stories' criteria;
   - the `gaps`: obligations no story covers;
   - G10: its status and who decided it, for example "G10 approved by
     Aysel K.". While G10 is not decided, say the obligations are proposals
     the compliance officer may still change. When `g10` is `null`, the task
     has no G10, and the obligations stand as proposed;
   - `obligations` is `[]`: show the `not_applicable` sentence, or say that
     no obligation is left.
6. **Draft**: the MR link, `source_branch`, origin (`developer` or `coder`),
   state, and the draft step's status.
7. **What can be done now**, from `can`:
   - `can.attach`: if `ok`, `/nexium:implement <id>` or `/nexium:attach <id>`;
     if not, its `message`.
   - `can.delegate`: if `ok`, `/nexium:delegate <id>`; if not, its `message`.

Do not start implementing. Never describe a gate as approved unless its
`status` says `approved`.

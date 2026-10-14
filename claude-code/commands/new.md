---
description: Create a new Nexium task in a project; it goes through intake and G1 before any code
argument-hint: "<project> <request>"
disable-model-invocation: true
allowed-tools: mcp__plugin_nexium_nexium__list_tasks
---

Create a new Nexium task. Arguments: `$ARGUMENTS`

The first word is the project name. Everything after it is the request, in
the developer's words, 1 to 8000 characters. If either is missing, ask for
it.

1. **Check it does not exist already.** Call `list_tasks` with that
   `project` and `state: "open"`. If an open task asks for the same thing,
   show it (id, title, link) and ask whether to create a new one anyway.
   Never create a duplicate without a yes.
2. Call `create_task` with `{"project": "<project>", "request": "<request>"}`.
   - Send the request as the developer wrote it. Do not rewrite it, add to
     it or invent detail.
   - Add `hints` only for what the developer said outright, for example
     "touches the database" means `touches_db: true`. The keys are `ui`,
     `touches_db`, `touches_auth`, `new_integration`, `infra_change`,
     `new_microservice` and `docs_only`. Do not guess any of them.
3. Report the task's id, its console link (`url`) and the `next` sentence.

The new task goes through intake and G1 (PO approval) first. Do not start
implementing it. `/nexium:task <id>` shows its progress.

On an error, show the console's message. `project_not_found` means the name
is not a project of this tenant. `no_gate_holder` means a gate has nobody to
decide it, and an admin must fix that in the console. `scope_missing` means
the token can read but not change tasks. An admin can issue a token with
"Create tasks and attach drafts" on the console's Users page.

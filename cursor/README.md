# Nexium for Cursor

The same workflow as the Claude Code plugin, for Cursor's agent. Nexium
supplies the task, its stories and acceptance criteria, the regulatory
obligations that apply to it, and the project's conventions. Cursor and you write the code on your own branch, and your git
pushes it. Nexium links the branch to the task as a Draft MR, and its gates
decide. No code passes through Nexium.

Two files:

| file | goes to | what it is |
|---|---|---|
| `mcp.json.example` | `~/.cursor/mcp.json` (all projects) or `<repo>/.cursor/mcp.json` (one project) | connects Cursor to the Nexium MCP server |
| `rules/nexium.mdc` | `<repo>/.cursor/rules/nexium.mdc` | tells the agent how to work on a Nexium task |

Neither file contains a secret, so both can be committed for the team.

## 1. Get a token

An admin of your Nexium console issues it on the **Users** page. You cannot
issue a token to yourself. To create tasks, attach drafts or delegate to
ai-coder, the admin must tick **"Create tasks and attach drafts"**. Without
it, the token can only read. The token is shown once. Treat it like a
password.

## 2. Set two environment variables

In your shell profile (`~/.zshrc`, `~/.bashrc`):

```sh
export NEXIUM_MCP_URL="https://mcp-<your-company>.cloud.nexium.llc/mcp"   # your admin gives you this
export NEXIUM_MCP_TOKEN="<your token>"
```

`mcp.json.example` reads them with Cursor's `${env:NAME}` syntax. Cursor
expands it in `url` and `headers` (Cursor docs, "MCP", "Config
interpolation"). If Cursor does not see the variables, quit it and start it
from that shell with `cursor .`.

## 3. Install

```sh
# MCP server: copy, or merge the "nexium" entry into an existing file
cp mcp.json.example ~/.cursor/mcp.json

# the rule, in each repository you work on
mkdir -p <repo>/.cursor/rules
cp rules/nexium.mdc <repo>/.cursor/rules/
```

**Check it works:** open Cursor Settings and find the MCP section (its name
varies between Cursor versions). `nexium` must be listed as connected, with
tools such as `list_tasks`, `get_task` and `attach_draft`.

## Use it

Cursor has no plugin commands. Ask the agent in plain words. The rule loads
when the request is about a Nexium task. Mention `@nexium` to force it.

- "List my Nexium tasks" / "List open Nexium tasks in payments-api"
- "Show Nexium task 482"
- "Implement Nexium task 482", or "… only story 2"
- "Attach my current branch to Nexium task 482"
- "Delegate Nexium task 482 to ai-coder: use the existing refund service"
- "Create a Nexium task in payments-api: merchants need partial refunds"

For an implementation, the agent reads the task and the conventions, creates
`nexium/<id>-<slug>` from the target branch, implements, runs the tests,
commits, has Nexium's reviewer check the diff (`review_changes`, a minute or
two) and fixes what it finds, **asks you before `git push`**, and then
attaches the branch.

## Regulations

A client can enable the regulations it must follow: GDPR, HIPAA, SOC 2, PCI
DSS, a regulation it uploaded, and others. A task then carries
**obligations**, each with a tag such as `[gdpr:Art. 32(1)(b)]`, a title and
an acceptance criterion.

- The agent treats each obligation's acceptance criterion like a story's,
  and names the code and the test that meet it.
- Its self-review fixes the critical, blocker and major findings and every
  compliance flag, in at most two rounds. It shows you what is left, the
  minor findings, and **every PCI and compliance flag, fixed or not**.
- The Draft MR gets one line on each obligation:
  `Compliance: [tag] addressed in <file>; [tag] not addressed — <why>`.

The compliance officer decides, at G10 in the console, which obligations
apply and what a flag means for the task. The agent never says a regulation
is met.

## Who decides what

- **Nexium, and the people on its gates in the console:** whether the task
  goes ahead (G1), the stories and acceptance criteria (G2), the regulatory
  obligations (G10), the conventions, the risk and gates, who reviews, and
  whether the change is approved.
- **You:** how to implement it and how the code meets each obligation, the
  branch and commits, whether and when to push, when to attach, and whether
  to write it yourself or hand it to ai-coder.

The rule tells the agent never to push to the target branch, never to
force-push, never to push without asking, and never to say a change is
approved. A rule is guidance to the model, not a lock. Keep Cursor asking
before it runs terminal commands and MCP tools, so that `git push` and the
three tools that change state (`create_task`, `delegate_task`,
`attach_draft`) always reach you first.

Before it attaches a branch, Nexium scans the branch's changes for secrets.
A critical hit blocks the attach, but by then the branch is already in
GitLab. Keep secrets out of commits before you push.

## Troubleshooting

| you see | it means |
|---|---|
| `nexium` not connected, or HTTP 401 | the URL or token is wrong, revoked or not set in Cursor's environment. Start Cursor from the shell that has the variables |
| "Your Nexium token can read but not change tasks" | ask an admin for a token with "Create tasks and attach drafts" |
| "The Nexium console is unreachable" | the MCP server is up but the console behind it is not. Try later, or tell your admin |

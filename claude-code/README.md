# Nexium for Claude Code

Take a task from Nexium, implement it with Claude Code on your own branch, and
attach the branch to the task as a Draft merge request. Nexium's gates then
review it.

- **Nexium supplies what to build:** the task, its stories and acceptance
  criteria, the regulatory obligations that apply to it, the project's
  conventions, and the knowledge vault.
- **Claude Code and you write the code.** Your own git pushes it. No code
  passes through Nexium.
- **Nexium's gates decide.** The plugin cannot approve anything.

This folder is the Claude Code plugin. The marketplace that lists it is
`.claude-plugin/marketplace.json` at the repository's root. The same
workflow exists for OpenAI Codex, Cursor and IntelliJ: see the
[repository's README](../README.md).

Or, if you prefer, ask Nexium's ai-coder to write the draft on the server
(`/nexium:delegate`).

## What you need

- Claude Code 2.1.284 or later. This plugin was checked against 2.1.284.
- A local clone of the project's repository, with permission to push
  branches.
- The address of your Nexium MCP server and a Nexium token (both below).

## 1. Get a token

An admin of your Nexium console issues it on the **Users** page. You cannot
issue a token to yourself.

- **For reading only** (list tasks, read a task, read conventions, search
  the vault), the default token is enough.
- **For creating tasks, attaching drafts and delegating to ai-coder**, the
  admin must tick **"Create tasks and attach drafts"** when issuing the
  token. Without it, those three commands answer: "Your Nexium token can read
  but not change tasks."

The token is shown once. Treat it like a password.

## 2. Set two environment variables

Put these in your shell profile (`~/.zshrc`, `~/.bashrc`), not in any
repository:

```sh
export NEXIUM_MCP_URL="https://mcp-<your-company>.cloud.nexium.llc/mcp"   # your admin gives you this
export NEXIUM_MCP_TOKEN="<your token>"
```

Start `claude` from a shell that has them. The plugin reads them when Claude
Code starts. Always use `https://`, because the token travels in a header.

## 3. Install

**From the marketplace** (inside Claude Code):

```
/plugin marketplace add NexiumAi/nexium-ide
/plugin install nexium@nexium
```

The plugin is published at <https://github.com/NexiumAi/nexium-ide>. A local
clone works as a marketplace too, for example inside a network that cannot
reach GitHub:

```
/plugin marketplace add /path/to/nexium-ide
/plugin install nexium@nexium
```

The same works from a terminal: `claude plugin marketplace add …`, then
`claude plugin install nexium@nexium`.

**For one session only**, without installing:

```sh
claude --plugin-dir /path/to/nexium-ide/claude-code
```

**Check it works:**

```sh
claude mcp list
```

The list must contain a line like this:

```
plugin:nexium:nexium: https://mcp-….cloud.nexium.llc/mcp (HTTP) - ✔ Connected
```

Inside Claude Code, `/mcp` shows the same.

If you already added the Nexium bridge (`nexium-mcp-bridge`) for the same
server by hand, remove one of the two, so that Claude does not see every
tool twice.

## Commands

| command | what it does | changes anything? |
|---|---|---|
| `/nexium:tasks [project]` | Lists open tasks: risk, status, which gate waits on whom, and the draft MR. Say "mine", "needs draft" or "all" for other lists. | no |
| `/nexium:task <id>` | Shows one task: its gates, its stories with their acceptance criteria, its regulatory obligations, the draft, and what can be done now. | no |
| `/nexium:implement <id> [story <n>] [extra instructions]` | Reads the task and the project's conventions, searches the code and the vault, and creates `nexium/<id>-<slug>` from the target branch. Then it implements, runs the tests, commits, has Nexium's reviewer check the diff and fixes what it finds, **asks you before pushing**, and attaches the branch as a Draft MR. | your branch; a Draft MR |
| `/nexium:delegate <id> [note]` | Asks Nexium's ai-coder to write the draft on the server. It takes 2 to 5 minutes. Your note goes to ai-coder. | queues ai-coder |
| `/nexium:new <project> <request>` | Creates a new task. It goes through intake and G1 (PO approval) before anyone writes code. It checks for an existing open task first. | a new task |
| `/nexium:attach <id> [branch]` | Has Nexium's reviewer check the branch (by default the current one), then attaches it to the task as a Draft MR. It asks before changing your code and before pushing. `!12` instead of a branch links an existing MR, without a review. | a Draft MR |

You do not need a command to start. Saying "implement Nexium task 482" also
works, through the plugin's `nexium-task` skill. Claude cannot run `new`,
`delegate`, `attach` or `implement` on its own. Only you can type them.

## Regulations

A client can enable the regulations it must follow: GDPR, HIPAA, SOC 2, PCI
DSS, a regulation it uploaded, and others. A task then carries
**obligations**. Each has a tag such as `[gdpr:Art. 32(1)(b)]`, a title, and
an acceptance criterion.

**What you see:**
- `/nexium:task` lists the obligations under **Regulations**, with the gaps
  (obligations no story covers) and G10's state.
- `/nexium:implement` treats each obligation's acceptance criterion like a
  story's. It names the code and the test that meet it, and writes the test
  when there is none.
- **Before pushing, it runs a self-review.** Nexium's reviewer reads the diff
  against the project and its regulations, which takes a minute or two.
  Claude fixes the critical,
  blocker and major findings and every compliance flag, in at most two
  rounds. Then it shows you what it fixed, what is left and why, the minor
  findings, and **every PCI and compliance flag, fixed or not**.
- The Draft MR gets one line on each obligation, for example
  `Compliance: [gdpr:Art. 32(1)(b)] addressed in src/export/Encrypt.java; [gdpr:Art. 30] not addressed — <why>`.
  When Nexium opens the MR, the line goes under "Compliance" in its
  description. Either way, the console counts it as evidence for each
  obligation it names.

**What the compliance officer decides, at G10 in the console:** which
obligations apply to the task (accepted, edited or removed), and what a PCI
or compliance flag means for it. Until G10 is approved, the obligations are
proposals and can change. Claude never says a regulation is met, and never
hides a flag.

## Who decides what

| Nexium, and the people on its gates in the console | You |
|---|---|
| Whether the task goes ahead (G1) | How to implement it |
| The stories and their acceptance criteria (G2) | The branch, the commits and their messages |
| The regulatory obligations, and what a flag means for the task (G10) | How the code meets each obligation |
| The project's conventions | Whether and when to push. Claude asks every time |
| The risk, and which gates the task passes | When to attach the branch |
| Who reviews the code, and whether it is approved | Whether to write it yourself or hand it to ai-coder |

`attach_draft` only opens or links a Draft MR. Nexium uses the project's own
GitLab credential for that, so your token never touches GitLab. The review
gate then waits on the people who hold it. Later pushes to the same branch
update the same MR, so you do not need to attach again.

## What the plugin never does

- Commit to, push to or attach the target branch.
- Force-push, or rewrite history that is already pushed.
- Push without asking you first.
- Say that a change is approved. It tells you which gate waits, and on whom.
- Say that a regulation is met, or hide a PCI or compliance flag from you.
- Merge an MR or take it out of Draft.
- Write a draft itself while ai-coder is writing one for the same task.
- Create a task that already exists.

Before it attaches a branch, Nexium scans the branch's changes for secrets. A
critical hit blocks the attach, but by then the branch is already in GitLab.
Keep secrets out of commits before you push.

## Permissions

The read-only tools are pre-approved inside the commands that use them:
`list_tasks`, `get_task`, `get_conventions`, `ask_vault`, `search_code`,
`search_vault`, `get_document`, `review_changes` and `get_status` (the
self-review), and read-only git (`status`, `log`, `diff`, `fetch`, …). `create_task`, `delegate_task` and `attach_draft` always ask
for permission, and so do file edits and `git push`, unless you have allowed
them yourself.

## Troubleshooting

| you see | it means |
|---|---|
| `Missing environment variables: NEXIUM_MCP_URL` | `NEXIUM_MCP_URL` is not set in the shell that started `claude`. |
| `Server rejected the configured Authorization header (HTTP 401)` | The token is wrong, expired or revoked, **or `NEXIUM_MCP_TOKEN` is not set.** Claude Code does not report a missing token variable: it sends the placeholder text instead. Check `echo $NEXIUM_MCP_TOKEN`. |
| "Your Nexium token can read but not change tasks" | Ask an admin for a token with "Create tasks and attach drafts". |
| "The Nexium console is unreachable" | The MCP server is up but the console behind it is not. Try again later, or tell your admin. |
| `draft_open` when attaching | The task already has an open draft. The message says which branch to continue on, or close that MR in GitLab first. |
| `summary_too_long` when attaching | The compliance summary is over 4000 characters. Claude shortens it, keeps every tag, and attaches again. |

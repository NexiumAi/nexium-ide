# Codeweir for OpenAI Codex

Take a task from Codeweir, implement it with Codex on your own branch, and
attach the branch to the task as a Draft merge request. Codeweir's gates then
review it.

- **Codeweir supplies what to build:** the task, its stories and acceptance
  criteria, the regulatory obligations that apply to it, the project's
  conventions, and the knowledge vault.
- **Codex and you write the code.** Your own git pushes it. Codeweir never
  pushes your code or writes your files. The diff you send for the
  self-review (`review_changes`) has its secrets masked, is read by Codeweir's
  reviewer model through your Codeweir server's model gateway, and is stored
  on that server with the review.
- **Codeweir's gates decide.** The plugin cannot approve anything.

The plugin brings one skill, `codeweir-task`: the workflow. The connection to
your Codeweir server is three lines in your Codex config (step 2), because
every company has its own server address.

## What you need

- The Codex CLI. This plugin was checked against `codex-cli 0.162.1`.
- A local clone of the project's repository, with permission to push
  branches.
- The address of your Codeweir MCP server and a Codeweir token (both below).

## 1. Get a token

An admin of your Codeweir console issues it on the **Users** page. You cannot
issue a token to yourself.

- **For reading only** (list tasks, read a task, read conventions, search
  the vault), the default token is enough.
- **For creating tasks, attaching drafts and delegating to ai-coder**, the
  admin must tick **"Create tasks and attach drafts"** when issuing the
  token. Without it, those three answer: "Your Codeweir token can read but not
  change tasks."

The token is shown once. Treat it like a password.

## 2. Connect Codex to your Codeweir server

Add these three lines to `~/.codex/config.toml`:

```toml
[mcp_servers.codeweir]
url = "https://mcp.example.com/mcp"   # your admin gives you this
bearer_token_env_var = "CODEWEIR_MCP_TOKEN"
```

The same from a terminal:

```sh
codex mcp add codeweir --url https://mcp.example.com/mcp --bearer-token-env-var CODEWEIR_MCP_TOKEN
```

Put the token in your shell profile (`~/.zshrc`, `~/.bashrc`), not in any
repository and not in `config.toml`:

```sh
export CODEWEIR_MCP_TOKEN="<your token>"
```

Start `codex` from a shell that has it. Always use `https://`, because the
token travels in a header.

**Recommended:** add a fourth line under `[mcp_servers.codeweir]`, so that
Codex asks you before every tool that changes something in Codeweir
(`create_task`, `delegate_task`, `attach_draft`). The read tools are marked
read-only by the server and run without asking.

```toml
default_tools_approval_mode = "writes"
```

## 3. Install the plugin

```sh
codex plugin marketplace add Codeweir/codeweir-ide
codex plugin add codeweir@codeweir
```

A local clone works as a marketplace too, for example inside a network that
cannot reach GitHub:

```sh
codex plugin marketplace add /path/to/codeweir-ide
codex plugin add codeweir@codeweir
```

**Check it works:**

```sh
codex plugin list      # codeweir@codeweir  installed, enabled  1.0.0
codex mcp list         # codeweir  https://mcp.example.com/mcp  CODEWEIR_MCP_TOKEN  enabled  Bearer token
```

Inside Codex, `/mcp` lists the server's tools.

**Later:** `codex plugin marketplace upgrade codeweir` fetches a new version;
`codex plugin remove codeweir@codeweir` uninstalls it.

## Use it

Type `$codeweir-task` and pick it (Codex lists it as `codeweir:codeweir-task`), or
just ask in plain words. Codex picks the skill by itself:

| you say | what happens | changes anything? |
|---|---|---|
| "List my open Codeweir tasks" | `list_tasks`: risk, status, which gate waits on whom, and the draft MR. | no |
| "Show Codeweir task 482" | `get_task`: its gates, its stories with their acceptance criteria, its regulatory obligations, the draft, and what can be done now. | no |
| "Implement Codeweir task 482" | Reads the task and the project's conventions, searches the code and the vault, and creates `codeweir/<id>-<slug>` from the target branch. Then it implements, runs the tests, commits, has Codeweir's reviewer check the diff and fixes what it finds, **asks you before pushing**, and attaches the branch as a Draft MR. | your branch; a Draft MR |
| "Attach my branch to 482" | Has Codeweir's reviewer check the branch (by default the current one), then attaches it as a Draft MR. It asks before changing your code and before pushing. "Attach !12 to 482" links an existing MR, without a review. | a Draft MR |
| "Let Codeweir write task 482" | Asks Codeweir's ai-coder to write the draft on the server. It takes 2 to 5 minutes. | queues ai-coder |
| "Create a Codeweir task in payments: …" | Creates a new task. It goes through intake and G1 (PO approval) before anyone writes code. It checks for an existing open task first. | a new task |

The skill calls `create_task`, `delegate_task` and `attach_draft` only when
you asked for that.

## Regulations

A client can enable the regulations it must follow: GDPR, HIPAA, SOC 2, PCI
DSS, a regulation it uploaded, and others. A task then carries
**obligations**. Each has a tag such as `[gdpr:Art. 32(1)(b)]`, a title, and
an acceptance criterion.

- Codex treats each obligation's acceptance criterion like a story's. It
  names the code and the test that meet it, and writes the test when there
  is none.
- **Before pushing, it runs a self-review.** Codeweir's reviewer reads the diff
  against the project and its regulations, which takes a minute or two.
  Codex fixes the critical, blocker and major findings and every compliance
  flag, in at most two rounds. Then it shows you what it fixed, what is left
  and why, the minor findings, and **every PCI and compliance flag, fixed or
  not**.
- The Draft MR gets one line on each obligation, for example
  `Compliance: [gdpr:Art. 32(1)(b)] addressed in src/export/Encrypt.java; [gdpr:Art. 30] not addressed — <why>`.
  The console counts it as evidence for each obligation it names.

The compliance officer decides at G10 in the console which obligations apply
and what a flag means. Codex never says a regulation is met, and never hides
a flag.

## What the plugin never does

- Commit to, push to or attach the target branch.
- Force-push, or rewrite history that is already pushed.
- Push without asking you first.
- Say that a change is approved. It tells you which gate waits, and on whom.
- Say that a regulation is met, or hide a PCI or compliance flag from you.
- Merge an MR or take it out of Draft.
- Write a draft itself while ai-coder is writing one for the same task.
- Create a task that already exists.

Before it attaches a branch, Codeweir scans the branch's changes for secrets. A
critical hit blocks the attach, but by then the branch is already in GitLab.
Keep secrets out of commits before you push.

## Troubleshooting

| you see | it means |
|---|---|
| Codex has no `codeweir` tools | `[mcp_servers.codeweir]` is missing from `~/.codex/config.toml`, or Codex was started before you added it. Check `codex mcp list`, then restart Codex. |
| HTTP 401 from the server | The token is wrong, expired or revoked, **or `CODEWEIR_MCP_TOKEN` is not set** in the shell that started `codex`. Check `echo $CODEWEIR_MCP_TOKEN`. |
| "Your Codeweir token can read but not change tasks" | Ask an admin for a token with "Create tasks and attach drafts". |
| "The Codeweir console is unreachable" | The MCP server is up but the console behind it is not. Try again later, or tell your admin. |
| `draft_open` when attaching | The task already has an open draft. The message says which branch to continue on, or close that MR in GitLab first. |
| `summary_too_long` when attaching | The compliance summary is over 4000 characters. Codex shortens it, keeps every tag, and attaches again. |

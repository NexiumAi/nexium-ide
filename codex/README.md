# Nexium for OpenAI Codex

Take a task from Nexium, implement it with Codex on your own branch, and
attach the branch to the task as a Draft merge request. Nexium's gates then
review it.

- **Nexium supplies what to build:** the task, its stories and acceptance
  criteria, the regulatory obligations that apply to it, the project's
  conventions, and the knowledge vault.
- **Codex and you write the code.** Your own git pushes it. No code passes
  through Nexium.
- **Nexium's gates decide.** The plugin cannot approve anything.

The plugin brings one skill, `nexium-task`: the workflow. The connection to
your Nexium server is three lines in your Codex config (step 2), because
every company has its own server address.

## What you need

- The Codex CLI. This plugin was checked against `codex-cli 0.162.1`.
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
  token. Without it, those three answer: "Your Nexium token can read but not
  change tasks."

The token is shown once. Treat it like a password.

## 2. Connect Codex to your Nexium server

Add these three lines to `~/.codex/config.toml`:

```toml
[mcp_servers.nexium]
url = "https://mcp-<your-company>.cloud.nexium.llc/mcp"   # your admin gives you this
bearer_token_env_var = "NEXIUM_MCP_TOKEN"
```

The same from a terminal:

```sh
codex mcp add nexium --url https://mcp-<your-company>.cloud.nexium.llc/mcp --bearer-token-env-var NEXIUM_MCP_TOKEN
```

Put the token in your shell profile (`~/.zshrc`, `~/.bashrc`), not in any
repository and not in `config.toml`:

```sh
export NEXIUM_MCP_TOKEN="<your token>"
```

Start `codex` from a shell that has it. Always use `https://`, because the
token travels in a header.

**Recommended:** add a fourth line under `[mcp_servers.nexium]`, so that
Codex asks you before every tool that changes something in Nexium
(`create_task`, `delegate_task`, `attach_draft`). The read tools are marked
read-only by the server and run without asking.

```toml
default_tools_approval_mode = "writes"
```

## 3. Install the plugin

```sh
codex plugin marketplace add NexiumAi/nexium-ide
codex plugin add nexium@nexium
```

A local clone works as a marketplace too, for example inside a network that
cannot reach GitHub:

```sh
codex plugin marketplace add /path/to/nexium-ide
codex plugin add nexium@nexium
```

**Check it works:**

```sh
codex plugin list      # nexium@nexium  installed, enabled  1.0.0
codex mcp list         # nexium  https://mcp-….cloud.nexium.llc/mcp  NEXIUM_MCP_TOKEN  enabled  Bearer token
```

Inside Codex, `/mcp` lists the server's tools.

**Later:** `codex plugin marketplace upgrade nexium` fetches a new version;
`codex plugin remove nexium@nexium` uninstalls it.

## Use it

Type `$nexium-task` and pick it (Codex lists it as `nexium:nexium-task`), or
just ask in plain words. Codex picks the skill by itself:

| you say | what happens | changes anything? |
|---|---|---|
| "List my open Nexium tasks" | `list_tasks`: risk, status, which gate waits on whom, and the draft MR. | no |
| "Show Nexium task 482" | `get_task`: its gates, its stories with their acceptance criteria, its regulatory obligations, the draft, and what can be done now. | no |
| "Implement Nexium task 482" | Reads the task and the project's conventions, searches the code and the vault, and creates `nexium/<id>-<slug>` from the target branch. Then it implements, runs the tests, commits, has Nexium's reviewer check the diff and fixes what it finds, **asks you before pushing**, and attaches the branch as a Draft MR. | your branch; a Draft MR |
| "Attach my branch to 482" | Has Nexium's reviewer check the branch (by default the current one), then attaches it as a Draft MR. It asks before changing your code and before pushing. "Attach !12 to 482" links an existing MR, without a review. | a Draft MR |
| "Let Nexium write task 482" | Asks Nexium's ai-coder to write the draft on the server. It takes 2 to 5 minutes. | queues ai-coder |
| "Create a Nexium task in payments: …" | Creates a new task. It goes through intake and G1 (PO approval) before anyone writes code. It checks for an existing open task first. | a new task |

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
- **Before pushing, it runs a self-review.** Nexium's reviewer reads the diff
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

Before it attaches a branch, Nexium scans the branch's changes for secrets. A
critical hit blocks the attach, but by then the branch is already in GitLab.
Keep secrets out of commits before you push.

## Troubleshooting

| you see | it means |
|---|---|
| Codex has no `nexium` tools | `[mcp_servers.nexium]` is missing from `~/.codex/config.toml`, or Codex was started before you added it. Check `codex mcp list`, then restart Codex. |
| HTTP 401 from the server | The token is wrong, expired or revoked, **or `NEXIUM_MCP_TOKEN` is not set** in the shell that started `codex`. Check `echo $NEXIUM_MCP_TOKEN`. |
| "Your Nexium token can read but not change tasks" | Ask an admin for a token with "Create tasks and attach drafts". |
| "The Nexium console is unreachable" | The MCP server is up but the console behind it is not. Try again later, or tell your admin. |
| `draft_open` when attaching | The task already has an open draft. The message says which branch to continue on, or close that MR in GitLab first. |
| `summary_too_long` when attaching | The compliance summary is over 4000 characters. Codex shortens it, keeps every tag, and attaches again. |

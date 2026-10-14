# Nexium for IDE

Nexium brings a task into your editor and keeps its rules with it. You pick a
task; your coding agent reads its stories, acceptance criteria, the
regulatory obligations your company enabled (GDPR, HIPAA, SOC 2, PCI DSS, …)
and the project's conventions, then writes the code on your own branch.
Before anything is pushed, Nexium's reviewer checks the diff against those
regulations and the agent fixes what it finds. Your own git pushes the branch
after you say yes, and Nexium attaches it to the task as a Draft MR. People
decide at the gates in the Nexium console; the agent never approves anything.
If you prefer, "Let Nexium write it" hands the draft to Nexium's ai-coder on
the server instead.

**Before you start**, for every editor:

- **A token.** An admin of your Nexium console issues it on the **Users**
  page. The default token reads. To create tasks, attach drafts and delegate
  to ai-coder, the admin ticks **"Create tasks and attach drafts"**
  (`tasks.write`).
- **Your server's address**, from the same admin:
  `https://mcp-<your-company>.cloud.nexium.llc/mcp`.

## Claude Code

```sh
export NEXIUM_MCP_URL="https://mcp-<your-company>.cloud.nexium.llc/mcp"
export NEXIUM_MCP_TOKEN="<your token>"
```

Then, inside Claude Code:

```
/plugin marketplace add NexiumAi/nexium-ide
/plugin install nexium@nexium
```

Commands, regulations and troubleshooting: [claude-code/README.md](claude-code/README.md).

## OpenAI Codex

In `~/.codex/config.toml`, and `export NEXIUM_MCP_TOKEN="<your token>"` in your shell:

```toml
[mcp_servers.nexium]
url = "https://mcp-<your-company>.cloud.nexium.llc/mcp"
bearer_token_env_var = "NEXIUM_MCP_TOKEN"
```

```sh
codex plugin marketplace add NexiumAi/nexium-ide
codex plugin add nexium@nexium
```

Usage (`$nexium-task`), regulations and troubleshooting: [codex/README.md](codex/README.md).

## Cursor

```sh
cp cursor/mcp.json.example ~/.cursor/mcp.json          # reads NEXIUM_MCP_URL and NEXIUM_MCP_TOKEN
mkdir -p <repo>/.cursor/rules && cp cursor/rules/nexium.mdc <repo>/.cursor/rules/
```

Then ask Cursor's agent "implement Nexium task 482". Details: [cursor/README.md](cursor/README.md).

## IntelliJ IDEA and other JetBrains IDEs

A task panel: the task list, a task's stories, regulations and gates, and
buttons to implement it with Claude Code, copy a prompt for Junie or AI
Assistant, let Nexium write it, or attach your branch.

- **Install:** from JetBrains Marketplace (coming soon). Until then, your
  Nexium admin can give you the plugin zip: **Settings › Plugins › ⚙ ›
  Install Plugin from Disk…**
- **Connect:** **Settings › Tools › Nexium**, with your server's address and
  the token from your admin (Users page; tick "Create tasks and attach
  drafts" for `tasks.write`).

Details: [intellij/README.md](intellij/README.md).

# Codeweir for IDE

Codeweir brings a task into your editor and keeps its rules with it. You pick a
task; your coding agent reads its stories, acceptance criteria, the
regulatory obligations your company enabled (GDPR, HIPAA, SOC 2, PCI DSS, …)
and the project's conventions, then writes the code on your own branch.
Before anything is pushed, Codeweir's reviewer checks the diff against those
regulations and the agent fixes what it finds. Your own git pushes the branch
after you say yes, and Codeweir attaches it to the task as a Draft MR. People
decide at the gates in the Codeweir console; the agent never approves anything.
If you prefer, "Let Codeweir write it" hands the draft to Codeweir's ai-coder on
the server instead.

**Before you start**, for every editor:

- **A token.** An admin of your Codeweir console issues it on the **Users**
  page. The default token reads. To create tasks, attach drafts and delegate
  to ai-coder, the admin ticks **"Create tasks and attach drafts"**
  (`tasks.write`).
- **Your server's address**, from the same admin, for example
  `https://mcp.example.com/mcp`.

## Claude Code

```sh
export CODEWEIR_MCP_URL="https://mcp.example.com/mcp"
export CODEWEIR_MCP_TOKEN="<your token>"
```

Then, inside Claude Code:

```
/plugin marketplace add Codeweir/codeweir-ide
/plugin install codeweir@codeweir
```

Commands, regulations and troubleshooting: [claude-code/README.md](claude-code/README.md).

## OpenAI Codex

In `~/.codex/config.toml`, and `export CODEWEIR_MCP_TOKEN="<your token>"` in your shell:

```toml
[mcp_servers.codeweir]
url = "https://mcp.example.com/mcp"
bearer_token_env_var = "CODEWEIR_MCP_TOKEN"
```

```sh
codex plugin marketplace add Codeweir/codeweir-ide
codex plugin add codeweir@codeweir
```

Usage (`$codeweir-task`), regulations and troubleshooting: [codex/README.md](codex/README.md).

## Cursor

```sh
cp cursor/mcp.json.example ~/.cursor/mcp.json          # reads CODEWEIR_MCP_URL and CODEWEIR_MCP_TOKEN
mkdir -p <repo>/.cursor/rules && cp cursor/rules/codeweir.mdc <repo>/.cursor/rules/
```

Then ask Cursor's agent "implement Codeweir task 482". Details: [cursor/README.md](cursor/README.md).

## IntelliJ IDEA and other JetBrains IDEs

A task panel: the task list, a task's stories, regulations and gates, and
buttons to implement it with Claude Code, copy a prompt for Junie or AI
Assistant, let Codeweir write it, or attach your branch.

- **Install:** from JetBrains Marketplace (coming soon). Until then, your
  Codeweir admin can give you the plugin zip: **Settings › Plugins › ⚙ ›
  Install Plugin from Disk…**
- **Connect:** **Settings › Tools › Codeweir**, with your server's address and
  the token from your admin (Users page; tick "Create tasks and attach
  drafts" for `tasks.write`).

Details: [intellij/README.md](intellij/README.md).

# Codeweir for IntelliJ

Codeweir's task panel for IntelliJ IDEA and the other JetBrains IDEs built on the
same platform. The plugin is not a chat agent. It shows what to build and
passes the writing to the assistant you choose:

- **Claude Code**, in the IDE's terminal;
- **Junie, AI Assistant or Copilot**, through the Codeweir MCP server;
- **Codeweir's ai-coder** (`delegate_task`).

The plugin talks only to your organisation's Codeweir MCP server, using your own
token. It sends no code. Your own git pushes the branch, and Codeweir links the
pushed branch to the task as a Draft MR.

- Plugin id: `com.codeweir.ide`.
- Requires IntelliJ IDEA 2024.3 or newer (build 243+).

## Install

- **From JetBrains Marketplace:** coming soon. Once it is listed, search for
  "Codeweir" in **Settings › Plugins › Marketplace**.
- **From disk, until then:** your Codeweir admin gives you the plugin zip
  (`codeweir-intellij-<version>.zip`). In the IDE, open **Settings › Plugins ›
  ⚙ › Install Plugin from Disk…** and choose the zip. Restart the IDE if it
  asks you to.

The plugin depends on the bundled **Git** and **Terminal** plugins, which are
enabled by default.

## Settings

Open **Settings › Tools › Codeweir**.

| field | what goes in it |
|---|---|
| MCP URL | The Codeweir MCP endpoint your admin gives you, for example `https://mcp.example.com/mcp`. |
| Token | Your `nxm_…` token. An admin issues it on the console's Users page. The token is stored in the IDE's password safe (by default the OS keychain), never in a settings file. |

**Test connection** calls `initialize` and then `tools/list`, and shows the
server's name and how many tools it offers.

The read tools need the scope `vault.read`. Creating tasks, delegating and
attaching drafts also need `tasks.write`: an admin issues that with the
"Create tasks and attach drafts" checkbox. If your token lacks the scope, the
plugin shows the console's message and an **Open settings** button.

The plugin uses the IDE's proxy settings and trusted certificates (Settings ›
Appearance & Behavior › System Settings › HTTP Proxy, and Server
Certificates). A bank's own CA can be added there.

## The Codeweir tool window

The tool window is on the right side of the IDE.

- **Toolbar:**
  - **Refresh**;
  - **New task**;
  - the project filter;
  - the state filter: Open, Mine, Needs a draft, or All.
- **The list** (`list_tasks`): one row per task, with:
  - the id, title and risk;
  - who the task waits on;
  - the draft MR, if there is one.
- **The detail** (`get_task`):
  - the review-gate card, for example "G5 Code review (human) — Jake H.
    decides in the console";
  - the draft link and the draft step;
  - the request;
  - the stories with their acceptance criteria. If G2 has not run, the pane
    says so. If the stories did not parse, it shows the analyst's raw text;
  - the gates in order, each with its status and who decides it;
  - a link to the console.
- **Refresh:** the selected task refreshes every 30 seconds while the tool
  window is visible. Nothing polls while it is hidden.

## Actions on a task

The actions act on the task shown in the detail pane. An action the console
would refuse is greyed out, and its tooltip gives the console's reason.

| action | what it does |
|---|---|
| **Implement with Claude Code** | Opens a terminal tab in the project root and runs `claude "/codeweir:implement <id>"`. This needs the Codeweir plugin for Claude Code ([claude-code/README.md](../claude-code/README.md)). If `claude` is not on the IDE's PATH, a notification links the install page and offers **Copy prompt** instead. |
| **Copy prompt for Junie / AI Assistant** | Copies a prompt that names the Codeweir MCP tools in order: `get_task <id>`, `get_conventions`, then `attach_draft` after the push. The assistant must have the Codeweir MCP server in its own MCP settings. |
| **Let Codeweir write it** | Calls `delegate_task`, with an optional note for ai-coder. ai-coder writes the draft on the server, which usually takes 2–5 minutes. Do not also write it yourself. |
| **Attach current branch** | Takes the current branch of the project's Git repository. If the branch has no upstream, or has commits that are not pushed, the action offers **Push first**. That opens the IDE's push dialog and waits for the push to finish. The action then calls `attach_draft`, which opens or links a Draft MR on the task. Nothing is approved by this: the gates wait on people in the console. |
| **Open in console** | Opens the task's console page. |
| **New task** (toolbar) | A dialog with a project and a request, which calls `create_task`. The new task goes through intake and G1 before any code is written. |

## The status bar

When the current branch names a task, the status bar shows it, for example
"Codeweir #482 · G5 waits on Jake H.". A click opens the task in the tool
window.

The rule for reading a task id from a branch name is the console's, plus this
plugin's own branch names:
- `#482`, `nx-482`, `feature-482` or `task-482`, not glued to a letter or
  digit before it;
- `codeweir/482-…`, the branch the plugin proposes. Branches pushed before
  the plugin took the name Codeweir, with the older prefix, still name their
  task.

## Errors

Every failure is shown as the server wrote it. The console's message is shown
verbatim, for example "ai-coder writes the draft automatically when BA story
review is decided.". All network calls run off the UI thread.

## Development

This part is for Codeweir's own developers: the plugin's source is not in the
public repository.

```sh
./gradlew buildPlugin    # build/distributions/codeweir-intellij-<version>.zip
./gradlew test           # unit tests: the JSON-RPC client, decoding, branch rule, prompts
./gradlew verifyPlugin   # the Plugin Verifier, against the platform built on (offline once cached)
./gradlew runIde         # a sandbox IDE with the plugin installed
```

The build needs JDK 21 (a JetBrains Runtime is fine). On the first run it
downloads IntelliJ IDEA Community 2024.3 (about 900 MB, 2.6 GB unpacked) into
the Gradle cache.

The layout of the Kotlin sources under `src/main/kotlin/`:

| package | what is in it |
|---|---|
| `core/` | No IDE code. The JSON-RPC client (`java.net.http` and the Gson the IDE bundles; no MCP SDK), the typed tools, the shapes of the console's IDE API, the branch rule and the prompt builder. |
| `settings/` | The settings page and the password-safe token. |
| `ui/` | The tool window. |
| `actions/` | The actions. |
| `status/` | The status-bar widget. |

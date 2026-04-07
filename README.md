<div align="center">

```
  ____  _____ _   _ ____    _    ___      ____ _     ___
 / ___|| ____| \ | / ___|  / \  |_ _|    / ___| |   |_ _|
 \___ \|  _| |  \| \___ \ / _ \  | |____| |   | |    | |
  ___) | |___| |\  |___) / ___ \ | |____| |___| |___ | |
 |____/|_____|_| \_|____/_/   \_\___|    \____|_____|___|
```

**The AI that senses what your code needs — before you ask.**

[![Latest Release](https://img.shields.io/github/release/immunisense/sensai?style=flat-square&color=C4A035)](https://github.com/immunisense/sensai/releases)
[![Build](https://img.shields.io/github/actions/workflow/status/immunisense/sensai/build.yml?style=flat-square&label=build)](https://github.com/immunisense/sensai/actions)
[![Go](https://img.shields.io/badge/go-%3E%3D1.23-00ADD8?style=flat-square&logo=go&logoColor=white)](https://go.dev)
[![License](https://img.shields.io/badge/license-proprietary-333?style=flat-square)](LICENSE.md)

<br />

A secure, terminal-first AI coding assistant.
Spec-driven planning · safe code execution · proxy-backed model access — all in one TUI.

<br />

[Install](#-installation) · [Quick Start](#-quick-start) · [Features](#-core-capabilities) · [Models](#-model-catalog) · [Docs](#-documentation)

</div>

---

## ⚡ Quick Start

```bash
# Install
curl -fsSL https://sensai.immunisense.com/install | bash

# Authenticate
sensai-cli auth login

# Launch
sensai-cli
```

After login, SensAI uses your account defaults for model and reasoning until
you change them in the interface or config.

---

## 📦 Installation

```bash
curl -fsSL https://sensai.immunisense.com/install | bash
```

---

## 🔄 Updating

```bash
sensai-cli update
```

The TUI also shows an "Update now" button when a newer version is available.
Updates are downloaded from the SensAI proxy, verified against checksums, and
the binary is replaced in-place.

---

## 🔐 Authentication

SensAI uses browser-based OAuth. Credentials are stored in the OS keyring
(macOS Keychain, Windows Credential Manager, or Linux `libsecret`/`pass`).

```text
sensai-cli auth login
        │
        ▼
  Open browser → sensai.immunisense.com/sensai-login
  (WSL/SSH/headless: URL printed for manual copy)
        │
        ▼
  User signs in → JWT + refresh token issued
        │
        ▼
  Tokens stored in OS keyring → Bearer JWT with silent refresh
```

No raw model provider credentials are stored on the client.

```bash
sensai-cli auth login    # browser OAuth
sensai-cli auth status   # check login state
sensai-cli auth logout   # clear credentials
```

---

## 🧠 Core Capabilities

<table>
<tr>
<td width="50%" valign="top">

### 🛡️ Security First
- All LLM traffic through `sensai.immunisense.com`
- Zero raw API keys — OAuth + OS keyring
- Pre-flight secrets scanning
- Permission-aware tool execution
- Audit logging & no-log mode

</td>
<td width="50%" valign="top">

### 🤖 Agent Intelligence
- Code, Plan, and Analyze workflows
- Multi-agent orchestration (up to 4 concurrent)
- Custom sub-agents from markdown
- 9 LSP tools for semantic navigation
- Smart MCP integration with circuit breakers

</td>
</tr>
<tr>
<td valign="top">

### 🔍 Search & Navigation
- `search_code` — ripgrep-powered lexical search
- `ast_search` — ast-grep structural pattern matching
- LSP definition, references, hover, symbols
- `@` context resolution (zero-cost, zero-latency)

</td>
<td valign="top">

### ⚙️ Developer Experience
- Conversation checkpoints with full undo
- Auto-formatting (25+ languages)
- Image paste/drop as visual context
- Auto session continuation at 95% context
- Self-update without package manager
- Credit-based billing with 3 balance buckets

</td>
</tr>
</table>

---

## 🎯 Modes

| Mode | Command | Description |
|------|---------|-------------|
| **Code** | `sensai-cli` | Default interactive mode for coding, edits, reviews, and tool use. Toggle `/todos` to force structured task lists. |
| **Plan** | `sensai-cli plan` | Spec-driven planning: requirements → design → tasks → approval gates → task execution |
| **Analyze** | `sensai-cli analyze` | Read-only exploration for safe codebase investigation |

Plan Mode runs three sequential phases (Requirements → Design → Tasks), each
with a TUI approval dialog. After all phases are approved, choose "Run All"
for automatic sequential execution or "Run Manually" to trigger tasks one at a
time with `#run_task:N`. See [`docs/plan_mode.md`](docs/plan_mode.md) for the
full guide.

---

## 📋 Commands

<details>
<summary><b>CLI Commands</b></summary>

| Command | Description |
|---------|-------------|
| `sensai-cli` | Launch the interactive TUI |
| `sensai-cli run "prompt"` | Non-interactive single prompt |
| `sensai-cli plan` | Create or resume a plan |
| `sensai-cli analyze` | Safe read-only exploration |
| `sensai-cli credits` | Show credit balance and burn rate |
| `sensai-cli credits history` | Per-turn consumption history |
| `sensai-cli usage` | Monthly usage stats |
| `sensai-cli topup` | Open manual top-up flow |
| `sensai-cli billing portal` | Open billing portal |
| `sensai-cli invoices` | List invoices |
| `sensai-cli auth status` | Show login state |
| `sensai-cli auth logout` | Clear stored credentials |
| `sensai-cli update` | Download and install the latest release |
| `sensai-cli lsp list` | List configured and running LSP servers |
| `sensai-cli lsp status` | Show LSP server health and diagnostics |
| `sensai-cli mcp add` | Add an MCP server |
| `sensai-cli mcp list` | List configured MCP servers |
| `sensai-cli mcp status` | Show MCP server runtime status |
| `sensai-cli mcp test` | Test MCP server connectivity |
| `sensai-cli mcp enable/disable` | Toggle an MCP server |
| `sensai-cli mcp restart` | Restart an MCP server |
| `sensai-cli mcp remove` | Remove an MCP server |
| `sensai-cli checkpoints list` | List checkpoints for the current session |
| `sensai-cli checkpoints restore` | Restore to a previous checkpoint |
| `sensai-cli agents list` | List all configured agents |
| `sensai-cli agents create` | Create a new custom agent |

</details>

<details>
<summary><b>Slash Commands (TUI)</b></summary>

| Command | Description |
|---------|-------------|
| `/compact` | Summarize conversation, continue in a new session |
| `/review` | Trigger code review workflow |
| `/reasoning` | Select reasoning effort |
| `/model` | Switch the active model |
| `/ctx` | Open the context manager |
| `/formatter` | Configure auto-formatters |
| `/todos` | Toggle To-Do list for Code Mode |
| `/credits` | Show credit balance breakdown |
| `/profile` | Switch or manage model profiles |
| `/sense` | Toggle Sense Mode (full context) |
| `/agents` | Manage and invoke sub-agents |
| `/plan` | Switch to plan mode |
| `/approve` | Approve the current plan phase |
| `/analyze` | Analyze the current codebase |

</details>

<details>
<summary><b>Keyboard Shortcuts</b></summary>

| Shortcut | Action |
|----------|--------|
| `Tab` | Switch focus between editor and chat |
| `Shift+Tab` | Toggle Code Mode ↔ Plan Mode |
| `Ctrl+P` | Command palette |
| `Ctrl+L` | Model picker |
| `Ctrl+S` | Session picker |
| `Ctrl+N` | New session |
| `Ctrl+J` | Insert newline |
| `Ctrl+C` | Quit |
| `Ctrl+G` | Toggle help |
| `Ctrl+V` | Paste image from clipboard |
| `Ctrl+F` | Add image via file picker |

Dialog buttons (permissions, quit, restore, welcome) are also mouse-clickable.

</details>

<details>
<summary><b>Image Context</b></summary>

Paste or drop images directly into the chat as visual context for the AI:

- `Ctrl+V` reads image data from the system clipboard (screenshots, copied images).
- `Ctrl+F` opens a file picker dialog for selecting image files.
- Drag an image file from the file manager into the terminal — the pasted
  file path is detected and attached automatically. `file://` URIs from
  Linux file managers are also handled.
- Supported formats: `.jpg`, `.jpeg`, `.png`, `.gif`, `.webp`. Max 5 MB.
- Models without image support automatically filter out image attachments.

</details>

---

## 💎 Model Catalog

The launch catalog is xAI Grok models, served through the SensAI proxy.

| Model | Type | Base Context | Sense Context |
|-------|------|-------------|---------------|
| Grok Code Fast | Non-reasoning | 256K | — |
| Grok 4.1 Fast (Non-Reasoning) | Non-reasoning | 128K | 2M |
| Grok 4.1 Fast | Reasoning | 128K | 2M |
| Grok 4 Fast | Reasoning | 128K | 2M |
| Grok 4.20 (Non-Reasoning) | Non-reasoning | 200K | 2M |
| Grok 4.20 | Reasoning | 200K | 2M |
| Grok 4.20 (Multi-Agent) | Multi-agent | 200K | 2M |

**Sense Mode** unlocks the full 2M context window. Toggle via `/sense` or the
command palette. Grok Code Fast uses its full 256K at standard context and does
not support Sense mode.

---

## 💰 Billing & Credits

Three balance buckets consumed in order: **tier → bonus → top-up**.
Only tier credits reset each billing cycle — bonus and top-up carry over.

| Tier | Price | Monthly Credits | Model Access |
|------|-------|-----------------|--------------|
| Free | $0 | 50 | `grok-code-fast` |
| Pro | $20 | 500 | All models + all reasoning |
| Ultra | $40 | 1,500 | All models |
| Sense | $100 | 4,000 |  |
| Sense Pro | $200 | 10,000 |  |

```text
$ sensai-cli credits

Tier balance:          3996.50
Bonus balance:         100.00
Top-up balance:        50.00
Total balance:         4146.50
Monthly allocation:    4000
Consumed this period:  3.50
Tier:                  Sense
Period ends:           2026-05-01
```

---

## 🔧 Configuration

Configuration is TOML, loaded from `~/.sensai/config.toml` → `.sensai/config.toml` → `SENSAI_*` env vars.

```toml
model = "grok-code-fast"
reasoning_level = "low"

[tui]
compact_mode = false
diff_mode = "unified"

[secrets_scanner]
mode = "warn"

[permissions]
allowed_tools = ["read_file", "list_dir", "search_code", "ast_search"]
```

<details>
<summary><b>Code Mode To-Do List</b></summary>

When enabled, the agent always creates a structured task list before starting
work and executes tasks one-by-one. Toggle via `/todos` in the TUI or config:

```toml
[options]
todo_list = true
```

The editor info bar shows "To-Do" when active. Disable with `/todos` again or
set `todo_list = false`.

</details>

<details>
<summary><b>Model Profiles</b></summary>

Profiles save a model + provider + reasoning effort as a named preset. Switch
between configurations with a single action via `/profile` in the TUI or the
command palette ("Manage Profiles"). See [`docs/profiles.md`](docs/profiles.md)
for the full guide.

</details>

<details>
<summary><b>LSP Configuration</b></summary>

SensAI auto-detects LSP servers from your toolchain. The agent has 9 LSP
tools: diagnostics, references, definition, hover, symbols, rename, code
actions, formatting, and restart. See [`docs/lsp.md`](docs/lsp.md) for the
full guide.

```toml
# Minimal — auto-detected
[lsp.go]
command = "gopls"

# Full configuration
[lsp.typescript]
command = "typescript-language-server"
args = ["--stdio"]
filetypes = [".ts", ".tsx", ".js", ".jsx"]
root_markers = ["tsconfig.json", "package.json"]
timeout = 30

# Disable all LSP
[options]
auto_lsp = false
```

</details>

<details>
<summary><b>MCP Configuration</b></summary>

MCP servers extend the agent with external tools. SensAI supports stdio, SSE,
and HTTP transports with circuit breaker protection, health monitoring, rate
limiting, response scanning, and audit logging. See [`docs/mcp.md`](docs/mcp.md)
for the full guide.

```toml
[mcp.filesystem]
type = "stdio"
command = "node"
args = ["/path/to/mcp-server.js"]

[mcp.remote]
type = "http"
url = "https://example.com/mcp/"
rate_limit = 60

[mcp.remote.headers]
Authorization = "Bearer $MCP_TOKEN"
```

</details>

<details>
<summary><b>Auto-Formatter</b></summary>

Automatic formatting after every agent write: custom → LSP → 25+ built-in CLI
formatters. See [`docs/formatter.md`](docs/formatter.md) for the full guide.

```toml
# Disable globally
[options]
auto_format = false

# Custom formatter
[formatter.my-sql-formatter]
command = ["pg_format", "$FILE"]
extensions = [".sql"]

# Disable a built-in
[formatter.prettier]
disabled = true
```

</details>

<details>
<summary><b>Custom Agents</b></summary>

Create specialized sub-agents from markdown files. Paid tiers get AI-powered
generation; free tier creates manually. See [`docs/custom_agents.md`](docs/custom_agents.md).

```bash
sensai-cli agents create code-reviewer        # AI-powered (paid)
sensai-cli agents create code-reviewer --manual  # manual (all tiers)
```

Agent files live in `.sensai/agents/` (project) or `~/.sensai/agents/` (global):

```markdown
---
description: Reviews code for quality and best practices
---

You are a code reviewer. Focus on security, performance, and
maintainability. Provide constructive feedback with file:line references.
```

Use `/agents` in the TUI to browse and select agents, or invoke directly
with `#agent:<name> <prompt>`:

```
#agent:code-reviewer review the changes in src/auth/
#agent:explorer where is the database connection configured?
```

</details>

---

## 📖 Documentation

| Guide | Description |
|-------|-------------|
| [Custom Agents](docs/custom_agents.md) | Create specialized sub-agents for your workflows |
| [Auto-Formatter](docs/formatter.md) | Automatic code formatting after every agent edit |
| [LSP Integration](docs/lsp.md) | Language server support for 30+ languages |
| [MCP Integration](docs/mcp.md) | Extend the agent with external tool servers |
| [Plan Mode](docs/plan_mode.md) | Spec-driven planning workflow |
| [Profiles](docs/profiles.md) | Named model presets with task-based routing |

---

## 🔄 Conversation Checkpoints

Automatic per-turn snapshots of file changes with full undo. Restore to any
point in the conversation, reverting all file modifications and truncating
history.

```bash
sensai-cli checkpoints list
sensai-cli checkpoints restore <id>
```

In the TUI, each user message shows a `[ Restore ]` button (click it with the
mouse or press `r` when the message is focused). The confirmation dialog lists
affected files with change indicators (`~` modified, `+` created, `-` deleted).

---

## ⚡ Performance

SensAI uses a cache-first startup strategy to minimize time-to-interactive:

- **Cache-first provider catalog**: model catalog loads from disk cache
  instantly, with a background refresh for the next launch.
- **Migration skip**: a version marker file bypasses migration checks
  when the database is already up-to-date.
- **Lazy agent building**: user-defined sub-agents are registered at startup
  but only constructed on first invocation.
- **Cached model construction**: all agents share a single set of proxy-backed
  models instead of rebuilding per agent.
- **Inline splash**: the startup animation plays within the same alt screen
  session for a seamless transition with no terminal flash.

---

<div align="center">

**Proprietary.** All rights reserved by Immunisense Corp.

[Website](https://immunisense.com) · [Releases](https://github.com/immunisense/sensai/releases) · [Issues](https://github.com/immunisense/sensai/issues) · [Discord](https://discord.gg/a2jafdGrsx)

</div>

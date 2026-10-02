# SensAI IDE and SensAI-Agent

SensAI IDE is the Windows workbench. SensAI-Agent is the same chat, cockpit,
and plans in VS Code, Cursor, or Windsurf. One account, the same credits, and
the same `.sensai/` project files as `sensai-cli`. The editor never holds a
provider API key.

| Surface | Install |
| --- | --- |
| **SensAI IDE** | Windows installer from [immunisense.com/solutions/sensai](https://immunisense.com/solutions/sensai). The engine is included. |
| **SensAI-Agent** | [Marketplace](https://marketplace.visualstudio.com/items?itemName=IMMUNISENSECORP.sensai-ide). Install the CLI and keep `sensai-cli` on `PATH`, or set `sensai.engine.path`. |

On macOS and Linux, use the terminal or SensAI-Agent. An IDE installer for
those systems is not published.

Issues: [`ide`](https://github.com/immunisense/sensai/issues?q=label%3Aide) for the workbench, [`extension`](https://github.com/immunisense/sensai/issues?q=label%3Aextension) for SensAI-Agent.

## Sign in

**SensAI: Sign In** opens [sensai.immunisense.com/sensai-login](https://sensai.immunisense.com/sensai-login). Finish in the browser. If the account uses an authenticator, the IDE asks for the code. **SensAI: Account** shows email, plan, and credits. The credits pill in the status bar opens the same card.

SensAI IDE stays on the sign-in window until you are signed in.

## Sense Chat

Open **Sense Chat** (`Ctrl+Alt+L`, `Cmd+Alt+L` on macOS). **SensAI: Connect Engine** starts the engine for the open folder. In SensAI IDE that binary is bundled. In VS Code, Cursor, or Windsurf it is `sensai-cli` on `PATH`.

### Workflows

| Workflow | What it does |
| --- | --- |
| **Code** | Tools, edits, and permission prompts |
| **Plan** | Spec first. See [plan_mode.md](plan_mode.md) |
| **Analyze** | Read-only |
| **Security** | Sense Protocol hunt. Included with Sense, Sense Pro, and Sense Ultra. The menu stays visible on other plans and explains the requirement |
| **Chat** | Conversation only. Choose it on the empty Sense Chat screen, or set `sensai.chat.defaultMode` to `chat` |

The composer menu switches Code, Plan, Analyze, and Security. **Design** (`/design` in the terminal) is not in that menu.

**Autopilot** on: SensAI edits and runs commands without asking. Off: every write and shell call shows Allow, Allow for session, or Deny. Saved per workspace.

**Model, effort, and Sense.** Models that offer Sense Mode (full context at the higher rate, the terminal's `/sense`) have a Sense switch in the model menu. The model button shows a Sense badge while it is on. **Auto** reasoning picks an effort for the prompt from the levels that model exposes.

**Profile.** The picker next to Autopilot uses the same profiles as `/profile`. While one is active, the model and effort pickers step aside. **Model & effort** brings them back. Manage profiles in **SensAI: Configure MCP Servers & Hooks** → Profiles. See [profiles.md](profiles.md).

### While you work

- Replies stream, with a collapsible thinking block and a tool timeline. Rules, skills, and agents pulled into context fold into Included rules, Included skills, or Included agents.
- Sub-agent cards show who is running. The reply footer counts their credits with the turn.
- Session tabs restore when you reopen the workspace. `+` starts a new chat (`Ctrl+Alt+N`). The list opens every stored session for the folder, including terminal sessions.
- Type `@` for a file or folder. `@repo:<name>/…` reads another workspace folder. `#<name>` points the agent at a folder without loading its files.
- Paste or drop up to eight PNG, JPEG, GIF, or WebP images, 5 MB each. A model without image input tells you so.
- Each prompt has **Checkpoint · Restore**. Restore puts that prompt back in the composer. If the turn changed files, Restore asks, then reverts those files and drops later turns.
- Copy a reply, or open `…` for Copy as Markdown, Copy conversation ID, and Copy request IDs. The footer shows estimated credits and elapsed time.
- `Esc` or the stop button cancels the turn.
- A failed turn shows an error banner with a title and a detail.

### Keyboard

These shortcuts are fixed.

| Key | Action |
| --- | --- |
| Enter | Send |
| Shift+Enter | New line |
| Esc | Stop the turn. In the session list, return to the chat |
| `@` | Mention a file or folder |
| `#` | Focus a workspace folder |
| Ctrl+Alt+N | New session (`Cmd+Alt+N` on macOS) |
| Ctrl+Alt+L | Focus Sense Chat |
| Ctrl+Alt+K | Add the selection as context |
| Tab | Accept an autocomplete suggestion |

**SensAI: Keyboard Shortcuts** opens this list in Settings.

## Autocomplete

Inline suggestions while you type. Tab accepts. Toggle with the status item, **SensAI: Toggle Autocomplete**, or `sensai.autocomplete.enabled`. Suggestions spend credits.

## Cockpit

The **SensAI** view lists the engine, rules, agents, skills, MCP servers, and hooks. Each group splits into **Workspace** (`<repo>/.sensai/…`) and **Global** (`~/.sensai/…`).

- `+` adds a rule, agent, or skill as a starter file. MCP servers and hooks open the configuration panel.
- Enable or disable one item, or a whole Workspace or Global folder. Disabled items are muted. The terminal sees the same on/off state.
- MCP rows show Connected (with a tool count), Connecting, Disabled, Connection failed with **Retry**, or Unauthenticated with **Authenticate**. A connected server expands to its tools.

## Configuration panel

**SensAI: Configure MCP Servers & Hooks** edits servers and hooks without opening `config.toml` by hand. Secrets are stored in the OS keyring. The same panel's Profiles tab edits [profiles](profiles.md). Hooks are described in [hooks.md](hooks.md). MCP fields are described in [mcp.md](mcp.md).

## Plans and tasks

**Plans** lists `.sensai/plans` with each plan's phase. **New Plan** starts Plan Mode. Approved phases are `01-requirements.md`, `02-design.md`, and `03-tasks.md` next to `plan.json`. The terminal can resume or apply that folder. See [plan_mode.md](plan_mode.md).

**Tasks** lists isolated worktrees for this workspace: status, review notes, compare, and schedules. `sensai-cli tasks` uses the same worktrees. Compare asks before it spends credits on two runs. Schedules are off until you enable one.

## Editor actions

Right-click in the editor:

| Command | What it does |
| --- | --- |
| Click to Fix | Send the diagnostic to Sense Chat |
| Explain Selection | Ask about the selection |
| Review File | Read-only review of the file |
| Add Selection as Context | Attach the selection (`Ctrl+Alt+K`) |
| Add File as Context | Attach the file |

The explorer has Review File and Add File as Context.

## Several folders

A workspace with more than one folder keeps `<first folder>/.sensai/workspace.toml` in step with the folders, so the engine can open each repo. Mention files as `@repo:<name>/path`. `#<name>` focuses a repo without loading it.

## What stays in the terminal

Sense Chat runs `/` skills and the local commands that answer in text (`/pin`, `/test`, `/lint`, `/pack`, `/help`, `/doctor`, and the others listed by `/help`). These open a terminal dialog or a local picker, so use them in `sensai-cli`:

`/fork`, `/wide`, `/replay`, `/rewind` (Sense Chat uses **Restore**), `/design`, `/vim`, `/voice`, `/split`, `/add-dir`, `/tag`, `/diff`, `/search`.

Model, effort, Sense Mode, and profile are the composer pickers.

## Commands

| Command | What it does |
| --- | --- |
| SensAI: Sign In / Sign Out | Browser login, or clear the session |
| SensAI: Account | Plan, credits, sign out, manage plan |
| SensAI: Connect Engine | Start the engine for this folder |
| SensAI: Focus Sense Chat / Toggle Sense Chat | Show the chat |
| SensAI: New Chat Session / Session List | New tab, or any stored session |
| SensAI: Stop Current Turn | Cancel the reply |
| SensAI: Toggle Autocomplete | Tab suggestions |
| SensAI: Add Rule, Agent, Skill, MCP Server, or Hook | Cockpit `+` |
| SensAI: Configure MCP Servers & Hooks | Servers, hooks, profiles |
| SensAI: New Plan | Plan Mode for this workspace |
| SensAI: Open config.toml | Engine config for this workspace or your profile |
| SensAI: About | Version, engine, workbench |
| SensAI: Report Issue | GitHub issue for this surface |

## Settings

Leave a setting on **default** to follow `config.toml`. See [cli.md](cli.md) for that file.

| Setting | Default | Notes |
| --- | --- | --- |
| `sensai.engine.path` | empty | Engine binary. A workspace cannot override it |
| `sensai.engine.debug` | off | Verbose engine log |
| `sensai.chat.defaultMode` | `code` | `code`, `plan`, `analyze`, `chat`, `security` |
| `sensai.chat.defaultModel` | empty | Used until the workspace has its own pick |
| `sensai.chat.defaultReasoning` | empty | Includes `auto` |
| `sensai.autocomplete.enabled` | on | Tab autocomplete |
| `sensai.notifications.actionRequired` | on | Ask when Sense Chat is not visible |
| `sensai.notifications.failure` | off | Turn failed |
| `sensai.notifications.success` | on | Reply finished and the chat is hidden |
| `sensai.notifications.billing` | on | Low credit, and the account card at the limit |
| `sensai.credits.statusBar` | on | Balance in the status bar |
| `sensai.agent.autoDiagnose` | default | Check changed files after a turn |
| `sensai.agent.autoCraft` | default | Craft check on written lines |
| `sensai.agent.todoList` | default | Code Mode to-do list |
| `sensai.agent.autoSummarize` | default | Summarize near the context limit, then continue |
| `sensai.agent.summarizeModel` | empty | Model for that summary. Empty follows `config.toml` (Gemma 4). `current` uses the chat model |
| `sensai.secretsScanner.mode` | default | `warn` or `block` |
| `sensai.senseEngineer.level` | default | `light`, `full`, `ultra`, or `auto` |
| `sensai.permissions.skipRequests` | default | Skip permission prompts. Autopilot is the per-workspace switch |

## Updates

SensAI IDE updates from the title bar. A gold Update pill names the current and latest versions. The Manage gear shows a gold dot at the same time. Check for Updates uses the Account window: up to date names the version you have; an update installs, then asks you to restart.

A newer CLI engine can download in the background. The settings cog then asks for a restart. That is separate from an IDE update. In VS Code, Cursor, or Windsurf, update the CLI with `sensai-cli update`.

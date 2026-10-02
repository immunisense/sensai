# CLI and terminal

`sensai-cli` with no arguments opens the terminal UI. The commands below are the same binary. SensAI IDE and SensAI-Agent are documented in [ide.md](ide.md).

Global flags, on any command:

| Flag | What it does |
| --- | --- |
| `-c`, `--cwd` | Working directory |
| `-D`, `--data-dir` | SensAI data directory |
| `-d`, `--debug` | Debug log |
| `-y`, `--yolo` | Accept permission prompts for this run |
| `-s`, `--session` | Continue a session by ID |
| `-C`, `--continue` | Continue the most recent session |

`--session` and `--continue` cannot be combined.

## Chat

| Command | What it does |
| --- | --- |
| `sensai-cli` | Interactive terminal |
| `sensai-cli run "prompt"` | One prompt, then exit. Stdin is prepended when it is a pipe or a file |
| `sensai-cli run --json "prompt"` | The same, as events: `session`, `text`, `done`, `error` |
| `sensai-cli plan` | Create or resume a plan. See [plan_mode.md](plan_mode.md) |
| `sensai-cli analyze "prompt"` | Read-only tools in a throwaway worktree |
| `sensai-cli session list` | Sessions |
| `sensai-cli session show <id>` | One session |
| `sensai-cli session last` | Most recent session |
| `sensai-cli session rename <id> <title>` | Rename |
| `sensai-cli session delete <id>` | Delete |
| `sensai-cli agents list` | Custom agents |
| `sensai-cli agents create [name]` | Create one. The prompt spends credits. `--manual` skips the model call. See [custom_agents.md](custom_agents.md) |
| `sensai-cli ctx` | Context size for the current session |
| `sensai-cli checkpoints list` | Per-turn snapshots |
| `sensai-cli checkpoints restore <id>` | Restore files and trim the conversation |
| `sensai-cli pr` | Open a GitHub pull request from the session worktree |
| `sensai-cli tasks list` | Isolated task worktrees |
| `sensai-cli tasks create <name>` | Create one |
| `sensai-cli tasks diff [id]` | Diff a worktree, or the current repo when `id` is omitted |
| `sensai-cli tasks archive <id>` | Remove the worktree. The branch is kept |

## Account

Browser sign-in. On a machine without a browser, the login URL is printed.

| Command | What it does |
| --- | --- |
| `sensai-cli auth login` | Sign in. `sensai-cli login` is the same command |
| `sensai-cli auth status` | Who is signed in |
| `sensai-cli auth logout` | Sign out and remove the stored login |
| `sensai-cli auth mfa status` | Authenticator enrollment |
| `sensai-cli auth mfa enroll` | Turn on an authenticator and confirm the code |
| `sensai-cli auth mfa backup-codes` | Make a new set of backup codes |
| `sensai-cli auth mfa disable` | Turn off an authenticator |
| `sensai-cli credits` | Balance: tier, bonus, then top-up |
| `sensai-cli credits history` | Credit history |
| `sensai-cli usage` | Usage this period |
| `sensai-cli topup` | Buy credits in the browser |
| `sensai-cli invoices` | Past invoices |
| `sensai-cli billing portal` | Payment method, plan, and invoices |

Passkeys, backup codes, and API keys are also on the [account page](https://sensai.immunisense.com/). An API key is shown once and is for scripts. It can call chat and read usage. Changing an authenticator, backup codes, a passkey, an API key, or other signed-in devices asks you to confirm again.

## Project

| Command | What it does |
| --- | --- |
| `sensai-cli init` | Create a SensAI project in this directory |
| `sensai-cli config` | View or update configuration |
| `sensai-cli dirs config` | Config directory |
| `sensai-cli dirs data` | Data directory |
| `sensai-cli projects` | Projects SensAI has opened |
| `sensai-cli models` | Models on your plan |
| `sensai-cli workspace init` | Create `.sensai/workspace.toml` |
| `sensai-cli workspace add <path>` | Add a repo. `--name` sets the `@repo:` name |
| `sensai-cli workspace list` | Repos in the workspace |
| `sensai-cli lsp list` | Language servers |
| `sensai-cli lsp status` | Health and diagnostic counts |
| `sensai-cli mcp` | MCP servers. See [mcp.md](mcp.md) |
| `sensai-cli hooks` | Lifecycle hooks. See [hooks.md](hooks.md) |

Mention another repo as `@repo:<name>/path` or `name:path/to/file`.

## System

| Command | What it does |
| --- | --- |
| `sensai-cli update` | Download the latest release and verify its checksum |
| `sensai-cli version` | Print the version |
| `sensai-cli changelog` | What changed since the version you last ran |
| `sensai-cli changelog --since v0.4.5` | Notes after that version |
| `sensai-cli changelog --product ide` | `cli`, `web`, `ide`, `models`, or `platform` |
| `sensai-cli logs` | Local log |
| `sensai-cli stats` | Local usage |
| `sensai-cli stats tools` | Tool calls, error rate, and edit failures from local sessions |
| `sensai-cli audit export` | Export the local audit log |
| `sensai-cli update-providers` | Refresh the cached model list |
| `sensai-cli uninstall` | Remove the login, `~/.sensai/`, and this binary. Asks first. `--force` skips the question |

`/release-notes` in the terminal shows the same notes as `sensai-cli changelog`. After an update, a What's new card appears once. If your version is older than a critical or high security fix, the terminal shows a banner.

Pick one installer per machine. Mixing the script installer with npm puts two binaries on `PATH`.

Remove only the npm copy, and keep the login:

```bash
npm uninstall -g sensai-cli
```

Remove only the Windows script copy, and keep the login: delete `%LOCALAPPDATA%\sensai\bin\sensai-cli.exe`, drop that folder from the user `PATH`, and delete the `# sensai-cli PATH` block from your PowerShell profile.

## Keyboard

| Key | Action |
| --- | --- |
| Enter | Send |
| Shift+Enter, Ctrl+J, Alt+Enter | New line |
| Shift+Tab | Cycle modes. Code → Security → Plan → Chat on Sense, Sense Pro, and Sense Ultra. Security is skipped on other plans |
| Tab | Move focus. Tab also accepts the suggested next prompt when it is showing |
| Ctrl+P | Command palette |
| Ctrl+L | Model picker. Ctrl+M is the same key |
| Ctrl+S | Session list |
| Ctrl+N | New session. A turn already running keeps its language servers |
| Ctrl+B | Rewind |
| Ctrl+Alt+F | Fork this session into a worktree |
| Ctrl+Y | Toggle yolo for this session |
| Ctrl+O | Open the prompt in `$EDITOR` |
| Ctrl+F | Attach an image |
| Ctrl+V | Paste an image from the clipboard |
| Ctrl+G | Help |
| Ctrl+C | Quit. The first Ctrl+C during a turn interrupts the turn |
| Esc | Cancel the current dialog |
| `@` | Mention a file |
| `!` | Run a local command. The output joins the session after a secrets scan |
| Up / Down | Previous and next prompt |
| Ctrl+R | While idle, search past prompts |

`/keybindings` writes a template you can edit. `/terminal-setup` explains newline keys when Shift+Enter does not arrive.

## Slash commands

Type `/` in the composer. Skills are in [skills.md](skills.md). A skill you add shows up in the same list.

### Modes and models

| Command | What it does |
| --- | --- |
| `/code` `/plan` `/chat` `/analyze` `/design` | Switch mode. `/security` switches to Security Mode, or explains which plans include it |
| `/approve` | Approve the current plan phase |
| `/model` | Model picker |
| `/reasoning` | Effort picker, including Auto |
| `/default` | Save the current model and effort for new sessions. Auto can be that default |
| `/summarize-model` | Model for `/compact` and automatic summary. **Chat model** keeps the conversation's model |
| `/sense` | Full context at the long-context rate |
| `/profile` | Named model presets. See [profiles.md](profiles.md) |
| `/fast` | Use the profile named `fast`, or keep the model and lower effort |
| `/todos` | Code Mode to-do list |
| `/advisor` | A second model that only comments |

### Session

| Command | What it does |
| --- | --- |
| `/rewind` | Preview, then restore a checkpoint. Ctrl+B |
| `/fork` | Clone this session into a parallel worktree |
| `/branch` | Copy the conversation into a new session, without a worktree |
| `/replay` | Re-run the last prompt |
| `/compact` | Summarize, then continue. Automatic when the window is 80–95% full (90% on smaller windows) |
| `/export` | Export the conversation to markdown |
| `/sessions` | Session picker |
| `/split` | Split the view |
| `/tag` | Label this session |
| `/copy` | Copy the last reply, or its first code block |
| `/btw` | A side question that stays out of this turn |
| `/cost` `/usage` `/credits` | This turn, this session, and the account balance |
| `/context` `/ctx` `/files` | What is in the window, and which files were read |

### Work

| Command | What it does |
| --- | --- |
| `/review` | Review recent changes |
| `/wide` | Isolated design alternatives, then a shortlist |
| `/test` `/lint` | Run the turn's checks. `/test <command>` sets the command for this session |
| `/pin` `/unpin` | Keep a file in the prompt and block writes to it until `/unpin` |
| `/pack` | Copy a secrets-scanned pack of what this session read |
| `/commit` | Draft a commit message and stop before push |
| `/pr` | Open a pull request from the session worktree |
| `/pr-comments` | Fetch review comments |
| `/diff` | `git diff --stat` |
| `/search` | Search the workspace |
| `/tasks` `/watch` | Queued work, and how to focus one agent |
| `/add-dir` | Add a directory for this session |

### Setup

| Command | What it does |
| --- | --- |
| `/help` | Composer and slash help |
| `/doctor` | Version, sandbox, folder trust, language servers |
| `/formatter` | Formatters. See [formatter.md](formatter.md) |
| `/mcp` `/mcp-tools` | Servers, and the tools they loaded. See [mcp.md](mcp.md) |
| `/lsp` | Language servers that are not running. See [lsp.md](lsp.md) |
| `/permissions` | Saved command allows. `/permissions allow <prefix>` adds one |
| `/sandbox` | Whether the shell sandbox is on |
| `/trust` | Trust this folder for tools. The answer has to be `/trust yes` |
| `/memory` | Path to `SENSAI.md` |
| `/create-sensai` | Create `SENSAI.md`, or import rules, skills, and agents from another assistant |
| `/sense-engineer` | Packing level `light`, `full`, `ultra`, or `auto`. `/compress` is the same command |
| `/output-style` | Default, explanatory, or learning |
| `/vim` | Vim keys in the composer |
| `/voice` | Voice capture |
| `/agents` | List and invoke sub-agents |
| `/feedback` | Where to report an issue |
| `/release-notes` | What changed |
| `/insights` | A short usage note |

Sense Chat runs the skills and the commands that reply with text. Commands that open a terminal picker are listed in [ide.md](ide.md).

## Configuration

Two TOML files. The project file wins over the global file. `SENSAI_*` environment variables win over both.

| File | Who it applies to |
| --- | --- |
| `~/.sensai/config.toml` | Every project |
| `.sensai/config.toml` | This project |

`sensai-cli dirs config` prints that global directory. It is `~/.sensai`.

```toml
model = "grok-build-0.1"
reasoning_level = "high"   # or "auto". /default saves the current pair

[tui]
compact_mode = false
diff_mode = "unified"

[secrets_scanner]
mode = "warn"               # warn, block, or off

[compression]               # Sense Engineer. [sense_engineer] is the same table
enabled = true
level   = "full"            # light | full | ultra | auto

[options]
summarize_model = "gemma-4-31b"  # /compact and auto-summarize. "current" uses the chat model

[plan]
tdd_enabled = false
```

Formatters, MCP servers, language servers, and agents have their own pages: [formatter.md](formatter.md), [mcp.md](mcp.md), [lsp.md](lsp.md), [custom_agents.md](custom_agents.md). IDE settings that follow this file are in [ide.md](ide.md).

Search in a turn tries, in order: the code map, `search_code`, language-server symbols, then `ast_search`.

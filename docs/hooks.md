# Hooks

A hook runs when something happens in a session: a tool call, a saved file, the start or end of a turn. It can run a command, ask an agent, or show a notification. The terminal, SensAI IDE, and SensAI-Agent share the same hooks.

Create them in the terminal, in `config.toml`, or in **SensAI: Configure MCP Servers & Hooks**.

## Create

```bash
sensai-cli hooks list
sensai-cli hooks create --event file_edited --file-patterns "*.ts,*.tsx" --command "npm run lint"
sensai-cli hooks create --event pre_tool_use --tool-types "shell" --command "./scripts/guard.sh"
sensai-cli hooks toggle <hook-id>
sensai-cli hooks run <hook-id>
sensai-cli hooks delete <hook-id>
```

`--workspace` saves the hook in the project config. Without it, the hook is global. `--once` runs the hook once and then disables it.

In the IDE, `+` on Hooks opens the same form: name, event, tool types or file patterns, action, timeout, and Workspace or Global.

## Actions

| Action | What it does |
| --- | --- |
| Run a command | Shell command. Placeholders: `{{tool_name}}`, `{{file_path}}`, `{{tool_output}}`. Exit code 2 stops the turn |
| Ask an agent | Sends a prompt to a named agent |
| Notify | Shows a message |

A command can be limited with `--if`, for example `run_shell(git *)`. Leave it empty to always run.

The CLI can also set an `http` action (`--url`) or a `prompt` action.

## Events

| Group | Events |
| --- | --- |
| Session | `session_start`, `session_end`, `setup`, `prompt_submit`, `pre_compact`, `post_compact` |
| Tools | `pre_tool_use`, `post_tool_use`, `post_tool_use_failure` |
| Files | `file_edited`, `file_created`, `file_deleted`, `file_changed` |
| Agents | `agent_stop`, `subagent_start`, `subagent_stop`, `user_triggered` |
| Permissions | `permission_request`, `permission_denied` |
| Plan | `pre_task_execution`, `post_task_execution`, `on_phase_approved`, `on_task_completed`, `on_plan_applied` |
| Other | `cwd_changed`, `worktree_create`, `worktree_remove`, `instructions_loaded`, `config_change`, `elicitation`, `elicitation_result`, `file_suggestion`, `on_diagnostics_error` |

`user_triggered` is what `sensai-cli hooks run` fires. File events take `--file-patterns`. Tool events take `--tool-types`.

## Limits

- A hook can add context to a turn or stop the turn.
- A hook cannot hide a failed tool.
- A hook cannot allow a path outside the workspace or turn the sandbox off.
- The Cockpit can enable or disable a hook. The terminal sees that change.

`/sense-hook` in the terminal walks you through creating one.

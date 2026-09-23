# Profiles

Profiles are named presets that map different tasks to different models
within a single configuration. Instead of manually switching models when
you go from coding to research to planning, a profile routes each task
type to the right model automatically.

## Concept

A profile is a named collection of task-to-model mappings. Each task type
(coding, research, planning, analysis, etc.) can use a different model
with its own reasoning effort. When a profile is active, SensAI
automatically selects the appropriate model based on what the agent is
doing.

```
Profile: "daily-driver"
├── default      → grok-4.3 (medium)
├── coding       → grok-build-0.1
├── research     → grok-4.6 (high)
├── planning     → grok-4.5 (medium)
├── analysis     → grok-4.3 (medium)
├── review       → grok-4.6 (high)
└── subagent     → grok-build-0.1
```

All models in a profile must come from the available catalog. The `default`
task is the fallback when no specific task mapping matches.

## Task Types

| Task        | When It Applies                                              |
|-------------|--------------------------------------------------------------|
| `default`   | Fallback for any task not explicitly mapped                  |
| `coding`    | Code Mode — writing, editing, and refactoring code           |
| `research`  | Web search, codebase exploration, answering questions         |
| `planning`  | Plan Mode — requirements, design, and task generation         |
| `analysis`  | Safe Analysis Mode — read-only codebase exploration           |
| `review`    | Code review via `/review`, `code_review`, or a *review* agent |
| `subagent`  | Fallback for spawned sub-agents when no more specific task matches |

Each task mapping includes:

| Field             | Required | Description                                      |
|-------------------|----------|--------------------------------------------------|
| `provider`        | yes      | Provider ID (e.g. `xai`)                         |
| `model`           | yes      | Model ID from the available catalog               |
| `reasoning_effort`| no       | `low`, `medium`, `high`, or empty for model default |

## Quick Start

```bash
# Open the profiles dialog in the TUI
/profile

# Or use the command palette
Ctrl+P → "Manage Profiles"
```

## Creating a Profile from the TUI

1. Type `/profile` to open the dialog
2. Press `n` or select "+ New profile"
3. Press `Tab` to choose **Global** (all projects) or **Project**
   (this workspace)
4. Enter a profile name and press `Enter`
5. The task editor opens with all task types listed
6. Use `↑`/`↓` to navigate tasks
7. Press `Enter` to cycle through available models for the selected task
8. Press `Tab` to cycle reasoning effort for the selected task's model
   (no-op when the model has no effort picker)
9. Press `x` to clear a task mapping (reverts to default fallback)
10. Press `s` to save the profile

The default task is pre-populated with your current model. Other tasks
show "(uses default)" until you assign a specific model.

## Editing an Existing Profile

1. Open `/profile`
2. Navigate to the profile you want to edit
3. Press `e` to enter the task editor
4. Modify task assignments as needed
5. Press `s` to save

## Automatic Task Routing

When a profile is active, SensAI applies the right model on every turn
and whenever you switch modes:

- Code Mode and Design Mode apply the `coding` task model
- Plan Mode applies the `planning` task model
- Chat Mode applies the `research` task model
- Analyze Mode and Security Mode apply the `analysis` task model
- `/review` (and Code Review in the command palette) applies the `review`
  task model for that turn
- Spawned sub-agents pick a task from what they do, without changing
  the parent model: `explore` / `scout` / default `task` → `research`;
  `designer` and other writable agents → `coding`; `code_review` and
  *review* agents → `review`; *plan* / *analy* in the name →
  `planning` / `analysis`. If that task is unmapped, `subagent` is used,
  then `default`
- A model pinned to an agent by name (`agents.<id>`, see below) wins
  over that agent's task
- If no specific task mapping exists, the `default` model is used

Saving or activating a profile applies the current mode's mapping
immediately. Picking a model from `/model` deactivates the profile so
the one-off choice is not overwritten.

The `credits used` line of a turn covers every agent that worked for it:
the main agent's steps plus each sub-agent run at the model and effort
the profile routed it to.

### Agent pins

`agents` maps a sub-agent id (`explore`, `scout`, `designer`, `task`, or
the file name of a `.sensai/agents` agent) to a model. It is edited in
SensAI IDE (Configuration › Profiles, "By agent"); the TUI dialog keeps
pins it does not show.

```toml
[profiles.agents.designer]
provider = "xai"
model = "grok-4.7"
reasoning_effort = "high"
```

The editor info bar shows the active profile name at the end:
`Code · Grok Build · daily-driver`

Switching to Plan Mode: `Plan · Grok 4.5 · Medium · daily-driver`

## Configuration

Global profiles are stored in the user data config
(`~/.local/share/sensai/sensai.json` on Unix,
`%LOCALAPPDATA%\sensai\sensai.json` on Windows). Project profiles are
stored in the workspace file (`.sensai/sensai.json`) and apply only in
that repo. A project profile that is active wins over a global one.

Each file has a `profiles` array and an `active_profile` key.

### Example: Multi-Model Profile

```toml
active_profile = "daily-driver"

[[profiles]]
name = "daily-driver"

[profiles.tasks.default]
provider = "xai"
model = "grok-4.3"
reasoning_effort = "medium"

[profiles.tasks.coding]
provider = "xai"
model = "grok-build-0.1"

[profiles.tasks.research]
provider = "xai"
model = "grok-4.6"
reasoning_effort = "high"

[profiles.tasks.planning]
provider = "xai"
model = "grok-4.5"
reasoning_effort = "medium"

[profiles.tasks.analysis]
provider = "xai"
model = "grok-4.3"
reasoning_effort = "medium"

[profiles.tasks.review]
provider = "xai"
model = "grok-4.6"
reasoning_effort = "high"

[profiles.tasks.subagent]
provider = "xai"
model = "grok-build-0.1"
```

### Example: Simple Single-Model Profile

If you only set `default`, every task uses the same model. This is the
simplest profile — equivalent to the old single-model behavior.

```toml
[[profiles]]
name = "all-fast"

[profiles.tasks.default]
provider = "xai"
model = "grok-build-0.1"
```

### Example: Cost-Conscious Profile

Use the cheapest model for most tasks, reserve the flagship for research
and planning where deep reasoning matters.

```toml
[[profiles]]
name = "budget"

[profiles.tasks.default]
provider = "xai"
model = "grok-build-0.1"

[profiles.tasks.research]
provider = "xai"
model = "grok-4.3"
reasoning_effort = "medium"

[profiles.tasks.planning]
provider = "xai"
model = "grok-4.5"
reasoning_effort = "high"
```

## Available Models

All models in a profile must be from the live catalog:

| Model ID | Display Name | Type |
|----------|--------------|------|
| `grok-build-0.1` | Grok Build | Reasoning (auto, no picker) |
| `grok-4.3` | Grok 4.3 | Reasoning |
| `grok-4.5` | Grok 4.5 | Reasoning |
| `grok-4.6` | Grok 4.6 | Reasoning |
| `grok-4.7` | Grok 4.7 | Reasoning |
| `claude-sonnet-5` | Claude Sonnet 5 | Reasoning |
| `claude-opus-5` | Claude Opus 5 | Reasoning |
| `claude-opus-5-5` | Claude Opus 5.5 | Reasoning (default medium) |
| `claude-fable-5.1` | Claude Fable 5.1 | Reasoning |
| `gpt-6-astra` | GPT-6 Astra | Reasoning |
| `gpt-6-sol` | GPT-6 Sol | Reasoning (default medium) |
| `gpt-6-luna` | GPT-6 Luna | Reasoning (default medium) |
| `gpt-5.6-sol` | GPT-5.6 Sol | Reasoning |
| `gpt-5.6-terra` | GPT-5.6 Terra | Reasoning |
| `gpt-5.6-luna` | GPT-5.6 Luna | Reasoning |
| `glm-5.3` | GLM-5.3 | Reasoning (low/high/max) |
| `glm-5.3-flash` | GLM-5.3 Flash | Reasoning (low/high/max) |
| `kimi-k3` | Kimi K3 | Reasoning (low/medium/high) |
| `minimax-m3` | MiniMax M3 | Reasoning (auto, no picker) |
| `deepseek-v4-flash` | DeepSeek V4 Flash | Reasoning (none/high/max) |
| `deepseek-v4.1-flash` | DeepSeek V4.1 Flash | Reasoning (low/high/max) |
| `deepseek-v4-pro` | DeepSeek V4 Pro | Reasoning (none/high/max) |
| `gemma-4-31b` | Gemma 4 | Reasoning (auto, no picker) |
| `gemini-3.8-flash` | Gemini 3.8 Flash | Reasoning (low/medium/high) |

Retired IDs still resolve: `grok-code-fast` → `grok-build-0.1`; `grok-4-1-fast-*` and `grok-4-fast-*` → `grok-4.3`. Models without a picker ignore `reasoning_effort`. Other reasoning models default to the model's standard effort when the field is omitted. Effort does not change the credit rate.

## Keyboard Shortcuts

### Profile List

| Key        | Action                          |
|------------|---------------------------------|
| `↑` / `↓`  | Navigate profiles              |
| `Enter`    | Activate selected profile       |
| `n`        | Create new profile              |
| `e`        | Edit tasks for selected profile |
| `d`        | Delete selected profile         |
| `Esc`      | Close dialog                    |
| Type       | Filter by name, model, or scope |

### New Profile

| Key        | Action                          |
|------------|---------------------------------|
| Type       | Profile name                    |
| `Tab`      | Toggle Global / Project         |
| `Enter`    | Continue to task editor         |
| `Esc`      | Cancel and return to list       |

### Task Editor

| Key        | Action                          |
|------------|---------------------------------|
| `↑` / `↓`  | Navigate task types            |
| `Enter`    | Cycle model for selected task   |
| `Tab`      | Cycle reasoning effort          |
| `x`        | Clear task (use default)        |
| `s`        | Save profile                    |
| `Esc`      | Cancel and return to list       |

## Backward Compatibility

Profiles created before the task-based routing update (single-model
profiles with just `provider`, `model`, and `reasoning_effort` fields)
continue to work. They are treated as having a single `default` task.
The task editor will show the existing model as the default task, and
you can add task-specific mappings on top.

## Tips

- Always define a `default` task. It's the safety net for any context
  that doesn't match a specific task type.
- Use fast, cheap models for `coding` and `subagent` — these run
  frequently and burn the most credits.
- Reserve flagship models (`grok-4.7`, `grok-4.6`, `grok-4.5`) for `research` and
  `planning` where deep reasoning pays off.
- Sense mode is independent of profiles. Toggling `/sense` applies to
  whichever model is active for the current task.
- Profile names should be descriptive: `daily-driver`, `budget`,
  `deep-research`, `plan-heavy`.
- Task mappings are optional. A profile with only `default` behaves like
  a single-model preset — you can start simple and add task mappings
  later as your workflow evolves.

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
├── default      → grok-4-1-fast-reasoning (medium)
├── coding       → grok-code-fast
├── research     → grok-4.20-reasoning (high)
├── planning     → grok-4.20-reasoning (medium)
├── analysis     → grok-4-fast-reasoning (medium)
├── review       → grok-4.20-reasoning (high)
└── subagent     → grok-4-1-fast-reasoning (low)
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
| `review`    | Code review via `/review` or the code-reviewer agent          |
| `subagent`  | Sub-agent invocations (custom agents, task agent)             |

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
3. Enter a profile name and press `Enter`
4. The task editor opens with all task types listed
5. Use `↑`/`↓` to navigate tasks
6. Press `Enter` to cycle through available models for the selected task
7. Press `x` to clear a task mapping (reverts to default fallback)
8. Press `s` to save the profile

The default task is pre-populated with your current model. Other tasks
show "(uses default)" until you assign a specific model.

## Editing an Existing Profile

1. Open `/profile`
2. Navigate to the profile you want to edit
3. Press `e` to enter the task editor
4. Modify task assignments as needed
5. Press `s` to save

## Automatic Task Routing

When a profile is active, SensAI automatically applies the right model
based on the current context:

- Switching to Code Mode (`Shift+Tab` or default) applies the `coding`
  task model
- Switching to Plan Mode (`/plan` or `Shift+Tab`) applies the `planning`
  task model
- If no specific task mapping exists, the `default` model is used

The editor info bar shows the active profile name at the end:
`Code · Grok Code Fast · daily-driver`

Switching to Plan Mode: `Plan · Grok 4.20 · Medium · daily-driver`

## Configuration

Profiles are stored in the global config (`~/.sensai/config.toml`) under
the `profiles` array and `active_profile` key.

### Example: Multi-Model Profile

```toml
active_profile = "daily-driver"

[[profiles]]
name = "daily-driver"

[profiles.tasks.default]
provider = "xai"
model = "grok-4-1-fast-reasoning"
reasoning_effort = "medium"

[profiles.tasks.coding]
provider = "xai"
model = "grok-code-fast"

[profiles.tasks.research]
provider = "xai"
model = "grok-4.20-reasoning"
reasoning_effort = "high"

[profiles.tasks.planning]
provider = "xai"
model = "grok-4.20-reasoning"
reasoning_effort = "medium"

[profiles.tasks.analysis]
provider = "xai"
model = "grok-4-fast-reasoning"
reasoning_effort = "medium"

[profiles.tasks.review]
provider = "xai"
model = "grok-4.20-reasoning"
reasoning_effort = "high"

[profiles.tasks.subagent]
provider = "xai"
model = "grok-4-1-fast-reasoning"
reasoning_effort = "low"
```

### Example: Simple Single-Model Profile

If you only set `default`, every task uses the same model. This is the
simplest profile — equivalent to the old single-model behavior.

```toml
[[profiles]]
name = "all-fast"

[profiles.tasks.default]
provider = "xai"
model = "grok-code-fast"
```

### Example: Cost-Conscious Profile

Use the cheapest model for most tasks, reserve the flagship for research
and planning where deep reasoning matters.

```toml
[[profiles]]
name = "budget"

[profiles.tasks.default]
provider = "xai"
model = "grok-code-fast"

[profiles.tasks.research]
provider = "xai"
model = "grok-4-1-fast-reasoning"
reasoning_effort = "medium"

[profiles.tasks.planning]
provider = "xai"
model = "grok-4-1-fast-reasoning"
reasoning_effort = "high"
```

## Available Models

All models in a profile must be from the live catalog. The current xAI
launch catalog:

| Model ID                          | Display Name                    | Type            |
|-----------------------------------|---------------------------------|-----------------|
| `grok-code-fast`                  | Grok Code Fast                  | Non-Reasoning   |
| `grok-4-1-fast-non-reasoning`     | Grok 4.1 Fast (Non-Reasoning)   | Non-Reasoning   |
| `grok-4-1-fast-reasoning`         | Grok 4.1 Fast                   | Reasoning       |
| `grok-4-fast-reasoning`           | Grok 4 Fast                     | Reasoning       |
| `grok-4.20-non-reasoning`         | Grok 4.20 (Non-Reasoning)       | Non-Reasoning   |
| `grok-4.20-reasoning`             | Grok 4.20                       | Reasoning       |
| `grok-4.20-multi-agent`           | Grok 4.20 (Multi-Agent)         | Multi-Agent     |

Non-reasoning models ignore the `reasoning_effort` field. Reasoning models
default to the model's standard effort when the field is omitted.

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
| Type       | Filter by name or model         |

### Task Editor

| Key        | Action                          |
|------------|---------------------------------|
| `↑` / `↓`  | Navigate task types            |
| `Enter`    | Cycle model for selected task   |
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
- Reserve flagship models (`grok-4.20-reasoning`) for `research` and
  `planning` where deep reasoning pays off.
- Sense mode is independent of profiles. Toggling `/sense` applies to
  whichever model is active for the current task.
- Profile names should be descriptive: `daily-driver`, `budget`,
  `deep-research`, `plan-heavy`.
- Task mappings are optional. A profile with only `default` behaves like
  a single-model preset — you can start simple and add task mappings
  later as your workflow evolves.

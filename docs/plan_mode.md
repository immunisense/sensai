# Plan Mode

Plan Mode is SensAI's spec-driven planning workflow. It breaks complex work
into three sequential phases — Requirements, Design, and Tasks — each
requiring explicit approval before proceeding. Once all phases are approved,
tasks can be executed automatically or one at a time.

## Entering Plan Mode

- **Slash command:** type `/plan` in the TUI input
- **Keyboard shortcut:** `Shift+Tab` toggles between Code Mode and Plan Mode
- **Command palette:** `Ctrl+P` → select "Plan Mode"
- **CLI:** `sensai-cli plan "your goal here"`

The status bar shows "Plan" when Plan Mode is active.

## The Three Phases

### Phase 1: Requirements

The agent generates a detailed requirements document using EARS notation.
Includes functional requirements, non-functional requirements, constraints,
and acceptance criteria.

### Phase 2: Design

Based on the approved requirements, the agent produces a technical design
document with Mermaid diagrams for architecture, data flow, and component
relationships. Includes a TDD plan.

### Phase 3: Tasks

Based on the approved design, the agent creates a granular task breakdown.
Each task references specific files, includes estimated complexity, and
lists dependencies.

## Approval Flow

After each phase completes, a centered dialog opens showing:

- A progress indicator: `Requirements → Design → Tasks` (current phase
  highlighted)
- A scrollable preview of the generated content (↑↓ keys or mouse wheel)
- Two buttons: **Approve** and **Request Changes**

**Approve** saves the phase output to `.sensai/plans/{date}-{name}/` and
advances to the next phase automatically.

**Request Changes** (or pressing Esc) closes the dialog and returns you to
the chat. Continue chatting to refine the output. When satisfied, type
`/approve` to approve the current phase and proceed.

All phase outputs are persisted as markdown files:

```
.sensai/plans/2026-04-02-my-plan/
├── plan.json            # state: current phase, approvals, timestamps
├── 01-requirements.md
├── 02-design.md
└── 03-tasks.md
```

## Task Execution

After all three phases are approved, a "Plan Ready" dialog appears with two
options:

### Run All

Executes every task sequentially. The task progress bar appears in the TUI
showing status icons for each task:

```
 ████████░░░░░░░░ 3/8

  ✓ Set up project structure
  ✓ Implement auth module
  ✓ Add database migrations
  ◐ Build REST endpoints
  ○ Write integration tests
  ○ Add error handling
  ○ Configure CI pipeline
  ○ Write documentation
```

Keyboard shortcuts during execution:
- `r` — retry a failed task
- `s` — skip the current task

### Run Manually

You control which task runs and when. The TUI lists all tasks with their
numbers:

```
Plan tasks ready for manual execution:

  1. Set up project structure
  2. Implement auth module
  3. Add database migrations
  ...

Send #run_task:N to execute a task (e.g. #run_task:1)
```

Type `#run_task:1` to start task 1. When it completes, a dialog appears:

- **Run Task N** — start the next task immediately
- **Chat** — return to chat to review, make changes, or ask questions

You can run tasks in any order and chat between tasks. Already-completed
tasks are tracked and cannot be re-run.

## Slash Commands

| Command    | Description                                    |
|------------|------------------------------------------------|
| `/plan`    | Switch to Plan Mode                            |
| `/approve` | Approve the current plan phase                 |

## CLI Subcommands

| Command                          | Description                          |
|----------------------------------|--------------------------------------|
| `sensai-cli plan "goal"`         | Start a new plan                     |
| `sensai-cli plan list`           | List saved plans                     |
| `sensai-cli plan resume <name>`  | Resume from last approved phase      |
| `sensai-cli plan apply <name>`   | Execute tasks from a completed plan  |
| `sensai-cli plan requirements`   | Run only the requirements phase      |
| `sensai-cli plan design`         | Run only the design phase            |
| `sensai-cli plan tasks`          | Run only the tasks phase             |
| `sensai-cli plan spec`           | Generate a spec from the plan        |
| `sensai-cli plan review`         | Review the plan with a reviewer agent|

## Switching Modes

`Shift+Tab` toggles between Code Mode and Plan Mode. Switching to Code Mode
clears any active plan session. The conversation history is preserved.

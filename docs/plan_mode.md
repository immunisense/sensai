# Plan Mode

Plan Mode is SensAI's spec-driven planning workflow. In the terminal, a
read-only research pass surveys the code, then three gated phases —
Requirements, Design, and Tasks — each go through an automatic check and
your approval. Once the tasks are approved, independent tasks run in
parallel in isolated git worktrees; dependents start when they unblock.

SensAI IDE and SensAI-Agent use the same `.sensai/plans/` folder. **New
Plan** walks Requirements, Design, and Tasks and asks you to approve each
phase. Research, the automatic checks, the task graph, and the coverage
table below are part of `/plan` and `sensai-cli plan`.

## Entering Plan Mode

- **Slash command:** type `/plan` in the TUI
- **Keyboard:** `Shift+Tab` cycles modes. On Sense, Sense Pro, and Sense Ultra the order is Code → Security → Plan → Chat. On other plans Security is skipped. Analyze is `/analyze`. Design is `/design`.
- **Command palette:** `Ctrl+P` → Plan Mode
- **CLI:** `sensai-cli plan "your goal here"`
- **IDE:** Plans view → **New Plan**, or the composer workflow **Plan**

The status bar shows "Plan" when Plan Mode is active. In Plan Mode the agent
can read the codebase but may only write files under `.sensai/plans/`.

## Research

Before requirements, the terminal agent surveys the codebase and writes
`00-research.md`: relevant code with `path:line`, existing patterns,
constraints, risks, and up to five open questions. The requirements phase
answers or asks them.

## The three phases

Each phase has a fixed template. The agent writes the document straight to
its file, so what you approve is what is saved.

### Phase 1: Requirements (`01-requirements.md`)

Requirements with stable IDs in EARS form, for example
`R-1: WHEN the user runs x THE SYSTEM SHALL print y.` Sections: Summary,
Requirements, Acceptance criteria (measurable, per ID), Non-goals, and Open
questions.

### Phase 2: Design (`02-design.md`)

Grounded in the code you will change: Existing code (`path:line`),
Architecture (a flowchart), Key flows (a sequence diagram), Interfaces and
data, Requirement mapping (every `R-N`), Alternatives considered, Risks, and
Test plan. `/wide` can fan out isolated alternatives before design. Enter
locks a pick. Esc keeps the report. `/wide` is a terminal command.

### Phase 3: Tasks (`03-tasks.md`)

A fenced `json` block is the source of truth:

```json
{"tasks":[{"id":1,"title":"Auth store","owns":["internal/auth/**"],"needs":[],
  "verify":"go test ./internal/auth -count=1","tier":"mechanical",
  "covers":["R-1"],"body":"What to change and the done condition."}],
 "branch_verify":"go test ./... -count=1"}
```

The older markdown form (`## Task N: Title` plus `Owns:`, `Needs:`,
`Verify:`, `Tier:`, `Covers:` lines) is still accepted. Numbered steps inside
a task body stay in that task. With `[plan] tdd_enabled = true` (or
`--tdd`), each task starts with a failing test.

## Automatic checks

Before each approval, `sensai-cli plan` and `/plan` check the plan.

| Check | Severity |
| --- | --- |
| No `R-N` IDs; requirement not covered by any task; task covers an unknown ID | Critical |
| Invalid diagram (only `flowchart` / `graph` and `sequenceDiagram`) | Critical |
| Task graph cycle, unknown `needs`, duplicate task numbers, bad tasks JSON | Critical |
| No EARS statements, no acceptance criteria, unanswered open questions | Warning |
| Placeholders (TODO, TBD, ???), no `path:line` references, no diagram | Warning |
| Task without `verify`, `owns`, or `covers`; `owns` path that does not exist | Warning |

Critical findings go back to the agent automatically, up to two rounds.
Whatever remains is shown at the top of the approval dialog. Approving with
a critical finding asks for confirmation. Run the same check any time with
`sensai-cli plan check <name>`.

## Approval dialog

The terminal dialog:

| Key | Action |
| --- | --- |
| `a` / Enter | Approve |
| `r` / Esc | Request changes. Your next message revises the file and the dialog reopens |
| `e` | Edit the document in `$EDITOR`, then re-check |
| `d` | Diff against the previous revision |
| `b` | Go back a phase. Later phases are regenerated |
| `↑↓` `PgUp/PgDn` `g/G` | Scroll |

Diagrams are drawn in the terminal. The saved files keep the diagram source.
Revisions are kept as `0N-<phase>.rK.md`.

In Sense Chat, each phase ends in **Approve** and **Request changes**.

`/approve` approves the current phase from the terminal.

## Generated files

After the tasks phase in the terminal:

- `04-graph.md`: task graph, waves (which tasks run together), and the critical path. Built from `needs`.
- `05-trace.md`: requirement × task coverage. Gaps show as `MISSING`.

## Execution

The Plan Ready dialog shows the task graph, waves, critical path, a credit
estimate (including one retry per task and Branch-Verify), any remaining
task warnings, and every shell command the plan will run.

- **Run All.** Choosing it approves the listed commands. Ready tasks with disjoint `owns` run in parallel, each in its own git worktree. A task gets its covered requirements and the relevant design excerpt, may only change files matching its `owns`, runs its `verify` inside the worktree before merging, and merges into HEAD one at a time. A conflicting merge is aborted. A failed task is retried once with the error. Branch-Verify runs once after every task passes, then a spec-drift check against the requirements. Outside a git repo, tasks run one at a time.
- **Manual.** Send `#run_task:N`. Tasks whose `needs` are not done are refused. `#run_task:0` re-runs Branch-Verify. Commands that Run All did not approve still ask for permission.

While tasks run, the progress panel shows each task's state, what it is
waiting on, and elapsed time. Press `r` to retry the first failed task, or
`s` to skip (and cancel) the first unfinished task. Skipping unblocks
dependents. Switching modes cancels running tasks.

Task progress is saved in `plan.json`. After a restart, run
`/plan resume [name]` or `sensai-cli plan resume <name>`.

## Plan storage

```
.sensai/plans/2026-09-01-add-rate-limiting/
├── plan.json
├── 00-research.md
├── 01-requirements.md
├── 02-design.md
├── 03-tasks.md
├── 04-graph.md
└── 05-trace.md
```

A second plan with the same name on the same day gets a `-2` suffix.
`00-research.md`, `04-graph.md`, and `05-trace.md` are written by the
terminal flow.

## CLI

| Command | Description |
| --- | --- |
| `sensai-cli plan "<goal>"` | Research and all phases. Approve each one in the terminal |
| `sensai-cli plan --yes "<goal>"` | Accept every phase without asking |
| `sensai-cli plan --tdd "<goal>"` | Test-first tasks |
| `sensai-cli plan list` | List plans and their phase |
| `sensai-cli plan resume <name>` | Continue at the first unapproved phase |
| `sensai-cli plan check <name>` | Check a saved plan. Exits with an error on a critical finding |
| `sensai-cli plan apply <name>` | Show the plan's commands, confirm, then execute |
| `sensai-cli plan review [name]` | Review each phase from a fresh context |
| `sensai-cli plan spec` | Generate a spec from the latest plan |

`<name>` is the full directory name or the part after the date. An ambiguous
name lists the matches. At the terminal prompt, type `a` to approve, any
other text to revise, or `q` to stop and resume later.

## Configuration

```toml
[plan]
tdd_enabled = true
```

# Skills

A skill is a `SKILL.md` the agent loads when you use it. Run one by typing `/name` in the terminal or in Sense Chat. **SensAI: Insert Skill** inserts the name into the composer. The Cockpit lists built-in skills and yours, and can disable one. A disabled skill is hidden from `/name`.

## Where they live

| Scope | Path |
| --- | --- |
| This project | `.sensai/skills/<name>/SKILL.md` |
| Global on Windows | `%LOCALAPPDATA%\sensai\skills\<name>\SKILL.md` |
| Global on macOS and Linux | `~/.config/sensai/skills/<name>/SKILL.md` |

Each skill is a folder with `SKILL.md`. The frontmatter needs a `name` and a `description`. Cockpit **+** on Workspace writes the project folder. **+** on Global writes `~/.sensai/skills`, which the Cockpit lists. The terminal runs project skills and the global folders in the table.

`/skillify` interviews you and writes a skill from the procedure, not a dump of the transcript. `/skillify [name]` takes the name on the command.

## Built in

| Command | What it does |
| --- | --- |
| `/verify` | Run the project's tests or the app, and say what could not be run |
| `/simplify` | Review the current diff and fix the issues worth fixing |
| `/skillify` | Save a repeatable process as `SKILL.md` |
| `/remember` | Propose notes for `SENSAI.md`, personal rules, or agent memory. Nothing is written until you accept each item |
| `/loop` | Repeat a prompt on an interval. Example: `/loop 5m check the deploy`. It also runs once now. Later runs fire while the session is idle, at most 8 times, and stop after 7 days |
| `/batch` | Plan a wide mechanical change, then run it as separate worktree agents |
| `/debug` | Read the SensAI log and explain a problem in this session |
| `/designer-direction` | Pick a direction for a web surface in this project |
| `/designer-critique` | Critique one surface against the project's design |
| `/designer-artifact` | Prepare a design brief from the current direction |
| `/designer-motion` | A motion storyboard, when you ask for one |
| `/sense-config` | Edit SensAI's TOML config with you |
| `/sense-hook` | Create or change a hook. See [hooks.md](hooks.md) |
| `/sensai-api` | Call SensAI from a project. Requests go through the proxy. You do not get a raw provider key |

`/keybindings` prints the terminal shortcut list. The longer keybindings help is a terminal-only skill.

Custom agents are a different file. See [custom_agents.md](custom_agents.md).

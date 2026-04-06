# Custom Agents

SensAI supports user-defined agents — specialized AI assistants you can
create for specific tasks and workflows. Custom agents run as sub-agents
alongside the main coder agent, with up to 4 running simultaneously.

## Quick Start

```bash
# AI-powered creation (paid tiers) — just describe what you want
sensai-cli agents create code-reviewer
# → "What should this agent do?" → "reviews Go code for bugs and performance"
# → SensAI generates a full agent definition automatically

# Manual creation (all tiers)
sensai-cli agents create code-reviewer --manual

# List all agents
sensai-cli agents list
```

Or create the file by hand:

```bash
mkdir -p .sensai/agents
cat > .sensai/agents/code-reviewer.md << 'EOF'
---
description: Reviews code for quality and best practices
---

You are a code reviewer. Focus on:
- Security vulnerabilities
- Performance issues
- Code clarity and maintainability
- Missing error handling

Provide constructive, specific feedback with file:line references.
EOF
```

Restart SensAI and the agent is ready to use.

## How It Works

Custom agents are defined as markdown files with a small YAML frontmatter
header. The filename becomes the agent ID, the frontmatter holds metadata,
and the markdown body becomes the agent's system prompt — its personality
and instructions.

When the main coder agent needs help with a specialized task, it can invoke
your custom agent by name through the `agent` tool. You can also ask the
coder to use a specific agent: "use the code-reviewer agent to check my
changes."

Each agent invocation:
- Gets its own session (visible in session history)
- Runs with the active model (inherits your current model selection)
- Has access to read-only tools by default (search, read, list)
- Returns a single result to the parent agent
- Costs are tracked and accumulated to the parent session

## Creating Agents

### AI-Powered Creation (Paid Tiers)

On Pro, Ultra, Sense, or Sense Pro tiers, `sensai-cli agents create` uses
`grok-4-1-fast-reasoning` to generate a complete agent definition from your
description. You provide a name and a short description of what the agent
should do — SensAI generates the full system prompt with detailed
instructions, workflow steps, and output formatting.

```bash
$ sensai-cli agents create security-auditor
What should this agent do? audits code for security vulnerabilities and OWASP top 10 issues
Generating agent with grok-4-1-fast-reasoning...
  Tokens used: 487 (credits will be deducted)

✓ Created agent "security-auditor" → .sensai/agents/security-auditor.md
  Restart SensAI to activate. Edit the file to refine.
```

This costs credits like any model call — the generation runs through the
same proxy, billing, and credit accounting as regular conversations.

### Manual Creation (All Tiers)

Free tier users, or anyone who prefers to write their own prompts, can use
`--manual` to skip AI generation:

```bash
sensai-cli agents create code-reviewer --manual
```

This asks for a description and an optional system prompt, then writes a
minimal agent file you can edit.

### Creating by Hand

You can also create agent files directly. Drop a `.md` file in
`.sensai/agents/` (project) or `~/.sensai/agents/` (global):

```markdown
---
description: Reviews code for quality and best practices
---

You are a code reviewer. Focus on security, performance, and
maintainability. Provide constructive feedback with file:line references.
```

## Agent File Format

An agent file has two parts: YAML frontmatter and a markdown body.

### Minimal (just a description)

```markdown
---
description: Explores codebases and answers architecture questions
---

You are a codebase explorer. When asked about code, search thoroughly
before answering. Always include file paths and line numbers.
```

### Full (all options)

```markdown
---
name: Security Auditor
description: Performs security audits and identifies vulnerabilities
mode: subagent
---

You are a security expert. Analyze code for:
- Input validation vulnerabilities
- Authentication and authorization flaws
- Data exposure risks
- Dependency vulnerabilities
- Hardcoded secrets or credentials

Report findings with severity (critical/high/medium/low), affected
file:line, and a recommended fix.
```

### Frontmatter Fields

| Field         | Required | Default        | Description                              |
|---------------|----------|----------------|------------------------------------------|
| `description` | yes      | —              | Short description of what the agent does |
| `name`        | no       | from filename  | Display name (auto-generated if omitted) |
| `mode`        | no       | `subagent`     | `subagent` or `primary`                  |
| `disabled`    | no       | `false`        | Set to `true` to disable without deleting |

The markdown body after the closing `---` is the agent's system prompt.
This is where you define the agent's personality, expertise, and
instructions. Write it like you're briefing a specialist.

## File Locations

| Location | Scope | Priority |
|----------|-------|----------|
| `~/.sensai/agents/*.md` | Global — available in all projects | Lower |
| `.sensai/agents/*.md` | Per-project — only this project | Higher (overrides global) |

If both directories contain an agent with the same filename, the
per-project version wins.

## CLI Commands

### `sensai-cli agents list`

List all configured agents with their mode, source, and status.

```
$ sensai-cli agents list
ID               MODE       SOURCE     STATUS   DESCRIPTION
────────────────────────────────────────────────────────────────────────────────
coder            primary    builtin    active   An agent that helps with exec...
task             subagent   builtin    active   An agent that helps with sear...
code-reviewer    subagent   markdown   active   Reviews code for quality and ...
explorer         subagent   markdown   active   Explores codebases and answer...
```

### `sensai-cli agents create`

Create a new agent interactively.

```bash
# AI-powered (paid tiers — uses grok-4-1-fast-reasoning, costs credits)
sensai-cli agents create code-reviewer

# Manual (all tiers — you write the prompt)
sensai-cli agents create code-reviewer --manual

# Save to global ~/.sensai/agents/ instead of project
sensai-cli agents create code-reviewer --global
```

## Using Agents

### Direct invocation with `#agent:`

Type `#agent:<name> <prompt>` in the chat input to invoke a specific
agent directly:

```
#agent:code-reviewer review the changes in src/auth/
#agent:explorer where is the database connection configured?
#agent:security-auditor check the new API endpoints
```

The coder agent receives the prompt and delegates to the named agent
automatically.

### From the `/agents` dialog

Type `/agents` to open the agents management dialog. Browse, filter,
and select agents. Selecting an agent shows usage instructions with the
`#agent:<name>` syntax.

### From the coder agent

The main coder agent can invoke custom agents automatically when it
determines a specialist would help. You can also direct it:

```
> use the code-reviewer agent to review the changes in src/auth/
> ask the explorer agent where the database connection is configured
> have the security-auditor check the new API endpoints
```

### Concurrent execution

Up to 4 sub-agents can run simultaneously. The coder agent can launch
multiple agents in parallel for independent tasks:

```
> search for all TODO comments (use explorer), review the auth module
  (use code-reviewer), and check for security issues (use security-auditor)
```

### Built-in agents

SensAI ships with two built-in agents:

| Agent   | Mode     | Description |
|---------|----------|-------------|
| `coder` | primary  | The main coding agent with full tool access |
| `task`  | subagent | Read-only agent for searching and context gathering (hidden, used internally) |

Custom agents extend this set. They appear in the agent tool's available
agents list and can be invoked by name.

## Examples

### Code Reviewer

`.sensai/agents/code-reviewer.md`:

```markdown
---
description: Reviews code changes for quality, bugs, and best practices
---

You are a senior code reviewer. When reviewing code:

1. Check for bugs, edge cases, and error handling gaps
2. Look for security issues (injection, auth bypass, data exposure)
3. Evaluate performance (unnecessary allocations, N+1 queries, blocking calls)
4. Assess readability and maintainability
5. Verify tests cover the changes

Format your review as:
- **Issue** (severity): description → file:line
- **Suggestion**: what to change and why

Be constructive. Praise good patterns. Don't nitpick style if it matches
the existing codebase.
```

### Architecture Explorer

`~/.sensai/agents/explorer.md` (global — works in any project):

```markdown
---
description: Explores codebases and explains architecture decisions
---

You are a codebase archaeologist. When asked about code:

1. Search broadly first, then narrow down
2. Trace call chains and data flow
3. Identify patterns, conventions, and architectural decisions
4. Always cite specific files and line numbers
5. Explain *why* things are structured the way they are, not just *what*

Present findings as a clear narrative, not a list of files.
```

## Tips

- Keep agent prompts focused. A specialist that does one thing well is
  better than a generalist.
- The markdown body is the most important part. Spend time crafting clear
  instructions — or let AI generate them for you.
- Sub-agents have read-only tool access by default. They can search, read,
  and list files but cannot edit or run commands.
- Use global agents (`~/.sensai/agents/`) for agents you want everywhere.
  Use project agents (`.sensai/agents/`) for project-specific specialists.
- Agent names should be kebab-case: `code-reviewer`, `test-writer`,
  `security-auditor`. The display name is auto-generated from the filename.
- AI-generated agents are a starting point. Edit the file to refine the
  prompt for your specific needs.

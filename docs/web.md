# SensAI web

Public app: [https://sensai.immunisense.com/](https://sensai.immunisense.com/)

Same account, catalog, and credit ledger as `sensai-cli`, SensAI IDE, and SensAI-Agent. Session is an HttpOnly `sensai_session` cookie. JWT is never stored in `localStorage`.

Hosted docs on the same origin: [https://sensai.immunisense.com/docs](https://sensai.immunisense.com/docs). Plans: [https://sensai.immunisense.com/pricing](https://sensai.immunisense.com/pricing).

## What it is for

| Page | Purpose |
|------|---------|
| Landing | Product pitch and install commands |
| Sign in / register | GitHub, Google, or password. TOTP when enrolled. Visible signup campaigns show on register |
| Chat | Tool-free conversation through the proxy |
| Account | Credits (tier / bonus / top-up, two decimal places), subscribe, top-up, billing portal |
| Pricing | Six plans, catalog, credit buckets, Sense Mode vs Security Mode |
| Docs | Install, modes, CLI, MCP/LSP, credits, proxy |
| Install | `curl` and Windows installer endpoints |

Code Mode tools, diffs, worktrees, Plan Mode, and Security Mode stay on the TUI and IDE. Web chat is Chat Mode: conversation only.

## Surfaces

| Surface | Where | What it is for |
|---------|--------|----------------|
| **sensai-cli** | Terminal TUI | Code Mode, tools, diffs, Plan Mode, Security Mode |
| **SensAI IDE** | Windows workbench | Editor, Sense Chat, cockpit, plans. [Guide](ide.md) |
| **SensAI-Agent** | VS Code / Cursor / Windsurf | The same chat in your editor. [Guide](ide.md) |
| **Web** | this host | Chat Mode, credits, subscribe, docs |

## Modes

| Mode | How | Behaviour |
|------|-----|-----------|
| **Code** | `sensai-cli` or `/code` | Default. Full tools, edits, shell, LSP, MCP |
| **Plan** | `sensai-cli plan` or `/plan` | Research, then requirements → design → tasks. [Guide](plan_mode.md) |
| **Chat** | `/chat` or [web chat](https://sensai.immunisense.com/chat) | Conversation only. No tools |
| **Analyze** | `sensai-cli analyze` or `/analyze` | Read-only tools in a throwaway git worktree |
| **Design** | `/design` | Architecture / DESIGN.md. No shell |
| **Security** | `/security` | Sense Protocol hunt. Included with Sense, Sense Pro, and Sense Ultra. |

`Shift+Tab` cycles Code → Plan → Chat. On Sense, Sense Pro, and Sense Ultra it is Code → Security → Plan → Chat. Analyze and Design stay on `/analyze` and `/design`.

**Sense Mode** (`/sense`) is full-context pricing on capable models. It is not a subscription tier and it is not Security Mode.

## Credits

Credits show two decimal places. Buckets are spent in order: **tier → bonus → top-up**. Only the tier bucket resets each cycle. Top-ups never expire. When a turn cannot start, SensAI opens subscribe or top-up.

1 plan credit = $0.04. Top-up credits are $0.05. A $10 top-up grants 200 credits. Local bash / edit / grep are $0 extra.

| Tier | Price | Monthly credits | Access |
|------|-------|-----------------|--------|
| Free | $0 | 50 | Grok Build + Gemma 4. Grok Build is 256K; prompts over 200K are 2× |
| Pro | $20 | 450 | All models + all reasoning + Sense context |
| Ultra | $40 | 950 | Same catalog, higher allocation |
| Sense | $100 | 2,400 | Security Mode |
| Sense Pro | $200 | 5,000 | Security Mode + priority routing |
| Sense Ultra | $400 | 10,000 | Highest allocation + Security Mode + priority routing |

Every plan includes the four surfaces, Code / Plan / Chat / Analyze / Design, MCP, LSP, custom agents, checkpoints, and the secrets scanner.

Manage billing in [Account](https://sensai.immunisense.com/account) or:

```bash
sensai-cli credits
sensai-cli topup
sensai-cli billing portal
```

## Issues

File web bugs with the [`web`](https://github.com/immunisense/sensai/issues?q=label%3Aweb) label: [Web bug template](https://github.com/immunisense/sensai/issues/new?template=bug-web.yml).

Include the URL, browser, and whether you were signed in. Do not paste cookies, JWTs, or backup codes.

Vulnerabilities: **security@immunisense.com** — never a public issue.

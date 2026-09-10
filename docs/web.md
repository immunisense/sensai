# SensAI web

Public app: [https://sensai.immunisense.com/](https://sensai.immunisense.com/)

Same account, catalog, and credit ledger as `sensai-cli`, SensAI IDE, and SensAI-Agent. Session is an HttpOnly `sensai_session` cookie. JWT is never stored in `localStorage`.

## What it is for

| Page | Purpose |
|------|---------|
| Landing | Product pitch and install commands |
| Sign in / register | GitHub, Google, or password. TOTP when enrolled |
| Chat | Tool-free conversation through the proxy |
| Account | Credits (tier / bonus / top-up, two decimal places), subscribe, top-up, billing portal |
| Docs | Short product docs on the same host |
| Install | `curl` and Windows installer endpoints |

Code Mode tools, diffs, worktrees, Plan Mode, and Security Mode stay on the TUI and IDE. Web chat is Chat Mode: conversation only.

## Credits

Display is `float64` with two decimal places. Buckets consume in order: **tier → bonus → top-up**. HTTP 402 when a hold cannot be placed.

Manage billing in the account page or:

```bash
sensai-cli credits
sensai-cli topup
sensai-cli billing portal
```

## Issues

File web bugs with the [`web`](https://github.com/immunisense/sensai/issues?q=label%3Aweb) label: [Web bug template](https://github.com/immunisense/sensai/issues/new?template=bug-web.yml).

Include the URL, browser, and whether you were signed in. Do not paste cookies, JWTs, or backup codes.

Vulnerabilities: **security@immunisense.com** — never a public issue.

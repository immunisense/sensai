# Security Policy

## Reporting a Vulnerability

If you discover a security vulnerability in SensAI, please report it
responsibly. **Do not open a public GitHub issue.**

Email **security@immunisense.com** with:

- A description of the vulnerability and its potential impact.
- Steps to reproduce or a proof of concept.
- The version of `sensai-cli` (`sensai-cli version`), SensAI IDE (Help → About),
  SensAI-Agent (Extensions view), or for web: the page URL and browser.
  Do not send cookies, JWTs, or backup codes.

We aim to acknowledge reports within 48 hours and provide a fix or mitigation
timeline within 7 business days. Reporters will be credited in the release
notes unless they prefer to remain anonymous.

## Supported Versions

| Version | Supported |
|---------|-----------|
| Latest release | Yes |
| Previous minor | Security fixes only |
| Older | No |

We recommend always running the latest release. Use `sensai-cli update` to
stay current.

## Security Architecture

### Proxy-First Design

All LLM traffic routes through the SensAI proxy at
`sensai.immunisense.com`. The client never holds raw provider API keys.
The proxy validates JWTs, enforces tier and credit limits, and forwards
requests to the upstream provider.

### Authentication

- Browser-based OAuth with PKCE.
- JWT + refresh token stored in the OS keyring (macOS Keychain, Windows
  Credential Manager, Linux `libsecret`/`pass`). The web app uses an
  HttpOnly `sensai_session` cookie, never `localStorage`.
- Silent token refresh before expiry; no plaintext credentials on disk.
- Optional authenticator (enroll, verify, and disable through the CLI, the TUI, or the account page). Turning it on shows backup codes once. `sensai-cli auth mfa backup-codes` makes a new set.
- Passkey sign-in from the account page.
- API keys for scripts, created on the account page and shown once. A key can call chat and read usage.
- Changing an authenticator, backup codes, a passkey, an API key, or other signed-in devices asks you to confirm again.
- Headless/WSL/SSH fallback prints the login URL for manual use.

### Pre-Flight Secrets Scanning

Before outgoing context is sent to the proxy, the built-in scanner
(`internal/scanner`) checks for leaked secrets using pattern-based rules.
Configurable modes:

| Mode | Behavior |
|------|----------|
| `warn` (default) | Prompts the user for confirmation before sending. |
| `block` | Aborts the request immediately. |
| `off` | Disables scanning. |

Custom patterns can be added alongside the built-in rules. Matched secrets
are partially redacted in any user-facing output. A secret printed by a
command, a fetch, or a file read is masked before it is stored and sent
back to the model.

### Permission-Aware Tool Execution

Every tool invocation passes through the permission service
(`internal/permission`). Behavior depends on the session mode:

- Tools not in the configured `allowed_tools` list require explicit user
  approval.
- Permissions can be granted per-invocation or persistently for the session.
- Sessions can be set to auto-approve for trusted workflows.
- Analyze Mode restricts the agent to read-only tools in a throwaway worktree.
- Security Mode (Sense Protocol v1.2) is a separate entitled mode: live
  tree, read-only hunt tools, class hunters with independent verifiers,
  no MCP instruction injection, no write/shell, no exploit payloads.
  Included with Sense, Sense Pro, and Sense Ultra. Product findings:
  security@immunisense.com.

### MCP Security Controls

External MCP tool calls are protected by multiple layers:

- **Circuit breakers** — 3 consecutive failures trigger a 30-second cooldown.
- **Rate limiting** — per-server request caps prevent runaway calls.
- **Response scanning** — MCP responses are scanned for leaked secrets before
  surfacing to the agent.
- **Audit logging** — every MCP tool invocation is logged with server name,
  tool name, duration, and success/failure status.
- **Health monitoring** — background pings every 60 seconds; crashed stdio
  servers are auto-restarted.

### Data Handling

- **No telemetry by default.** Audit logs are local unless explicitly
  configured otherwise.
- **No-log mode** can be enabled for sensitive projects.
- **Conversation checkpoints** are stored locally in SQLite; no conversation
  data leaves the machine except through the proxy for LLM inference.
- **`.sensaiignore`** controls which files are excluded from context
  gathering, similar to `.gitignore`.

### Build and Supply Chain

- Binaries are built with `CGO_ENABLED=0` (pure Go, no C dependencies).
- Installers and `sensai-cli update` download the binary from the SensAI
  proxy and verify the SHA-256 checksum before replacing it. GitHub
  Releases are not the download source.
- The curl installer (`deploy/install.sh`) performs platform detection and
  checksum verification before installation.

## Configuration Hardening

```toml
# Enforce strict secrets scanning.
[secrets_scanner]
mode = "block"

# Restrict tool access to read-only operations.
[permissions]
allowed_tools = ["read_file", "list_dir", "search_code", "ast_search"]
# Ask before every search, read, or shell directory outside the workspace.
allow_outside_search = false

# Disable auto-formatting if untrusted formatters are a concern.
[options]
auto_format = false
```

## Responsible Disclosure

We follow coordinated disclosure. If a fix requires a client update, we will
notify affected users through the TUI update prompt and the web app.

# MCP (Model Context Protocol) — User Guide

SensAI integrates with MCP servers to extend the AI agent's capabilities with
external tools, resources, and prompts. MCP servers can provide database
access, web search, browser automation, file operations, and any custom
functionality you need.

## Quick Start

```bash
# Add an MCP server
sensai-cli mcp add my-server npx @modelcontextprotocol/server-github

# Check it's connected
sensai-cli mcp status

# Test connectivity
sensai-cli mcp test my-server
```

## How It Works

SensAI uses a **zero-context discovery** approach. MCP server tool catalogs
are never pre-loaded into the conversation. Instead, the agent discovers and
invokes tools on-demand through lightweight meta-tools:

1. `list_registered_mcps` — See all configured servers and their status
2. `discover_mcp_tools` — List tools on a server (with optional parameter schemas)
3. `discover_mcp_resources` — List resources on a server
4. `call_mcp_tool` — Invoke a specific tool

This keeps context lean and avoids wasting tokens on irrelevant tool
descriptions.

## Configuration

MCP servers are configured in TOML config files:

- **Global**: `~/.sensai/config.toml` (applies everywhere)
- **Per-project**: `.sensai/config.toml` (overrides global for this project)

### Stdio Server (most common)

```toml
[mcp.my-server]
type = "stdio"
command = "npx"
args = ["-y", "@modelcontextprotocol/server-github"]
timeout = 30

[mcp.my-server.env]
GITHUB_PERSONAL_ACCESS_TOKEN = "${GITHUB_TOKEN}"
```

### HTTP Server

```toml
[mcp.my-api]
type = "http"
url = "https://my-mcp-server.example.com/mcp"
timeout = 15

[mcp.my-api.headers]
Authorization = "Bearer ${MY_API_TOKEN}"
```

### SSE Server

```toml
[mcp.my-sse]
type = "sse"
url = "http://localhost:3000/mcp"
```

### Configuration Options

| Field           | Type     | Default | Description                                    |
|-----------------|----------|---------|------------------------------------------------|
| `type`          | string   | stdio   | Transport: `stdio`, `sse`, or `http`           |
| `command`       | string   | —       | Command to run (stdio only)                    |
| `args`          | []string | —       | Command arguments (stdio only)                 |
| `url`           | string   | —       | Server URL (sse/http only)                     |
| `env`           | map      | —       | Environment variables (supports `${VAR}`)      |
| `headers`       | map      | —       | HTTP headers (sse/http, supports `${VAR}`)     |
| `timeout`       | int      | 15      | Connection timeout in seconds                  |
| `disabled`      | bool     | false   | Disable without removing                       |
| `disabled_tools`| []string | —       | Tools to hide from the agent                   |
| `rate_limit`    | int      | 0       | Max requests per minute (0 = unlimited)        |
| `auto_restart`  | bool     | true    | Auto-restart stdio servers on crash             |

## CLI Commands

### `sensai-cli mcp add`

Add a new MCP server.

```bash
# Basic stdio server
sensai-cli mcp add github npx -y @modelcontextprotocol/server-github

# With environment variables
sensai-cli mcp add github npx -y @modelcontextprotocol/server-github \
  --env GITHUB_PERSONAL_ACCESS_TOKEN=ghp_xxx

# HTTP server with auth header
sensai-cli mcp add my-api --type http \
  --url https://api.example.com/mcp \
  --header "Authorization=Bearer token123"

# With timeout and rate limiting
sensai-cli mcp add slow-server npx my-slow-server \
  --timeout 60 --rate-limit 30

# Disable specific tools
sensai-cli mcp add risky-server npx my-server \
  --disabled-tools "delete_all,drop_table"

# Save to workspace config (per-project)
sensai-cli mcp add my-server npx my-server --workspace

# Skip command validation
sensai-cli mcp add my-server custom-binary --no-validate
```

### `sensai-cli mcp list`

List all configured servers with their type and status.

```bash
sensai-cli mcp list
```

### `sensai-cli mcp status`

Show runtime status including connection state, tool counts, uptime, and
errors.

```bash
# All servers
sensai-cli mcp status

# Detailed info for one server
sensai-cli mcp status github
```

Example output:

```
MCP servers (3):

  ✓ github     [connected]  tools=12  uptime=2h30m
  ✓ filesystem [connected]  tools=5   uptime=2h30m
  ✗ postgres   [error]      error="connection refused"
```

### `sensai-cli mcp test`

Test connectivity, list tools, and report latency.

```bash
sensai-cli mcp test github
```

Example output:

```
Testing MCP server "github"...

  Connection: OK (45ms)
  Tools:      12
  Resources:  0
  Prompts:    0

  Available tools:
    - create_issue: Create a new issue in a repository
    - list_issues: List issues in a repository
    ...

  Result: All checks passed
```

### `sensai-cli mcp enable` / `disable`

Toggle servers without editing config files.

```bash
sensai-cli mcp disable postgres
sensai-cli mcp enable postgres
```

### `sensai-cli mcp restart`

Restart a server (closes session, resets circuit breaker, reconnects).

```bash
sensai-cli mcp restart github
```

### `sensai-cli mcp remove`

Remove a server configuration.

```bash
# Remove from global config
sensai-cli mcp remove my-server

# Remove from workspace config
sensai-cli mcp remove my-server --workspace
```

## Resilience & Health

### Circuit Breaker

SensAI automatically protects against failing MCP servers. After 3
consecutive failures, the circuit "opens" and requests are rejected for a
30-second cooldown period. This prevents the agent from wasting time on
broken servers.

The circuit breaker resets automatically when:
- The cooldown expires and a subsequent request succeeds
- You run `sensai-cli mcp restart <name>`

### Background Health Monitoring

A background process pings all connected servers every 60 seconds. If a
server becomes unresponsive:

1. The state updates to `error` in the TUI and CLI
2. For stdio servers with `auto_restart = true` (the default), SensAI
   attempts an automatic restart
3. The circuit breaker tracks failures to avoid hammering a dead server

### Rate Limiting

Set `rate_limit` in your config to cap requests per minute to a server:

```toml
[mcp.my-server]
rate_limit = 60  # Max 60 requests per minute
```

This prevents a runaway agent from overwhelming an MCP server.

## Security

### Response Scanning

MCP tool responses are automatically scanned for leaked secrets (API keys,
tokens, passwords) using the same scanner that checks outgoing context.
Detected secrets are redacted before they enter the conversation.

### Audit Logging

When audit logging is enabled (`audit.enabled = true` in config), all MCP
tool calls are logged with:

- Server name and tool name
- Call duration
- Success/failure status
- Error details (if any)

### Permission Enforcement

Every MCP tool call goes through SensAI's permission system. In supervised
mode, you'll be prompted before each call. Docker MCP management tools
(find, add, remove, config-set, code-mode) are whitelisted for convenience.

### Tool Filtering

Disable specific tools you don't want the agent to use:

```toml
[mcp.my-server]
disabled_tools = ["dangerous_delete", "drop_database"]
```

## Docker MCP

SensAI has first-class Docker MCP integration. If Docker Desktop with MCP
support is installed:

- The TUI offers an "Enable Docker MCP" option
- Docker MCP tools are auto-whitelisted for permissions
- Enable/disable via the command palette or config

```bash
# Check if Docker MCP is available
docker mcp version
```

## Environment Variable Resolution

Config values support `${VAR}` syntax for environment variable
interpolation:

```toml
[mcp.github.env]
GITHUB_PERSONAL_ACCESS_TOKEN = "${GITHUB_TOKEN}"

[mcp.my-api.headers]
Authorization = "Bearer ${API_TOKEN}"
```

Variables are resolved at runtime from your shell environment.

## Troubleshooting

### Server won't connect

1. Run `sensai-cli mcp test <name>` to diagnose
2. Check the command exists: `which npx` or `which docker`
3. Verify environment variables are set
4. Increase timeout: `sensai-cli mcp add <name> ... --timeout 60`
5. Check logs: `sensai-cli logs` for MCP-related errors

### Server keeps disconnecting

1. Check `sensai-cli mcp status <name>` for error details
2. The circuit breaker may be open — wait 30s or run `sensai-cli mcp restart <name>`
3. For stdio servers, ensure the process doesn't exit unexpectedly
4. Set `auto_restart = false` if auto-restart is causing issues:

```toml
[mcp.my-server]
auto_restart = false
```

### Tools not showing up

1. Run `sensai-cli mcp test <name>` to see available tools
2. Check `disabled_tools` in your config
3. The agent discovers tools on-demand — ask it to "list MCP tools"
4. Restart the server: `sensai-cli mcp restart <name>`

### Rate limit errors

Increase or remove the rate limit:

```toml
[mcp.my-server]
rate_limit = 120  # Or 0 for unlimited
```

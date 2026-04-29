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

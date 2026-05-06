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

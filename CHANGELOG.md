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

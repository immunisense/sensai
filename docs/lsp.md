# Language Server Protocol (LSP) Integration

SensAI integrates with Language Server Protocol servers to give the AI agent
semantic understanding of your codebase. Instead of relying on grep and
pattern matching alone, the agent can jump to definitions, inspect types,
rename symbols across your project, apply quick fixes, and format code — all
through the same language servers your editor uses.

## How It Works

When you open a file or the agent reads one, SensAI checks the file extension
against all known LSP servers. If a matching server is installed on your
system, SensAI starts it automatically in the background. No manual setup
required for most languages.

The flow:

1. Agent opens or edits a file (e.g., `main.go`).
2. SensAI detects the `.go` extension and starts `gopls` if it's in your PATH.
3. The LSP server indexes your project and provides diagnostics, type info,
   references, and more.
4. The agent uses LSP tools to navigate and modify code with semantic accuracy.
5. After every edit, SensAI feeds diagnostic results back to the agent so it
   can self-correct if new errors were introduced.

## Supported Languages

SensAI ships with built-in support for 30+ languages. Servers are detected and
started automatically when the required toolchain is installed.

| Language | LSP Server | Requirements |
|----------|-----------|--------------|
| Go | gopls | `go` in PATH |
| Rust | rust-analyzer | `rust-analyzer` in PATH |
| TypeScript / JavaScript | typescript-language-server | `typescript` in project |
| Python | pylsp / pyright | `pylsp` or `pyright` in PATH |
| C / C++ | clangd | `clangd` in PATH |
| Java | jdtls | Java 21+ installed |
| Ruby | ruby-lsp | `ruby` and `gem` in PATH |
| PHP | intelephense | Auto-installs for PHP projects |
| C# | csharp-ls | .NET SDK installed |
| Kotlin | kotlin-language-server | Auto-installs for Kotlin projects |
| Lua | lua-language-server | Auto-installs for Lua projects |
| Bash / Shell | bash-language-server | Auto-installs |
| Svelte | svelte-language-server | Auto-installs for Svelte projects |
| Vue | vue-language-server | Auto-installs for Vue projects |
| Astro | astro-ls | Auto-installs for Astro projects |
| Terraform | terraform-ls | Auto-installs from GitHub releases |
| YAML | yaml-language-server | Auto-installs |
| Zig | zls | `zig` in PATH |
| Elixir | elixir-ls | `elixir` in PATH |
| Haskell | hls | `haskell-language-server-wrapper` in PATH |
| OCaml | ocaml-lsp | `ocamllsp` in PATH |
| Gleam | gleam | `gleam` in PATH |
| Dart | dart | `dart` in PATH |
| Nix | nixd | `nixd` in PATH |
| Swift | sourcekit-lsp | Xcode / Swift installed |
| Clojure | clojure-lsp | `clojure-lsp` in PATH |
| F# | fsautocomplete | .NET SDK installed |
| Julia | julials | `julia` + LanguageServer.jl |
| Prisma | prisma-language-server | `prisma` in project |

If a server is not in your PATH, SensAI silently skips it. No errors, no
noise. You can always add it later.

## Agent Tools

The agent has access to 9 LSP-powered tools. These are used automatically
during coding tasks — you don't need to invoke them manually.

### lsp_diagnostics

Get errors, warnings, and hints for a file or the entire project.

The agent calls this after edits to check for problems. SensAI also
automatically includes diagnostic feedback after every `patch_file`,
`patch_files`, and `write_file` operation, with a delta indicator that warns
the agent when new errors were introduced.

### lsp_references

Find all references to a symbol across the project. Semantic-aware — only
returns real code references, not comments or unrelated string matches.

### lsp_definition

Jump to where a function, type, or variable is defined. The agent uses this
to navigate to implementations instead of grepping through files.

### lsp_hover

Get type signatures, documentation, and parameter info for a symbol without
reading the full source file. Useful for understanding APIs from external
packages.

### lsp_symbols

Two modes:
- **Document symbols**: list all functions, types, variables, and methods in
  a file with their hierarchy.
- **Workspace symbols**: search for symbols by name across the entire project.

### lsp_rename

Rename a symbol across the entire project in one operation. Semantic-aware —
only renames actual code references, not comments or strings. Much safer than
find-and-replace.

### lsp_code_actions

List and apply quick fixes and refactorings from the LSP server. This
includes things like:
- Adding missing imports
- Fixing typos in symbol names
- Extracting functions or variables
- Organizing imports
- Applying suggested fixes for diagnostics

The agent can list available actions for a line range, then apply a specific
one by index.

### lsp_format

Format a file using the project's LSP formatter. Respects your project's
formatting configuration (e.g., `.prettierrc`, `rustfmt.toml`, `gofmt`
defaults). The agent uses this after writing code to ensure consistent style.

### lsp_restart

Restart one or all LSP servers. Useful when a server becomes unresponsive or
after major project structure changes.

## Configuration

Most users don't need to configure anything. SensAI auto-detects LSP servers
from your PATH and project structure.

### Basic setup

Add to `~/.sensai/config.toml` (global) or `.sensai/config.toml` (project):

```toml
[lsp.go]
command = "gopls"

[lsp.typescript]
command = "typescript-language-server"
args = ["--stdio"]
```

### Full configuration options

Each LSP server supports these fields:

| Field | Type | Description |
|-------|------|-------------|
| `command` | string | The executable to run |
| `args` | string[] | Command-line arguments |
| `env` | map | Environment variables |
| `filetypes` | string[] | File extensions this server handles (e.g., `[".go", ".mod"]`) |
| `root_markers` | string[] | Files that indicate the project root (e.g., `["go.mod"]`) |
| `init_options` | map | Initialization options sent to the server |
| `options` | map | Server-specific settings |
| `timeout` | int | Initialization timeout in seconds (default: 30) |
| `disabled` | bool | Set to `true` to disable this server |

### Example: full TypeScript configuration

```toml
[lsp.typescript]
command = "typescript-language-server"
args = ["--stdio"]
filetypes = [".ts", ".tsx", ".js", ".jsx"]
root_markers = ["tsconfig.json", "package.json"]
timeout = 30

[lsp.typescript.env]
NODE_OPTIONS = "--max-old-space-size=4096"

[lsp.typescript.init_options]
preferences.importModuleSpecifierPreference = "relative"

[lsp.typescript.options]
typescript.inlayHints.parameterNames.enabled = "all"
```

### Example: custom LSP server

```toml
[lsp.my-language]
command = "/usr/local/bin/my-lsp"
args = ["--stdio", "--log-level=warn"]
filetypes = [".mylang"]
root_markers = ["mylang.config"]
```

### Disabling LSP

Disable a specific server:

```toml
[lsp.eslint]
disabled = true
```

Disable all LSP servers globally:

```toml
[options]
auto_lsp = false
```

### Environment variables

LSP server commands support variable expansion. For example, if your server
needs a path that varies by machine:

```toml
[lsp.custom.env]
MY_SDK_PATH = "$HOME/sdk/latest"
```

## CLI Commands

### List servers

```bash
sensai-cli lsp list
```

Shows all configured LSP servers and any currently running servers with their
state.

Example output:

```
Configured LSP servers:
  go        command=gopls       status=enabled
  typescript command=typescript-language-server status=enabled

Running LSP servers:
  gopls     state=ready  diagnostics=3
```

### Server status

```bash
sensai-cli lsp status
```

Shows detailed health information for running LSP servers including diagnostic
counts broken down by severity.

Example output:

```
gopls:
  State: ready
  Diagnostics: 5
  Errors: 2  Warnings: 3  Info: 0  Hints: 0
  Connected: 14:32:07

typescript-language-server:
  State: ready
  Diagnostics: 1
  Errors: 0  Warnings: 1  Info: 0  Hints: 0
  Connected: 14:32:09
```

## How the Agent Uses LSP

The agent follows a search strategy that combines fast lexical search with
semantic LSP tools:

1. **`@` context** — instant, zero-cost file and symbol references
2. **`search_code`** — ripgrep-backed lexical search for patterns and strings
3. **LSP tools** — `lsp_definition`, `lsp_references`, `lsp_hover`,
   `lsp_symbols` for semantic navigation when the symbol is known
4. **`read_file`** — targeted file reads when full content is needed
5. **Deeper reasoning** — only when the above steps are insufficient

For refactoring tasks, the agent prefers `lsp_rename` over manual
find-and-replace, and `lsp_code_actions` over hand-written fixes for common
patterns like missing imports.

After every file edit, SensAI automatically:
1. Notifies the LSP server about the change.
2. Waits for updated diagnostics.
3. Includes the diagnostic summary in the tool response.
4. Warns the agent if the edit introduced new errors.

This feedback loop means the agent catches and fixes mistakes in the same
turn instead of requiring a separate diagnostics check.

## Troubleshooting

### LSP server not starting

Check that the server command is in your PATH:

```bash
which gopls          # macOS/Linux
where.exe gopls      # Windows
```

If the command is installed but SensAI isn't starting it, add it explicitly
to your config:

```toml
[lsp.go]
command = "gopls"
```

Alternatively, enable auto-install to let SensAI install missing servers
automatically:

```toml
[options]
auto_install_lsp = true
```

SensAI can auto-install servers via npm, pip, `go install`, and cargo for
the most common languages. Servers that require manual installation (like
rust-analyzer or lua-language-server) will log a message with instructions.

## Auto-Install

When `auto_install_lsp = true` in your config, SensAI automatically installs
missing LSP servers the first time a matching file is opened.

Supported install methods:

| Method | Servers |
|--------|---------|
| npm | bash-language-server, typescript-language-server, pyright, yaml-language-server, svelte, vue, astro, intelephense |
| pip | pylsp (python-lsp-server) |
| go install | gopls |
| cargo | (future) |
| manual | rust-analyzer, lua-language-server, kotlin-language-server |

Enable it:

```toml
[options]
auto_install_lsp = true
```

Auto-install is disabled by default to avoid unexpected global package
installations. When enabled, servers are installed globally using the
appropriate package manager.

## Diagnostic Hooks

SensAI fires the `on_diagnostics_error` hook event after every agent turn if
LSP diagnostics contain errors. This enables automatic fix cycles.

### Example: notify on errors

```toml
[[hooks]]
event = "on_diagnostics_error"

[hooks.action]
notify = "LSP found {{error_count}} error(s) in {{file_paths}}"
```

### Example: run linter auto-fix

```toml
[[hooks]]
event = "on_diagnostics_error"

[hooks.action]
shell = "npm run lint:fix"
```

### Example: trigger the agent to fix errors

```toml
[[hooks]]
event = "on_diagnostics_error"

[hooks.action]
agent = "coder"
```

The hook data includes:
- `{{error_count}}` — total number of error-severity diagnostics
- `{{file_paths}}` — comma-separated list of files with errors
- `{{session_id}}` — the current session ID

## Multi-Workspace (Monorepo Support)

For monorepos where different subdirectories have their own project roots
(e.g., separate `tsconfig.json` or `go.mod` files), configure workspace
paths on the LSP server:

```toml
[lsp.typescript]
command = "typescript-language-server"
args = ["--stdio"]
workspaces = ["packages/frontend", "packages/backend", "packages/shared"]
```

Each workspace path gets its own LSP client instance scoped to that
subdirectory. This means:

- Each subdirectory has its own diagnostics, symbols, and references.
- Root markers are checked relative to the workspace root, not the
  monorepo root.
- The agent automatically routes files to the correct workspace client.

Workspace paths are relative to the project root (where `.sensai/` lives).

### LSP server crashing or unresponsive

The agent can restart servers using the `lsp_restart` tool. From the CLI:

```bash
sensai-cli lsp status
```

If a server shows `state=error`, check the logs:

```bash
sensai-cli logs
```

Or run with debug LSP logging:

```toml
[options]
debug_lsp = true
```

### Diagnostics not updating

If diagnostics seem stale after edits, the LSP server may need a restart.
The agent will do this automatically if it detects issues, or you can
configure a longer timeout:

```toml
[lsp.typescript]
timeout = 60
```

### Too many diagnostics from linters

If you have both a language server and a linter (e.g., `typescript` +
`eslint`), you may see duplicate diagnostics. Disable the one you don't
want:

```toml
[lsp.eslint]
disabled = true
```

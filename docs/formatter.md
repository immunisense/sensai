# Auto-Formatter

SensAI automatically formats files after every agent write and edit. This
ensures code produced by the agent follows your project's formatting rules
without the agent needing to call a formatter explicitly.

## How It Works

When the agent writes or edits a file (`write_file`, `patch_file`,
`patch_files`), SensAI runs the formatter before collecting LSP diagnostics.
The fallback chain is:

1. **Custom formatter** — if you configured one for the file extension, it
   runs first.
2. **LSP formatter** — if an LSP server is running and supports formatting
   for the file type, SensAI uses it.
3. **Built-in CLI formatter** — SensAI auto-detects formatters installed on
   your machine based on file extension and project config.
4. **No-op** — if nothing matches, the file is left as-is.

The agent sees a brief note in the tool response when formatting occurs:
`✓ Auto-formatted (gofumpt)`.

## Built-in Formatters

SensAI ships with detection logic for 25+ formatters. Each one is activated
only when the binary is available and any required config files are present.

| Formatter | Extensions | Requirements |
|-----------|-----------|--------------|
| gofumpt | .go | `gofumpt` in PATH |
| goimports | .go | `goimports` in PATH |
| gofmt | .go | `gofmt` in PATH |
| cargo fmt | .rs | `cargo` in PATH + `Cargo.toml` |
| rustfmt | .rs | `rustfmt` in PATH |
| ruff | .py, .pyi | `ruff` in PATH + config file |
| black | .py, .pyi | `black` in PATH |
| biome | .js, .jsx, .ts, .tsx | `biome.json` or `biome.jsonc` |
| prettier | .js, .jsx, .ts, .tsx, .html, .css, .md, .json, .yaml | `prettier` in package.json |
| rubocop | .rb, .rake, .gemspec, .ru | `rubocop` in PATH |
| shfmt | .sh, .bash | `shfmt` in PATH |
| dart format | .dart | `dart` in PATH |
| zig fmt | .zig, .zon | `zig` in PATH |
| terraform fmt | .tf, .tfvars | `terraform` in PATH |
| nixfmt | .nix | `nixfmt` in PATH |
| mix format | .ex, .exs, .eex, .heex, .leex | `mix` in PATH + `mix.exs` |
| ktlint | .kt, .kts | `ktlint` in PATH |
| clang-format | .c, .cpp, .h, .hpp, .cc, .cxx | `clang-format` in PATH + `.clang-format` |
| ormolu | .hs | `ormolu` in PATH |
| gleam format | .gleam | `gleam` in PATH |
| dfmt | .d | `dfmt` in PATH |
| air | .R | `air` in PATH |
| ocamlformat | .ml, .mli | `ocamlformat` in PATH + `.ocamlformat` |
| cljfmt | .clj, .cljs, .cljc, .edn | `cljfmt` in PATH |
| pint | .php | `laravel/pint` in composer.json |
| htmlbeautifier | .erb | `htmlbeautifier` in PATH |

For Go files, the priority is gofumpt → goimports → gofmt. The first one
found wins.

## TUI Configuration

Type `/formatter` in the TUI to open the formatter configuration dialog.
The dialog shows:

- **Auto-Format on Write** — global toggle for the entire feature
- **Custom formatters** — any formatters you defined in config, with
  enable/disable toggle
- **Detected built-in formatters** — formatters found on your machine

Use arrow keys to navigate, Enter or Space to toggle, and Esc to close.
Changes take effect immediately.

## TOML Configuration

### Disable all formatting

```toml
[options]
auto_format = false
```

### Disable a specific built-in formatter

Override a built-in by name and set `disabled = true`:

```toml
[formatter.prettier]
disabled = true
```

### Add a custom formatter

```toml
[formatter.my-sql-formatter]
command = ["pg_format", "$FILE"]
extensions = [".sql"]

[formatter.my-sql-formatter.environment]
PGFORMAT_SPACES = "4"
```

The `$FILE` placeholder is replaced with the absolute path to the file
being formatted.

### Override a built-in formatter

```toml
[formatter.gofmt]
command = ["gofumpt", "-extra", "-w", "$FILE"]
extensions = [".go"]
```

Custom formatters take priority over built-in detection. If a custom
formatter matches the file extension, the built-in and LSP formatters
are skipped.

### Formatter chaining

Define multiple custom formatters with the same extensions. They run in
map iteration order (non-deterministic in Go), so for strict ordering,
use a wrapper script:

```toml
[formatter.python-chain]
command = ["./scripts/format-python.sh", "$FILE"]
extensions = [".py"]
```

## How It Compares to LSP Formatting

The `lsp_format` agent tool still exists and can be called explicitly by
the agent. Auto-formatting is different:

- **Auto-format** runs silently after every write/edit, before diagnostics.
- **`lsp_format`** is an agent tool the model calls when it decides to.
- Auto-format uses the full fallback chain (custom → LSP → CLI).
- `lsp_format` only uses the LSP server.

Both can coexist. If auto-format already formatted via LSP, the agent
calling `lsp_format` afterward will see "already formatted" with zero edits.

## Troubleshooting

**Formatter not detected?**
- Check that the binary is in your PATH: `which gofumpt`
- For project-scoped formatters (prettier, biome, pint), check that the
  dependency is in your package.json or composer.json.
- For formatters that need config files (clang-format, ruff, ocamlformat),
  check that the config file exists in your project root.

**Formatting breaks my code?**
- Disable the specific formatter: `[formatter.NAME] disabled = true`
- Or disable auto-format globally: `[options] auto_format = false`
- The `lsp_format` tool remains available for manual use.

**Want to see what happened?**
- Run with `--debug` to see formatter selection in the logs.
- The tool response includes `✓ Auto-formatted (NAME)` when formatting
  occurs.

# MEDITECH NPR — VSCode/VSCodium Extension

Syntax highlighting for MEDITECH NPR — the MAGIC imperative code and Report Writer attribute language used in MEDITECH EHR.

Supports three file types:

| Extension | Content |
|---|---|
| `.npr` | MAGIC code — NPR rule bodies, macros |
| `.pha` | PHA rule bodies |
| `.rw` | Report Writer field definitions |

MEDITECH enforces no extension when copying code out of the system; these are conventions for local editing.

## Install

1. Download the `.vsix` file from the [Releases](https://github.com/nocalla/meditech-npr/releases) page
2. In VSCode or VSCodium: open the Extensions panel, click the `...` menu, choose **Install from VSIX…**, and select the downloaded file
3. Open any `.npr`, `.pha`, or `.rw` file — highlighting is applied automatically

## Current Coverage

The grammar highlights:

- **Strings** — double-quoted literals
- **Comments** — lines beginning with `;`
- **Control flow** — `IF{`, `DO{`, `FOR{`
- **Bracket keywords** — `[f rx ok]`, `[evaluate at verification]`, and similar PHA rule declarations
- **Field references** — `@field.name` (data refs) and `@FuncName(` (built-in function calls)
- **Positional operators** — `$(`, `?(`, `#(`
- **Operators** — `^` (assignment/pipe), `&` (AND), `!` (OR), `'=` (not equal), `=` (equal)
- **Numbers** — bare integer literals
- **Punctuation** — `,` `;` `{` `}`

Constructs not yet highlighted include: `\_` (string concatenation), `|0` (default-to-empty), `:` (format qualifier), `/varname` (local variable sigil), `%func` (external function prefix), and the RW attribute keyword vocabulary (`COMMA`, `DAT`, `FNC`, etc.). These are known gaps planned for future releases.

## Language Background

NPR/MAGIC is a MUMPS derivative used inside MEDITECH EHR. Three dialects share a common expression syntax:

- **Report Writer (RW)** — declarative `ATTRIBUTE=value` syntax defining report fields (`COMMA`, `DAT`, `FNC=TOT`, etc.)
- **MAGIC code** — imperative logic embedded in RW attribute values and PHA rule bodies
- **PHA rules** — pharmacy decision rules, headed by bracket declarations (`[f rx ok]`, `[evaluate at verification]`) followed by MAGIC code

**Key quirk — single-letter functions:** the runtime ignores all characters between the first letter of a function name and its opening `(`. So `L(x)`, `LENGTH(x)`, and `LOCH(x)` are identical at runtime. Highlighting individual built-in functions by name is impractical for this reason; the grammar highlights the `@UpperCase(` call form as a function reference instead.

## Development

```bash
# Install dependencies
npm install

# Launch Extension Development Host (live grammar testing)
# Press F5 in VSCodium/VSCode with this folder open, then open an .npr file

# Reload grammar after edits
# Press Ctrl+R in the Extension Development Host window

# Lint
npm run lint

# Package as .vsix
npm run package
```

The grammar lives entirely in `syntaxes/meditech-npr.tmLanguage.json` — a TextMate grammar with a flat `patterns` array delegating to named rules in `repository`. Match order matters: more specific patterns must appear before more general ones. See `examples/` for sample files covering all supported constructs.

Releases are cut by pushing a `v*` tag to `main`; GitHub Actions runs lint, then builds and attaches the `.vsix` to a GitHub Release automatically.

Contributions welcome — open an issue or PR.

## Roadmap

### Grammar gaps (near term)

- `\_` concatenation, `|0` default-to-empty, `:` format qualifier
- `/varname` local variable sigil, `%func` external function prefix
- RW attribute keyword vocabulary

### Language Server (longer term)

Syntax highlighting is purely static regex. Richer editor features require a Language Server (LSP):

| Feature | Notes |
|---|---|
| Completion | `[f ...]` field names, `@` data refs, control flow keywords |
| Hover | Descriptions for known fields and functions |
| Diagnostics | Mismatched `{}`/`}`, unknown field references |
| Go to definition | Field ref → definition (requires workspace indexing) |
| Document symbols | Outline view for long rule bodies |

Build order once the grammar is stable: scaffold LSP server with `vscode-languageserver-node` → hand-written recursive-descent parser for rule bodies → diagnostics → completion from a field dictionary → hover.

## Licensing and Disclaimer

This extension is an **unofficial, independent community tool** and is not affiliated with, endorsed by, or sanctioned by MEDITECH. "MEDITECH", "NPR", "MAGIC", and related terms are trademarks of Medical Information Technology, Inc.

The grammar and tooling were developed from publicly available documentation, community resources, and code samples. No proprietary MEDITECH source code, data, or internal documentation was used.

Provided as-is for productivity purposes under the [MIT License](LICENSE). Users are responsible for ensuring their use complies with any applicable MEDITECH licence agreements at their site.

# meditech-npr-syntax

VSCodium syntax highlighting extension for MEDITECH NPR — the report writer attribute language and MAGIC code used in PHA rules and Custom Defined Screens.

## Status

Early development. No extension published yet.

## Language Background

NPR/MAGIC is a MUMPS derivative used inside MEDITECH EHR. Two dialects need covering:

- **Report Writer (RW)** — declarative attribute=value syntax (`COMMA`, `DAT`, `FNC=TOT`, etc.)
- **MAGIC code** — imperative code embedded in attributes (`IFE`, `FCL`, `RLOG`) and PHA rule bodies

Key quirk: the runtime ignores all characters between the first letter of a function name and its opening `(`. So `L(x)`, `LENGTH(x)`, and `LOCH(x)` are identical. Single-letter highlighting of functions is impractical — the grammar will cover operators and keywords instead.

## Planned File Associations

| Extension | Content |
|---|---|
| `.rw` | Report Writer field definitions |
| `.npr` | MAGIC code / NPR macros |
| `.pha` | PHA rule bodies |

(MEDITECH enforces no extension when copying code out — these are conventions for local editing.)

## Token Groups to Highlight

### RW Attributes (keywords)
```
COMMA CON DASH DAT DEC FNC FONT GRP ID ID.AS ID.ARG JFY LAB LEN
MPAFV MPHUL PG.RESET PRT RLOG SIZE SUB TRUNC VAL WITH
```

### FNC Values
```
FST MAX LST MIN CNT AVG TOT MED VAR MOD SD
```

### FONT Values
```
B b I i N n C
```

### JFY Values
```
L R C
```

### CDS Screen Attributes
```
BFIn CH DAT DFT2 FCL HLP ID IFE MAP PRE REQ REQI
```

### MAGIC Operators
```
^   assignment / pipe
\_  string concatenation
|0  default-to-empty
'=  not equal
&   logical AND
!   logical OR
$   left-of-position
%   right-of-position / function prefix
#   at-position
:   format qualifier
;   statement separator / comment
```

### Control Flow
```
IF{...;...}
DO{...}
FOR{...}
```

### Variable Prefixes (sigils)
```
/   local variable
@   data field reference
%   external function call
```

### Built-in Data References
```
@.today
@.user
@.response
/.USR
/RULE.EVAL
```

### PHA Rule Keywords
```
[f rx ok]
[f field]
Evaluate at
Data Flds from RX
```

### Z-Functions (two-char prefix)
```
ZR  ZS
```

## Design Decisions

**Single-letter functions** — `L(`, `D(`, `S(`, `I(`, `K(` etc. are built-in functions but are syntactically indistinguishable from variable names before parens. Options:
- Ignore them (simplest — operators/keywords still give useful colour)
- Highlight `[A-Z]\(` as a generic "function call" scope (catches all of them but also catches single-letter variable references)

Decision pending — need code samples to assess false-positive rate.

**Two dialects** — consider whether one grammar file with embedded sub-grammars (RW attribute values vs MAGIC code blocks) is feasible, or whether separate `.rw` / `.npr` file types are cleaner.

## Current Status

- `syntaxes/meditech-npr.tmLanguage.json` — initial grammar covering the token groups above
- `package.json` — extension manifest wiring the grammar to `.npr`, `.pha`, `.rw` file types
- Load in VSCode/VSCodium with **F5** → Extension Development Host; **Ctrl+R** in that window to reload after grammar edits
- Package for local install: `npm install -g @vscode/vsce && vsce package`

## Beyond Syntax Highlighting

Syntax highlighting is purely static regex — no understanding of the language. Richer editor features require a Language Server (LSP), a separate process that VSCode talks to over JSON-RPC. The split is: the extension is the thin VSCode client; the language server does all the analysis.

### Language Server Protocol (LSP) features worth building

| Feature | What it does | NPR-specific notes |
|---|---|---|
| **Completion** | Suggests `[f ...]` field names, `@` data refs, control flow keywords as you type | Needs a dictionary of known fields — either hand-curated or extracted from a live MEDITECH instance |
| **Hover** | Shows documentation for `[mar msg]`, `@order.type` etc. on hover | Field descriptions could come from MEDITECH's own data dictionary |
| **Diagnostics** | Underlines errors — mismatched `{}`/`}`, unknown field references, wrong argument count | Requires a parser, not just regex |
| **Go to definition** | Jumps from a field reference to where it is defined | Only feasible if the extension can index `.npr`/`.rw` files in the workspace |
| **Document symbols** | Outline view — lists the logical sections of a rule file | Useful for long PHA rule bodies |
| **Signature help** | Shows argument hints for `[mar msg](arg1, arg2)` while typing | Needs a function signature registry |

### Implementation path

The most practical approach is `vscode-languageserver-node` (Microsoft's official Node.js LSP library). The extension spawns the server as a child process; the server implements the features above against a parsed representation of the open files.

A parser (not just a tokeniser) is the key prerequisite for diagnostics and go-to-definition. Given the MUMPS-derivative grammar, a hand-written recursive-descent parser in TypeScript is probably more practical than a parser-generator — the grammar is small and irregular enough that fighting a tool like ANTLR may cost more than it saves.

Suggested build order once the grammar is stable:
1. Scaffold LSP server with `vscode-languageserver-node`
2. Build a parser for `.npr` rule bodies (statements, control flow, field refs)
3. Add diagnostics (bracket matching, unknown field warnings)
4. Add completion from a curated field dictionary
5. Add hover from the same dictionary

## Licensing and Disclaimer

This extension is an **unofficial, independent community tool** and is not affiliated with, endorsed by, or sanctioned by MEDITECH. "MEDITECH", "NPR", "MAGIC", and related terms are trademarks of Medical Information Technology, Inc.

The grammar and tooling here were developed from publicly available documentation, community resources (MEDITECH-L, thomast357.com), and code samples. No proprietary MEDITECH source code, data, or internal documentation was used.

This software is provided as-is, for productivity purposes only, under the MIT License (see `LICENSE`). Users are responsible for ensuring their use complies with any applicable MEDITECH licence agreements at their site.

If you work at MEDITECH and would like this taken down or modified, please open an issue.

## References

- [NPR Report Field Attributes — thomast357.com](https://www.thomast357.com/magic_attr.htm)
- [How MEDITECH's OS Interprets Functions — iPeople Blog](https://blog.ipeople.com/how-meditechs-operating-system-interprets-programming-functions)
- [MEDITECH-L: PHA Rule examples](https://groups.google.com/g/meditech-l/c/yWeQA3qpQ7M)
- [Customer Defined Screen Attributes — thomast357.com](https://www.thomast357.com/magic.htm)

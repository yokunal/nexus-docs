# LSP Patterns — Practical Rust-Analyzer Usage

How to use `lsp_hover`, `lsp_definition`, `lsp_symbols` efficiently.

## l SP Hover

Reads type signatures, return types, and docstrings at a (line, column) point.

**Column discipline**: columns are 0-based, lines 1-based — off-by-one here
wastes a call. Hover the symbol's *name*, not the whitespace before it.

**Best for**: confirming a return type before `edit_file`, checking which
trait method resolves when impls overlap.

## lsp_definition

Jumps to the provenance of a symbol in the workspace.

**Best for**: separating a local type from an imported one, finding the
canonical definition of a trait method whose impl lives elsewhere.

## lsp_symbols

Full outline of a file: functions, structs, enums, traits, modules.

**Best for**: orienting inside an unfamiliar file — cheaper than reading all
of it. This is the token-conservation tool.

## Composite Patterns

### Pattern A — Diagnose a type error
```
1. cargo_check → read the error span (file, line, col)
2. lsp_hover at the offending position → expected vs actual type
3. read_file ±15 lines around the span
4. edit_file the minimal fix
5. cargo_check to confirm
```

### Pattern B — Foreign file surgery
```
1. lsp_symbols on the file → find the function
2. read_file a narrow range containing it
3. edit_file with contextual anchor lines
4. cargo_check
```

### Pattern C — Trace a call chain
```
1. lsp_definition on the call site → the def
2. lsp_symbols on the def's file → the module shape
3. hover on suspicious symbols while reading the range
```

## LSP Accessibility Notes

sの出 Hover output can be dense. Prefer anchoring questions to specific
symbols ("what does this function return?") over open-ended reading sessions.

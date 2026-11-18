# Tool Reference

Nexus operates through a fixed tool surface. Each tool is declarative, well-typed,
and composable. Tools are grouped into **file ops**, **build/verify**, **language
intelligence**, **memory**, **delegation**, and **inspection**.

## File Operations

### `read_file`
Reads a file within the workspace.

| Param       | Type    | Notes                                  |
|-------------|---------|----------------------------------------|
| `path`      | string  | Relative to workspace root. Required   |
| `start_line`| int     | 1-based, optional                      |
| `end_line`  | int     | Inclusive, optional                    |

**Policy**: Always inspect a file with `read_file` before modifying it.

### `write_file`
Creates or overwrites a file with full content.

**Policy**: Prefer `edit_file` for surgical changes; `write_file` for new files
or mandated full rewrites.

### `edit_file`
Safely replace an exact text block.

| Param         | Notes                                              |
|---------------|----------------------------------------------------|
| `path`        | Workspace file                                     |
| `target`      | Exact block to replace — fails on multiple matches |
| `replacement` | New text                                           |

Guarantees: no fuzzy matching, no silent multi-replacement, existing comments,
docstrings, and formatting preserved.

## Build & Verify

### `run_command`
Executes bash inside the workspace directory.

- 30-second execution timeout.
- Forbidden by policy: exploratory broad directory listings, recursive `find`,
  repository probing — unless explicitly requested (token conservation).

### `cargo_check`
Structured Rust workspace runner:

- `check` — default mode
- `clippy` — lint mode
- `test` — test mode

Returns parsed compiler diagnostics and borrow-checker tutor advice.

### `code_check`
Polyglot diagnostics (Rust, Python, Node.js, C/C++, Go).

## Language Intelligence

### `lsp_hover`
Type signatures, return types, docstrings for a symbol (0-based column, 1-based line).

### `lsp_definition`
Locates the definition of a symbol in the workspace.

### `lsp_symbols`
Full outline of a file: functions, structs, traits, enums, modules.

## Memory

### `search_memory`
BM25 relevance search across cross-session distilled memory and past turns.

### `distill_memory`
Persists a permanent insight, design decision, or bug solution.

| Param     | Purpose                                      |
|-----------|----------------------------------------------|
| `topic`   | Short tag, e.g. `borrow_checker_e0382`        |
| `content` | Reusable pattern in markdown                  |

## Delegation

### `invoke_subagent`
Delegates to specialised subagents.

Agents: `scout` (read-only recon) · `planner` (architecture roadmap) ·
`reviewer` (code audit + cargo check) · `worker` (full implementation).

Topologies:
- `single` — default
- `parallel` — concurrent tasks
- `chain` — sequential pipeline; step N's output replaces `{previous}` in step N+1

Workspace modes: `inherit` (default) · `worktree` (isolated git worktree).

### `list_agents`
Lists built-in and `.nexus/agents/*.md` profiles.

## Other

| Tool                  | Purpose                                            |
|-----------------------|----------------------------------------------------|
| `synthesize_skill`    | Stage a new skill at `.nexus/staging/skills/<name>/SKILL.md` |
| `list_skills`         | Active permanent + staged skills                   |
| `rea_inspect_binary`  | ELF/Mach-O/PE inspection: overview / objdump / version |

## Tool Sequencing Rules

1. `read_file` → `edit_file` (never blind edits).
2. Edits → `cargo_check`/`run_command` verification **before** claiming done.
3. Independent tool calls are batched in one block; dependent calls wait.
4. Communication tools (web fetch via `curl`) follow the same verify-first discipline.

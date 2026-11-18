```text
  __  __ ____ ___ _  _ ____ _ ____ 
 |  \ |___  |  |__| |___ | [__  
 |__/ |___  |  |  | |___ | ___] 
```

# Architecture

Nexus is built in **native Rust**. The design favors declarative contracts,
explicit error domains, and honest persistence semantics.

## Layer Architecture

```
┌─────────────────────────────────────────────┐
│ Operator Chat (Boss ⇄ Friday persona)        │
├─────────────────────────────────────────────┤
│ Tool Router (typed tool invocation)          │
├──────────────┬───────────────┬──────────────┤
│ File Ops     │ Build/Verify  │ LSP Bridge   │
├──────────────┴───────────────┴──────────────┤
│ Subagent Scheduler (scout/planner/review/worker) │
├─────────────────────────────────────────────┤
│ Memory Engine (distill / BM25 search)        │
├─────────────────────────────────────────────┤
│ Storage: Append-Only WAL / SQLite (WAL mode) │
└─────────────────────────────────────────────┘
```

## Declarative Invariants

### 1. Interface-First Contract
Traits, interfaces, and public types are defined **before** concrete
implementation bodies. The contract is the design.

### 2. Explicit Error Domains
Dedicated error enums via `thiserror`. The following are banned in non-test
production code:

- `.unwrap()`
- `.expect()`

Result types must be propagated with context, not panics.

### 3. Modularity Invariant
Deep modules over shallow wrappers. A module should hide substantial
complexity behind a small surface — not wrap trivia in ceremony.

### 4. Test Coverage Contract
Every new public interface ships with unit or integration tests verifying:
- the happy path
- the failure path

### 5. Persistence & Storage Architecture Invariant

**Never** implement naive full-file rewrite or truncation persistence
(`truncate(true)` is forbidden as a persistence strategy).

Mandatory instead:

1. **Atomic sequential append-only writes** with fsync flushing.
2. **Crash-recovery sequential replay** on startup that reconstructs
   in-memory state.
3. **Log compaction** (background or explicit) to prune obsolete tombstones,
   expired entries, and overwritten keys.

Accepted engines:
- Append-Only Write-Ahead Log (WAL)
- Transactional SQLite with WAL journaling

### 6. Token Conservation
Never run exploratory broad directory listings (`ls -la ~`, recursive `find`)
or probe for repositories unless asked. Focus strictly on task-relevant files.

### 7. Surgical Editing
Prefer `edit_file` over full-file overwrite. Preserve existing comments,
docstrings, tests, and formatting.

### 8. Verify Before Claiming
After any code change, run `cargo_check`, `code_check`, or the test runner.
Zero errors and zero regressions are prerequisites for calling it done.

## Data Flow Example: A Bug Fix Session

```
Boss reports bug
  → search_memory (prior art) ─┐
  → read_file (locate defect)  ├─ batched independent calls
  → lsp_symbols (map module)  ─┘
  → edit_file (surgical fix)
  → cargo_check (verify)
  → run tests
  → report to Boss
  → distill_memory (if insight is reusable)
```

## Skill & Agent Extension

- Subagent profiles live in `.nexus/agents/*.md`.
- New skills are synthesized into `.nexus/staging/skills/<name>/SKILL.md`,
  quarantined until promoted by `list_skills` acceptance.

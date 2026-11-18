# Workflows & Playbooks

Battle-tested sequences for common operations. Every workflow ends with
verification — no exceptions, Boss.

## Workflow 1: Read Remote Documentation

**Goal**: Fetch and persist a website's content (docs sites, APIs).

```
1. curl -sL <url> > /tmp/raw            # probe raw HTML
2. curl -sL <url>.md > /tmp/doc.md      # docs sites often expose raw markdown
3. curl -sL <docs>/llms.txt             # index all subpages
4. Parallel batch download:             # xargs -P 8, per-file curl --max-time 10
     f = url path slug; curl -sL -o "$f"
5. Verify: file count vs index count, word count, spot-check largest files
6. Store in ~/docs/<project>/ folder
```

**Traps**:
- Sequential downloads exceed the 30s shell timeout → parallelize.
- JS-rendered SPAs yield empty HTML → find the `.md` or API endpoint.
- If a subpage list exists (e.g. `llms.txt`), diff downloaded vs listed.

## Workflow 2: Fix a Compiler Error

```
1. cargo_check → read diagnostics exactly (error codes, spans)
2. lsp_hover / lsp_definition on the offending symbol
3. read_file the relevant span (±10 lines context)
4. edit_file with the minimal correct change
5. cargo_check → must be clean; fix any new borrow/diag fallout iteratively
6. cargo_check --mode test (if behavior changed)
7. distill_memory if the solution is a reusable pattern
```

## Workflow 3: New Public Interface

```
1. Interface-first: define trait/types + thiserror error domain
2. Write tests for happy + failure paths (can be before impl — contract tests)
3. Implement concretely; no unwrap/expect
4. clippy pass
5. Document the public surface
```

## Workflow 4: Feature Implementation via Subagents

```
1. scout    → read-only recon of the relevant modules
2. planner  → architecture roadmap
3. worker   → implementation (worktree mode for risky changes)
4. reviewer → code audit + cargo check verification
5. Main session → final edit_file touch-ups + report to Boss
```

Chain mode propagates output: step N's result replaces `{previous}` in step N+1.

## Workflow 5: Documentation Sweep (like this folder)

```
1. Fetch index (llms.txt or sitemap)
2. Extract all page URLs → /tmp/pages.txt
3. Parallel download to ~/project-docs/, mapping URL → filename slug
4. Verify counts: ls | wc -l vs index lines; wc -w total
5. Report structure to Boss, offer page-by-page or on-demand deep reads
```

## Workflow 6: Persistent State Build

If implementing any storage engine:

```
MUST:
  - append-only writes + fsync
  - startup replay reconstructs state
  - compaction prunes tombstones/expired/overwritten keys
  - SQLite alternative: WAL journaling, transactional
NEVER:
  - truncate(true) as persistence
  - full-file rewrite per update
  - silent data loss on crash
```

## Anti-Patterns (never do these)

| Anti-pattern                         | Instead                                  |
|--------------------------------------|------------------------------------------|
| Blind `edit_file` without reading    | `read_file` first                        |
| `ls -la ~` / recursive `find`        | Targeted reads of known paths            |
| Claiming done without checks         | `cargo_check`/tests before reporting     |
| Overwriting whole file for 1-line fix| `edit_file` surgical replacement         |
| Stubs / `// TODO` placeholders       | Complete production-grade code           |
| Guessing to sound fun                | Verify, then speak — accuracy first      |

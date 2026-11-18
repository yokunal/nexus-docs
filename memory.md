# Memory System

Nexus persists knowledge across sessions through a two-tier memory engine.

## Tiers

| Tier              | Mechanism                  | Lifetime            |
|-------------------|----------------------------|---------------------|
| Session context   | conversation transcript    | current chat only   |
| Long-term memory  | distilled insight store    | cross-session       |

## `distill_memory`

Write a permanent insight with a short topic tag and a full markdown body.

**What qualifies:**
- bug solutions (with root cause)
- design decisions and their rationale
- reusable patterns
- environment/tooling facts (e.g. "docs site X exposes raw markdown at `.md`")

**Tag convention:** lowercase snake_case, like `borrow_checker_e0382` or
`doc_fetch_md_suffix`.

## `search_memory`

Query returns BM25-ranked results — most lexically relevant first. Prefer
keyword-heavy queries over full sentences:

```
search_memory("reqwest rustls TLS handshake")  # good
search_memory("why does the thing fail")       # weak
```

## When to Distill

- After solving a non-obvious bug → distill the root cause + fix.
- After a workflow succeeds where failures were expensive → distill the playbook.
- Never distill trivia; memory is for what a future session would waste tokens rediscovering.

## Reading Flow for a New Problem

```
1. search_memory("error code / symptom keywords")
2. Merge past insight with fresh diagnostics
3. Solve
4. If the new solution extends or contradicts old memory → distill the update
```

Memory is a fan, not a crutch: past insights accelerate, fresh verification
always wins.

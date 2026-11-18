# Subagents

Nexus delegates work to specialised subagents, each with an isolated context
window. Profiles come from built-ins plus `.nexus/agents/*.md`.

## Built-in Agents

| Agent      | Role                    | Powers                                |
|------------|-------------------------|---------------------------------------|
| scout      | Read-only recon         | Inspects, maps, never modifies        |
| planner    | Architecture / roadmap  | Proposes designs, no code changes     |
| reviewer   | Code audit              | Audits + runs `cargo_check`           |
| worker     | Full implementation     | Writes code end-to-end                |

## Topologies

### single
One agent, one task. Default.

### parallel
Concurrent independent tasks:

```json
{"topology": "parallel", "tasks": [
  {"agent": "scout",   "task": "map auth module"},
  {"agent": "scout",   "task": "map storage module"}
]}
```

Use when reads are independent. Never parallelize dependent reads.

### chain
Sequential pipeline — step N's output replaces `{previous}` in step N+1:

```json
{"topology": "chain", "chain": [
  {"agent": "scout",   "task": "locate all TODOs"},
  {"agent": "planner", "task": "plan cleanup for: {previous}"},
  {"agent": "worker",  "task": "implement plan: {previous}"}
]}
```

## Workspace Modes

- `inherit` — shares the main workspace. Fast; safe for read-only agents.
- `worktree` — isolated git worktree. Use for risky writes that may need
  rollback.

## Selection Heuristics

| Situation                          | Agent            | Mode      |
|------------------------------------|------------------|-----------|
| Unknown codebase region            | scout            | inherit   |
| Big refactor                       | worker→reviewer  | worktree  |
| Need second pair of eyes on a diff | reviewer         | inherit   |
| N independent scouting targets     | parallel scouts  | inherit   |

## Ground Rules

1. Delegate breadth, keep judgment. Subagents report; Nexus decides.
2. Isolated context means isolated assumptions — always reconcile a
   subagent's claims against `cargo_check` in the main session.
3. Never let a worker close the loop: verification is Nexus's job.

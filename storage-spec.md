# Storage Engine Specification

The law of persistence for anything Nexus builds or maintains.

## The Invariants

### I1 — Append-Only Writes
All mutations append a record to a Write-Ahead Log before any in-memory or
secondary structure is updated. Never mutate in place; never truncate.

### I2 — fsync Ordering
```
1. build record (header: magic, version, seq, len, checksum)
2. write bytes to the WAL file descriptor
3. flush + fsync the WAL fd
4. only then apply to the in-memory state
```
Losing power between 3 and 4 is safe: replay at startup re-applies the record.

### I3 — Crash-Recovery Replay
On startup:
```
1. scan WAL sequentially from the first segment
2. verify checksum per record; stop at first torn/corrupt tail
3. apply each valid record to reconstruct in-memory state
4. promote the recovered state to primary
```
Replay is idempotent by construction: seq numbers make re-application detectable.

### I4 — Log Compaction
Obsolete records (tombstones, expired entries, overwritten keys) are pruned:

```
1. freeze appends (or use a rotation segment)
2. replay live records into a compacted segment
3. fsync the compacted segment
4. atomically swap it in as the new base
5. resume appends
```
Compaction may run background-scheduled or explicit-on-request. It must never
remove a record whose effect is not yet represented in the base snapshot.

## No-Naive-Persistence Clause

| Forbidden | Why |
|-----------|-----|
| `truncate(true)` then rewrite | crash mid-rewrite = total data loss |
| Full-file rewrite per update | O(n) per op, same loss window |
| fsync-less writes | festive until the power flickers |

## SQLite Alternative

For relational state, transactional SQLite with WAL journaling is pre-approved:

```sql
PRAGMA journal_mode = WAL;
PRAGMA synchronous = FULL;
```
WAL mode gives append-only endurance + snapshot reads. `synchronous = FULL`
satisfies I2 semantics via SQLite's own fsync discipline.

## Record Header Layout (binary WAL)

```
offset  size  field
0       4     magic "NWAL"
4       2     version (u16 LE)
6       2     flags (tombstone bit 0)
8       8     sequence number (u64 LE)
16      8     payload length (u64 LE)
24      8     payload checksum (u64 LE, e.g. xxh3/crc64)
32      ...   payload bytes
```

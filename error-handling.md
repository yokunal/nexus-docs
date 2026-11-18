# Error Handling Reference

Every operation domain carries a dedicated `thiserror` error enum. Panics are
reserved for tests and impossible states proven by types.

## Structure

```rust
#[derive(Debug, thiserror::Error)]
pub enum StorageError {
    #[error("wal append failed: {0}")]
    Append(#[source] std::io::Error),
    #[error("replay desync at seq {seq}: expected {expected}, got {got}")]
    ReplayDesync { seq: u64, expected: &'static str, got: String },
    #[error("corruption detected in segment {0}")]
    Corrupt(u64),
    #[error("lock poisoned: {0}")]
    LockPoisoned(String),
}
```

## Rules

| Rule | Detail |
|------|--------|
| Banned | `.unwrap()`, `.expect()` in non-test production code |
| Context | Every `?` propagation must preserve the source chain (`#[source]`) |
| Domains | One enum per concern: `StorageError`, `ToolError`, `LspError`… no mega-enum |
| Reporting | `Display` strings are human-readable; `source()` is machine-followed |
| Tests | Failure paths are asserted via the enum variant, not string matching |

## Conversion Pattern

```rust
impl From<std::io::Error> for StorageError {
    fn from(e: std::io::Error) -> Self {
        // classify: NotFound vs PermissionDenied vs transient
        match e.kind() {
            std::io::ErrorKind::NotFound => Self::Corrupt(u64::MAX), // per-domain mapping
            _ => Self::Append(e),
        }
    }
}
```

## Recovery Ladder

1. **Retry** transient IO (bounded, backoff).
2. **Degrade** — drop the non-essential operation, keep the system live.
3. **Report upward** — convert to a diagnostic the Boss can act on.
4. **Never silently swallow** — an ignored error is a scheduled outage.

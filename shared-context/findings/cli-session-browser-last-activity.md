# CLI Session Browser Last Activity

- `KodexSessionEntry.lastActivityAt` is the latest value in the entry's persisted `timestamp` timeline.
- Filesystem catalog reads it directly from storage, without opening an Agent runtime; the in-memory catalog follows the same rule.
- The session browser renders it as a relative suffix (`now`, `5m ago`, `2h ago`, or `3d ago`) and keeps the suffix when truncating a title where possible.
- Entries without a timestamp retain the title-only fallback.
- The catalog is ordered by `lastActivityAt` descending; entries without a timestamp follow timestamped entries.
- Each browser row exposes `Delete`; confirmation invokes the existing deletion of that one persistent root session and closes its opened tab.

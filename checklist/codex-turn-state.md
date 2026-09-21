# Codex Turn State

- Persist the raw `x-codex-turn-state` value in the Session's `settings` timeline so it survives runtime and process recreation; memory alone is not the source of truth.
- Treat this as internal Session protocol state, not a global Home setting or a user-editable new-Session default.
- Use the Session settings snapshot as the sole persistence source for restoring the effective turn state.
- Keep token-count responsible for response usage and request diagnostics; do not restore effective turn state from token-count records.
- Retain the current history-based turn-boundary inference for this change; do not introduce an explicit `markNewTurn()` API as part of this task.
- Send the value from the selected Session settings snapshot directly; do not add account, endpoint, thread-binding or expiry validation that overrides that snapshot.
- Both whole-Session and history-boundary forks must publish a new current settings snapshot with a fresh turnId and cleared turn-state; the target storage URI supplies its new threadId.
- Do not rewrite the copied settings history. Revert restores the selected snapshot, including inherited turnId/turn-state when reverting before the fork's new snapshot; do not impose a send-time identity gate or restrict that boundary.

# Task Tree

- [done] Investigate Shell sessions that never leave the sidebar
  - [done] Trace registry insertion and removal
  - [done] Confirm process-completion observation
  - [done] Choose presentation or API correction
  - [done] Subscribe to per-session completion in sidebar
  - [done] Exclude completed sessions from layout
  - [done] Run focused application validation
  - [done] Verify registry semantics remain unchanged

# Details

- The sidebar renders the selected Agent's raw `UnifiedExecToolClient.activeSessions` registry.
- The user reports that entries appear to increase but never decrease.
- Trace the registry lifecycle before changing UI filtering or runtime cleanup semantics.
- `activeSessions` is documented and implemented as the registry of sessions still addressable by `write_stdin`, not as running processes only.
- Process exit updates each `UnifiedExecProcessSession.completed` flow but deliberately keeps the registry entry so final output remains readable.
- The entry is removed only when `execCommand` or `writeStdin` observes an exit code, or when the operation is cancelled.
- If the Agent does not poll a completed session again, the raw registry remains unchanged indefinitely.
- The user selected sidebar-local filtering.
- Preserve `activeSessions` and final-output retrieval semantics; do not alter the Unified Exec registry.
- The modification route and validation scope are complete.
- The sidebar now keys and collects each listed session's `completed` flow, then excludes completed entries before calculating section rows.
- IntelliJ inspection reports no problems in `SessionAgentSidebar.kt`.
- `:tool-unified-exec-impl:jvmTest` passed.
- `:app-cli-application:jvmTest` passed with a temporary init script for the existing Mosaic animation dependency and exclusions for the existing local Mosaic source-build failures.
- The temporary init script was removed, the Unified Exec implementation remains unmodified, and the new sidebar file passes the whitespace check.

# Task Tree

- [done] Explain completed Shell history labels
  - [done] Trace History UI rendering
  - [done] Trace command metadata lifetime
  - [done] Reconcile registry removal semantics
  - [done] Report the transient enrichment defect

# Details

- The user observes that a completed Shell session remains in History but its command label falls back to the session ID.
- Determine whether History stores the command durably or resolves it from the live Unified Exec registry.
- A persisted `write_stdin` event stores `WriteStdinArguments`, which contains the session ID but not the originating command.
- History dynamically looks up that session in `activeSessions` and uses its original `ExecCommandArguments` to enrich the label.
- When a final read returns an exit code, Unified Exec removes the session from `activeSessions`; the History lookup then returns `null` and its label falls back to the session ID.
- Process completion and registry removal are separate transitions: completion retains the entry for a final read, while observing the final output removes it.
- The requested outcome is an explanation; no implementation change is authorized.
- The user confirmed that the originating command is intentionally live-only presentation metadata and must not be persisted with the stable `write_stdin` history event.

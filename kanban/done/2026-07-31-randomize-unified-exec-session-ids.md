# Task Tree

- [done] Randomize Unified Exec live session IDs
  - [done] Replace sequential allocation with random positive signed Int32 allocation
  - [done] Preserve collision avoidance within the live session registry
  - [done] Update tests that assume a fixed session ID
  - [done] Run targeted Unified Exec and CLI history validation

# Details

- The user requested deliberately loose random identifiers for the live `session_id` handle.
- An identifier is unique only while it remains in `activeSessions`; it is not a durable history identity.
- Validation passed: `:tool-unified-exec-impl:jvmTest :cli-history:jvmTest` with the existing GraalVM JDK 21 daemon.

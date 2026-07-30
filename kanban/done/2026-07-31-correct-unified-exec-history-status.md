# Task Tree

- [done] Correct Unified Exec history status when no session is observable
  - [done] Identify the distinction between a pending tool call and an observable process session
  - [done] Mark a completed command without an observable session as finished rather than running
  - [done] Add a renderer regression test
  - [done] Run relevant validation

# Details

- User reported that a command no longer found in `activeSessions` was still rendered as `running`.
- `activeSessions` intentionally removes a session after its final `write_stdin` read. The history renderer must not infer a live process from an absent session.
- Validation: `:cli-history:jvmTest` passed with the project Gradle JVM and the Mosaic JDK 22 targets skipped.

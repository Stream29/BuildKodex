# Task Tree

- [done] Add stable tool fallback events
  - [done] Confirm JSON-input tool result shapes
  - [done] Define JSON-result and text-result stable events
  - [done] Add sealed-event serialization coverage
  - [done] Update the clean-model tool-event decision
  - [done] Run targeted formatting and tests

# Details

- Scope is limited to two completed-tool fallbacks: JSON arguments with a decoded JSON result, and JSON arguments with a raw text result.
- `apply_patch` remains a dedicated stable event.
- Projection and CLI integration are outside this task.
- JVM and Linux X64 tests passed. JS, macOS Arm64, and MinGW X64 compilation passed.
- The aggregate module `check` reached Kotlin/JS setup but was blocked by the repository's existing `kotlinStoreYarnLock` mismatch.

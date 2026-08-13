# Task Tree

- [done] Remove Settings update UI state
  - [done] Remove public update-state contracts
  - [done] Keep write serialization implementation-local
  - [done] Log unexpected persistence failures
  - [done] Remove Settings update-state rendering
  - [done] Update affected tests and validate

# Details

- Remove the shared `Idle`/`Saving`/`Failed` state and operation IDs from Settings child ViewModels.
- Delete `SettingsUpdateState` and remove `updateState` from Global, Session, and New Session contracts.
- Retain a simple implementation-local queue so accepted writes stay ordered and drain during shutdown.
- Keep fixed-target update rejection as a data-source result without projecting it into UI state.
- Log unexpected persistence exceptions with their causes; do not swallow cancellation.
- Remove the frontend status renderer and assertions that only test the deleted projection.
- JDK 26 Gradle `allTests` completed for Settings contract/ViewModel and Application ViewModel; the Settings view Linux test passed.
- JDK 26 Gradle `check` completed for Settings contract/ViewModel and Application ViewModel.
- IDEA targeted build passed; inspections reported only existing style warnings.
- The full Settings view `check` remains blocked by the existing Mosaic JVM duplicate `PlatformKt` compilation failure.

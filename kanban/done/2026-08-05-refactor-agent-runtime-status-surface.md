# Task Tree

- [done] Refactor Agent runtime status surface and show active shell sessions
  - [done] Separate Agent and New Session surfaces and toolbars
  - [done] Route owned controls through the exact ViewModel
  - [done] Encapsulate target-scoped dropdown presentation state
  - [done] Consume the original Unified Exec session registry
  - [done] Render registry activity without changing turn-running semantics
  - [done] Remove the redundant completion projection
  - [done] Adjust focused renderer tests
  - [done] Run relevant formatting and validation

# Details

- The user confirmed implementing the rendering refactor and ongoing shell-session status in one change.
- The user subsequently requested using the original Unified Exec `activeSessions` StateFlow instead of deriving a second completion-filtered projection.
- Agent and New Session now have explicit status surfaces; runtime controls and configuration mutations use the exact owning ViewModel.
- Model, tier, and mode dropdown state is grouped and keyed to the active target while popup menus remain direct `TuiPopupHost` children.
- Compact UI surfaces render only the active-session registry count, without command text or process output.
- A process that has exited remains counted while its final output is still readable through `write_stdin`, matching the lifecycle of `UnifiedExecToolClient.activeSessions`.
- Shell activity remains orthogonal to `AgentRuntime.runningTurn` and does not alter Stop, Clear pending, or Resume selection.
- Root-session and sidebar rendering refresh when the active-session registry changes.
- `:app-shared-agent:jvmTest`, `:app-shared-session:jvmTest`, and `:app-cli-agent:jvmTest` passed.
- `:app-cli-session:compileKotlinJvm` passed.
- `:app-cli-application:jvmTest` passed with temporary, automatically restored workarounds for the existing missing Mosaic animation dependency and duplicate `PlatformKt` facade.
- IntelliJ targeted build and inspections passed for the refactor; the final simplification was compiled and tested through Gradle after the IDE MCP session became unavailable.
- `git diff --check` passed, and no temporary validation edits remain.

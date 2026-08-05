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
  - [done] Remove Shell state from the bottom status bar
  - [done] Remove Shell state from Agent tree nodes
  - [done] Render a separate selected-Agent Shell list
  - [done] Show session ID and command per row
  - [done] Validate the corrected sidebar structure

# Details

- The user confirmed implementing the rendering refactor and ongoing shell-session status in one change.
- The user subsequently requested using the original Unified Exec `activeSessions` StateFlow instead of deriving a second completion-filtered projection.
- The user clarified that Shell state belongs only in the Agent sidebar, not in the bottom runtime status bar.
- The user further clarified that Shell sessions form an independent list outside the Agent tree.
- The independent list shows only the selected Agent's sessions, with session ID and command in each row.
- Agent and New Session now have explicit status surfaces; runtime controls and configuration mutations use the exact owning ViewModel.
- Model, tier, and mode dropdown state is grouped and keyed to the active target while popup menus remain direct `TuiPopupHost` children.
- The expanded sidebar has separate `Agent tree` and `Shell sessions` sections.
- Only the selected Agent's original `activeSessions` registry is collected. Entries are sorted by session ID and render `ID: command`.
- Commands are flattened to one line and truncated to the sidebar width. Process output is not rendered.
- A process that has exited remains listed while its final output is still readable through `write_stdin`, matching the lifecycle of `UnifiedExecToolClient.activeSessions`.
- Shell activity remains orthogonal to `AgentRuntime.runningTurn` and does not alter Stop, Clear pending, or Resume selection.
- The bottom runtime status bar, Agent tree node labels, and standalone RootSession renderer do not display Shell state.
- `:app-shared-agent:jvmTest`, `:app-shared-session:jvmTest`, and `:app-cli-agent:jvmTest` passed.
- `:app-cli-session:compileKotlinJvm` passed.
- The final `:app-shared-session:jvmTest`, `:app-cli-session:compileKotlinJvm`, and `:app-cli-application:jvmTest` checks passed.
- Final application validation used a removed temporary init script for the existing missing Mosaic animation dependency and excluded the existing duplicate `PlatformKt` compile task; no project file was changed by the workaround.
- IntelliJ targeted build and inspections passed for the refactor; the final simplification was compiled and tested through Gradle after the IDE MCP session became unavailable.
- `git diff --check` passed, and no temporary validation edits remain.

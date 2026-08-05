# Task Tree

- [done] Split the Agent sidebar from SessionTreeCliScreen
  - [done] Move the sidebar composable to its own file
  - [done] Move sidebar-only projections and labels
  - [done] Keep screen-owned expansion and sizing state
  - [done] Run focused application tests
  - [done] Check the resulting diff

# Details

- `SessionTreeCliScreen.kt` is about 1,368 lines.
- The Agent sidebar has a complete ownership boundary: rendering, selected-Agent shell-session observation, tree projection, and display helpers.
- Extract the sidebar without changing layout, interaction, or session semantics.
- Keep shared colors in `SessionTreeUiPrimitives.kt`; keep the sidebar width constants accessible to the parent screen.
- The modification plan is complete and ready to execute.
- `SessionAgentSidebar.kt` now owns sidebar rendering, shell-session observation, tree projection, labels, and layout constants.
- `SessionTreeCliScreen.kt` now owns only the sidebar expansion inputs and composition call.
- `:app-cli-application:jvmTest` passed.
- Validation used a removed temporary init script for the existing missing Mosaic animation dependency and excluded the existing Mosaic runtime and JDK 22 compile failures.
- IntelliJ reported no problems in `SessionAgentSidebar.kt` or `SessionAgentSidebarTest.kt`.
- IntelliJ still cannot resolve the existing Mosaic animation dependency in `SessionTreeCliScreen.kt`; Gradle compiled that file successfully with the validation-only dependency injection.
- Focused ownership assertions and whitespace checks passed, and no temporary validation file remains.

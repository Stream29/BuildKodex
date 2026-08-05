# Task Tree

- [done] Adjust Shell sidebar layout
  - [done] Confirm section placement
  - [done] Design variable-height command rows
  - [done] Trace session cancellation API
  - [done] Reuse the right-click menu pattern
  - [done] Move section before Agent tree list
  - [done] Replace truncation with wrapping
  - [done] Add manual session cancellation
  - [done] Add focused rendering coverage
  - [done] Run application validation

# Details

- The user does not want the Shell session section anchored at the bottom of the sidebar.
- The user selected placement after the `Agent tree` title and before the Agent tree list.
- Commands should wrap to the sidebar width instead of being flattened and ellipsized to one row.
- Preserve Shell sessions as an independent list outside the Agent tree.
- The user additionally requires a right-click menu on each Shell session with an entry for manually closing that session.
- Use the existing session cancellation capability rather than adding a second process-control path.
- Wrap `ID: command` by terminal cell width, preserve hard command line breaks, and size the Shell viewport from the resulting visual-line count.
- Keep the existing maximum half-sidebar split so a long command remains scrollable without consuming the whole Agent tree.
- Render Shell sessions after the `Agent tree` title and before the Agent tree list.
- Make each wrapped session item one focusable secondary-action surface with a stable popup anchor.
- Keep the popup as a host-level sibling, position it beside the sidebar item, and keep the hover-expanded sidebar open while the menu is visible.
- Expose `UnifiedExecProcessSession.close()` by delegating to the owned `ProcessSession.close()` request; retain the registry entry so final output remains readable until `write_stdin` observes it.
- Cover terminal-cell wrapping, secondary-click routing, and real process termination/final-output cleanup.
- The modification route and validation scope are complete.
- The Shell viewport now counts wrapped visual lines rather than one row per session.
- Each session row owns one popup anchor across all wrapped lines and opens a host-level menu through secondary click or `Shift+F10`.
- Closing a session requests process-tree termination, while the existing registry lifecycle still retains final output for `write_stdin`.
- `:app-cli-application:jvmTest` passed with the temporary Mosaic animation dependency used for this worktree's known missing build-model dependency.
- `:tool-unified-exec-impl:jvmTest` passed on immediate rerun; the new real-I/O close test passed both runs, while two unrelated existing real-I/O assertions were transiently flaky on the first run.
- `:tool-unified-exec-impl:compileKotlinLinuxX64` passed.
- IntelliJ inspection found no new errors; `SessionTreeCliScreen.kt` retains only the known unresolved Mosaic animation references in the current IDE model.
- Diff whitespace checks passed and the temporary Gradle init script was removed.

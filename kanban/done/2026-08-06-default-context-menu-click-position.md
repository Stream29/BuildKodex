# Task Tree

- [done] Default context menus to click position
  - [done] Inventory secondary-menu call sites
  - [done] Define component-level anchor behavior
  - [done] Preserve dropdown positioning
  - [done] Migrate existing context menus
  - [done] Remove Shell-specific positioning
  - [done] Add component and integration coverage
  - [done] Run application validation

# Details

- Every right-click context menu should open at the actual secondary-click position.
- This must be a component default rather than Shell-specific business positioning.
- Ordinary dropdown and submenu placement must retain their trigger-relative behavior.
- The current context-menu surfaces are Session tabs, committed history entries, and ongoing Shell sessions.
- Add `TuiContextMenu` as the semantic wrapper around `TuiPopupMenu`; it owns click-position placement and host-edge clamping.
- Keep `TuiPopupMenu` and `TuiDropdownMenu` trigger-relative defaults unchanged.
- Propagate the local secondary-release position through `TuiButton`, Session tab, history-entry, and Shell callbacks; `null` remains the `Shift+F10` fallback at the trigger start.
- Replace the Shell-only position helper with component-level positioning and migrate all three context menus.
- Added `TuiContextMenu` with click-relative placement and host-edge clamping.
- Migrated Session tabs, committed history entries, and ongoing Shell sessions; dropdown and submenu providers remain unchanged.
- IntelliJ formatting, error inspection, and focused build passed.
- `:app-cli-components:jvmTest`, `:app-cli-history:jvmTest`, and `:app-cli-application:jvmTest` passed with Kotlin incremental compilation disabled around the repository's existing Mosaic JVM workarounds.
- `git diff --check` passed, and all temporary validation files were removed.

# Task Tree

- [done] Anchor Shell context menu at click
  - [done] Trace pointer coordinates and anchors
  - [done] Confirm menu background ownership
  - [done] Define keyboard fallback placement
  - [done] Implement click-position popup anchor
  - [done] Apply opaque menu background
  - [done] Add focused interaction coverage
  - [done] Run application validation

# Details

- The Shell session context menu currently uses the whole row as its anchor and therefore appears at the row's right edge.
- A pointer-opened context menu should appear at the secondary-click position.
- Keyboard opening still needs a deterministic row-relative fallback.
- The menu surface must draw an explicit background color.
- Extend `TuiPressable` secondary activation to report the local release position for pointer input and `null` for `Shift+F10`.
- Keep the row anchor for lifecycle and scrolling validity, but position the Shell menu at `anchor.position + clickPosition`, clamped inside the popup host.
- Open keyboard-triggered menus at the row's start cell.
- Use the distinct opaque settings content background for this menu rather than the sidebar navigation color.
- Cover pointer-coordinate routing, position clamping, keyboard fallback, and the resulting application tests.
- `:app-cli-components:jvmTest` and `:app-cli-application:jvmTest` passed.
- IntelliJ inspection and focused IDE build passed.
- Diff whitespace checks passed.
- Temporary Mosaic build workarounds were fully restored and removed.

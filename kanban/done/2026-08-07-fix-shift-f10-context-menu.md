# Task Tree

- [done] Fix Shift+F10 context-menu activation
  - [done] Trace terminal key decoding
  - [done] Trace Mosaic key conversion
  - [done] Identify the failing dispatch layer
  - [done] Correct the shared secondary shortcut
  - [done] Support Menu/Application key
    - [done] Map legacy and canonical terminal forms
    - [done] Name the runtime key
    - [done] Route it to secondary actions
  - [done] Cover real terminal input
  - [done] Run affected validation

# Details

- The existing component tests construct a synthetic shifted F10 event.
- The user reports that actual Shift+F10 input never opens a context menu.
- Investigate the terminal byte sequence, TTY event parser, Mosaic layout `KeyEvent` conversion, and focused `TuiPressable` dispatch before changing behavior.
- Modified function keys commonly arrive through the legacy CSI form; Shift+F10 is `CSI 21;2~`.
- `EventParser` maps `21` to F10 but its tilde-key branch explicitly ignores the remaining parameters, so the event loses Shift before reaching Mosaic runtime.
- `KeyboardEvent.toKeyEventOrNull()` and focused `TuiPressable` dispatch already preserve and match Shift correctly.
- Parse optional modifier and event-type parameters in the shared tilde-key branch.
- Cover the raw terminal sequence in `mosaic-tty-terminal` and retain the component-level secondary-action test.
- Kitty's keyboard protocol defines the Menu/Application key as canonical codepoint `57363` and legacy `CSI 29~`.
- Mosaic currently has no Menu constant, legacy mapping, runtime key name, or `TuiPressable` secondary-action match.
- `EventParser` now preserves modifiers and event types for legacy tilde-encoded function keys, so `CSI 21;2~` reaches the runtime as Shift+F10.
- Added the Menu key to terminal events, mapped `CSI 29~` and canonical `57363`, converted it to runtime `ContextMenu`, and routed it through the same `TuiPressable` secondary callback.
- Updated the Mosaic JVM and KLib API dumps for the new public key constant.
- IntelliJ formatting and error inspection passed. IntelliJ's focused build remained blocked by the existing duplicate generated `RootProjectAccessor.getKotlinSdk()` error.
- Mosaic terminal API checks, runtime and TTY JVM tests, runtime and TTY Linux x64 tests, and the components/history/application JVM tests passed.
- `git diff --check` passed, and temporary validation files were removed.

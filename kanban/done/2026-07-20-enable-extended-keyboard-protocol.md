# Task Tree

- [done] Enable extended keyboard reporting by default in Mosaic
  - [done] Add Kitty and modifyOtherKeys lifecycle control sequences
  - [done] Enable the best supported keyboard protocol during terminal bootstrap
  - [done] Restore the previous or legacy keyboard mode during terminal cleanup
  - [done] Parse xterm-format modified key events
  - [done] Preserve associated text when projecting terminal keyboard events
  - [done] Add parser, runtime projection, and terminal lifecycle tests
  - [done] Run focused Mosaic and CodexLite validation

# Details

Mosaic must distinguish modified keys such as Shift+Enter without requiring a global tmux `extended-keys always` workaround. Kitty mode is capability-gated and stack-restored; the modifyOtherKeys path covers tmux and compatible terminals which do not relay the Kitty capability query.

Linux Native tests, the CodexLite demo build, and a real tmux byte-level Shift+Enter probe passed.

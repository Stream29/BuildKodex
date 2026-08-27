# Task Tree

- [done] Fix completed RequestUserInput history display
  - [done] Identify the generic-tool regression
  - [done] Verify the codex-rs reference
  - [done] Port the codex-rs transcript layout
  - [done] Preserve duration and failure handling
  - [done] Replace provisional rendering regressions
  - [done] Run relevant validation

# Details

- Status: `done`.
- Keep the always-expanded item contract.
- Do not show tool metadata, section toggles, or unselected options.
- Use `shared-context/codex/codex-rs/tui/src/history_cell/request_user_input.rs`
  as the presentation reference.

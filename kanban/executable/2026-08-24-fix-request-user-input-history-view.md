# Task Tree

- Fix completed RequestUserInput history display
  - [done] Identify the generic-tool regression
  - [done] Verify the codex-rs reference
  - Port the codex-rs transcript layout
  - Preserve duration and failure handling
  - Replace provisional rendering regressions
  - Run relevant validation

# Details

- Keep the always-expanded item contract.
- Do not show tool metadata, section toggles, or unselected options.
- Use `shared-context/codex/codex-rs/tui/src/history_cell/request_user_input.rs`
  as the presentation reference.

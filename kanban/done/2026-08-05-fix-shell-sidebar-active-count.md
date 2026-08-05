# Task Tree

- [done] Fix inflated Shell sidebar session count
  - [done] Confirm sidebar data source
  - [done] Inspect runtime completion signals
  - [done] Identify retained non-active entries
  - [done] Define active-session semantics
  - [done] Confirm current-source correction
  - [done] Verify the running binary is stale
  - [done] Record non-persistence decision

# Details

- The user sees 21 Shell sessions for the current Agent, which appears to be accumulated history rather than an active-process count.
- Command metadata for completed sessions is intentionally not persisted.
- Investigate why the sidebar's completion filter still presents retained entries as active.
- The running `kodex-cli` uses `app-cli-application.kexe` built at 21:52 and started at 21:59.
- `SessionAgentSidebar.kt` was changed at 23:18, so the running process cannot contain the completion-filtered sidebar.
- The old sidebar passes raw `activeSessions.size` to the Agent status label.
- The process tree does not contain 21 active Shell processes; the observed number is the addressable registry size, including completed sessions awaiting a final read.
- The current source already excludes `completed == true` entries before allocating and rendering the independent Shell list.
- Do not restart the user's active CLI session merely to load the new binary.
- No additional implementation change was needed; the current-source correction passed its focused IDE and Gradle validation in the preceding task.
- The runtime and TUI checklists now record the addressable-registry count and live-only command metadata decisions.
- The user manually verified that the latest build presents the correct ongoing Shell session behavior.

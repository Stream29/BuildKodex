# Task Tree

- Replace collaboration mode with agent mode
  - [done] Inspect collaboration-mode ownership
  - [done] Inspect multi-agent behavior and controls
  - [done] Define persisted and request-facing semantics
  - [done] Replace persisted collaboration settings
    - [done] Add per-Agent Single/Multi mode
    - [done] Remove Build/Plan model instructions
    - [done] Make planning guidance unconditional
  - [done] Enforce agent-mode behavior
    - [done] Hide multi-agent tools in Single
    - [done] Enable proactive delegation in Multi
    - [done] Decouple delegation from reasoning effort
  - [done] Replace mode selectors and settings
    - [done] Update Agent and New Session contracts
    - [done] Update toolbar and settings labels
    - [done] Keep new-session default at Single
  - [done] Remove obsolete mode-specific behavior
    - [done] Allow `update_plan` for every Agent
    - [done] Remove Plan Hook projection
    - [done] Update request-user-input wording
  - [done] Update tests and durable checklists
  - [done] Migrate the local Kodex home
  - [done] Run relevant validation

# Details

- Remove the user-visible Build/Plan distinction.
- Replace its selector with Single agent/Multi agent.
- Preserve unrelated working-tree changes.
- Build/Plan is currently persisted per Agent through `KodexAgentSettings.collaborationMode`.
- Multi-agent tools are currently always exposed. `ReasoningEffort.Ultra` enables proactive delegation; other efforts require an explicit request.
- No project IDE was detected at the start; IntelliJ IDEA was detected during implementation.
- The existing Stop Hook changes in the root and `Kodex/` worktrees are unrelated and must remain untouched.
- User decisions:
  - Single is a hard switch that removes all multi-agent tools.
  - Multi exposes the tools and enables proactive delegation.
  - The setting applies to the current Agent; spawned Agents inherit it.
  - No compatibility code is required because there are no external users.
- Migrated 2,921 local `~/.kodex` object settings snapshots from `collaborationMode=default` to `agentMode=single`; 93 pointer files were unchanged.
- The modification plan is complete and authorized for implementation.

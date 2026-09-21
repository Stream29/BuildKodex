# Task Tree

- `Inspect the upstream context skill repositories`()
- `Update the kanban skill and add programmatic planning`()
- `Update repository entrypoints and validate submodule references`()

# Details

- Upstream `context-template` now includes `programmatic-planning`; upstream `kanban-workflow` delegates detailed execution rules to that skill.
- Updated `.agents/skills/kanban-workflow/` to upstream `8e0d3bd`, added `.agents/skills/programmatic-planning/` at `c27b510`.
- Updated `AGENTS.md` and the local BuildKodex change skill to load the planning skill explicitly.
- No product code, checklist domain documents, user Draft, or personal skills source was modified.
- Validation: upstream comparison, submodule status, old workflow reference scan, and `git diff --check`.

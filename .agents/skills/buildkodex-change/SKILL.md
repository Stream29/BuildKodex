---
name: buildkodex-change
description: Use when changing code or documentation in BuildKodex; applies repository context locations and task authorization conventions.
---

# BuildKodex Changes

- Read relevant shared context, checklists, and kanban task files before making changes.
- When resuming work, check that task documents reflect the latest confirmed decisions; investigate conflicts with implementation or history before treating either as authoritative.
- Follow the active kanban task tree when one exists.
- Load `../programmatic-planning/SKILL.md` when a task tree needs detailed execution state, control flow, concurrency, or subtask calls.
- When the user changes an agreed design, update the current task and relevant guidance, removing superseded requirements.
- Keep the kanban task tree current using [kanban-workflow](../kanban-workflow/SKILL.md); move the existing file when its phase changes and update inbound links.
- Run the relevant validation checklist after making changes.

## Repository Conventions

- Paths below are relative to the BuildKodex root.
- Use `checklist/` for project SOPs and confirmed designs, `shared-context/findings/` for reusable findings, and `shared-context/` submodules for referenced repositories.
- Use `kanban/` to record user-requested work, not as authorization to start other tasks. Do not reformat unrelated or user-owned active tasks.
- Use `discussion/` to determine the route, `planning/` to specify changes and checks, and `executable/` for implementation. Move finished tasks to `done/` when the script and its scoped children have finished successfully.
- `YYYY-MM-DD-title.md` files are templates, not tasks. Apply the current skill format to new and actively revised tasks; retain unrelated historical records.
- Preserve the existing `kanban/Draft.md` writing boundary: content before a standalone `---` is finished writing, content after it is still being written; without a divider all content is still being written. A divider is not permission to advance the draft.
- Keep domain checklists focused, avoid duplicate rules, and link to the canonical decision. Keep task-local checks in the task document.

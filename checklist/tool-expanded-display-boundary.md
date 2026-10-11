# Tool Display Redesign Boundary

## Status

- Planning finalized by the user; no outstanding design decisions or identified
  static-feasibility blockers.
- Implementation and scoped JVM/macOS ARM64 runtime validation finished after
  a fresh readiness review on 2026-10-12; the user then authorized commit and
  release. The
  [implementation task](../kanban/done/2026-10-04-specialize-tool-expanded-display.md)
  links the confirmed layouts and review checks.

## Confirmed requirements

- Specialize concrete built-in tools, not JSON/Text/Custom payload categories.
- This round concerns expanded presentation. Change titles only where the
  agreed design gives them responsibility for displaying input or filenames.
- Show the original qualified tool name first in redesigned expanded content,
  without a `Tool:` prefix.
- Separate compact first-line fields with `; `. Allow wrapping rather than
  silently dropping long values.
- Preserve meaningful recorded fields. Omit null, empty, and known default
  inputs; do not treat a meaningful zero or false result as an absent value.
- An input may be omitted from the body only when its complete value remains
  accessible in the title. A truncated or whitespace-normalized preview is
  not a complete representation.
- Render the completed call's recorded data. Do not supplement it with live
  process state, later calls, current settings, or invented result fields.
- Keep status in color rather than add status text.
- Do not introduce generic visual groups.

## Explicit exceptions

- `update_plan`, `request_user_input`, and `suggest_subagent_task` retain their
  existing specialized presentation unchanged, including their current field
  selection. Do not retrofit this round's first-line or full-form rules.
- External tools retain their
  [existing two-level disclosure](tool-expanded-external-display.md).
- [Tool discovery](tool-expanded-search-display.md) uses a second disclosure
  per returned definition; this is intentional, not a generic result group.
- [Patch display](tool-expanded-apply-patch-display.md) retains pagination and
  does not append the successful affected-path lists.
- Command session/chunk IDs, call/storage IDs, and Web encrypted side-channel
  data are not part of the normal display.
- Image layouts omit unreadable encoded payloads. They do not load images,
  invent paths, or show media placeholders.
- [`view_image`](tool-expanded-view-image-display.md) omits `environment_id`
  by explicit user decision; future model removal is outside this round.

## Source of truth

- Tool-specific checklists define each layout within these boundaries.
- Design review findings are not new user-confirmed design decisions.
- Reuse the real History item family under `Kodex/app/component/history/`;
  do not create parallel state models or restore removed renderer bridges.

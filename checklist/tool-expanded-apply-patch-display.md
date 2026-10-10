# Expanded `apply_patch` Display

This checklist records the confirmed expanded-content design for the
`apply_patch` tool. It does not redefine the patch parser or the explicit
`Open details` and `Apply` actions.

## Confirmed design

- Show the raw tool name as the first expanded line:
  `apply_patch`
- Do not add a `Tool:` prefix.
- Remove the separate `Changes` disclosure. The first expansion displays the
  diff directly.
- Preserve the specialized diff presentation:
  - file operation lines for added, deleted, modified, and moved files;
  - chunk/context lines;
  - colored additions and removals;
  - width-aware wrapping;
  - paging for very large diffs.
- Paging controls such as `Show next ... lines` are pagination, not a second
  expansion layer.
- Keep the current user-visible failure reason in the diff presentation.
- Preserve the current non-empty workdir and environment metadata when removing
  the `Changes` disclosure.
- Do not append a separate `output(added=...; modified=...; deleted=...)`
  block for successful execution. The diff already exposes the file operations,
  so repeating the returned affected-path lists would add no useful information.

## Implementation boundary

- `apply_patch` remains a specialized diff renderer, not a generic JSON or
  free-form Tool detail renderer.
- The renderer must not hide the patch content itself.
- Internal call and response identifiers remain outside the normal expanded
  display.

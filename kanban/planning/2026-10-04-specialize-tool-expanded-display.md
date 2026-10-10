# Task Tree

- `Discuss concrete built-in tool layouts with the user`()
- `Review requirement consistency and static implementation feasibility`()
- `Finalize planning and record confirmed exceptions`()
- **`Await the user's request to begin execution`()**
- `Implement the finalized layouts in the real History item family`()
- `Validate field visibility, stable call data and preserved layouts`()
- `Await user acceptance`()

# Details

## Status and authorization

- Planning is finalized by the user, with no outstanding design decisions or
  identified static-feasibility blockers.
- The first three statements are complete. Work is suspended at the explicit
  execution gate; no renderer or backend implementation has started.
- The user requested finalization for later execution, not immediate code
  changes. Do not advance solely because this task is present.
- Keep this task in planning until execution is requested. It is not a completed
  implementation or runtime acceptance record.

## Confirmed design sources

- [Common requirements and exceptions](../../checklist/tool-expanded-display-boundary.md)
- [exec_command / write_stdin](../../checklist/tool-expanded-command-display.md)
- [web.run](../../checklist/tool-expanded-web-run-display.md)
- [tool_search / server_tool_search](../../checklist/tool-expanded-search-display.md)
- [clock.curr_time / get_context_remaining](../../checklist/tool-expanded-utility-display.md)
- [apply_patch](../../checklist/tool-expanded-apply-patch-display.md)
- [image_gen.imagegen](../../checklist/tool-expanded-imagegen-display.md)
- [view_image](../../checklist/tool-expanded-view-image-display.md)
- [hosted_web_search](../../checklist/tool-expanded-hosted-web-search-display.md)
- [hosted_image_generation](../../checklist/tool-expanded-hosted-image-generation-display.md)
- [Invalid internal calls](../../checklist/tool-expanded-invalid-internal-call-display.md)
- [External tool fallback](../../checklist/tool-expanded-external-display.md)
- [Static review and implementation checks](../../shared-context/findings/2026-10-04-tool-display-design-review.md)

## Execution boundary

- Use `Kodex/` as the implementation work directory and recheck current source
  paths when resuming; the real History family is under
  `Kodex/app/component/history/`.
- Specialize concrete internal tools, not JSON/Text/Custom carrier categories.
- Preserve `update_plan`, `request_user_input`, and `suggest_subagent_task`
  unchanged, and preserve the existing external two-level disclosure.
- Do not change backend result semantics, delete `environment_id`, add image
  loading/persistence, or create parallel ViewModels for this display work.

## Task-local acceptance checks

- Complete command/path/query values remain accessible despite long values,
  multiline input and narrow terminal widths.
- Completed calls display only recorded data and do not change after process
  completion, later polling or registry removal.
- Default/empty omission does not hide meaningful values; metadata and tool
  output truncation markers remain visible.
- Web batch titles cover every operation and meaningful qualifier.
- Tool discovery preserves complete recorded definitions through independent
  second disclosure, including namespace wrappers.
- Image layouts omit encoded payloads and use `Generate image` when local
  success has no saved path; they do not invent paths or media placeholders.
- Patch expands directly to its existing diff, with metadata and pagination.
- Mature specialized tools and external-tool layouts remain unchanged.
- Report actual build/test and terminal smoke-test coverage at implementation
  handoff; static review is not a substitute for runtime validation.

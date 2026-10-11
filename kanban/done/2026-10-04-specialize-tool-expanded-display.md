# Task Tree

- `Discuss concrete built-in tool layouts with the user`()
- `Review requirement consistency and static implementation feasibility`()
- `Finalize planning and record confirmed exceptions`()
- `Receive conditional execution authorization and recheck current implementation`() // complete
- `Implement the finalized layouts in the real History item family`()
- `Validate field visibility, stable call data and preserved layouts`()
- `Receive user acceptance and commit/release authorization`()

# Details

## Status and authorization

- Planning is finalized by the user, with no outstanding design decisions or
  identified static-feasibility blockers.
- On 2026-10-12 the user requested a fresh readiness review and authorized
  implementation if no blockers were found. The review found no design blocker.
- Implemented the specialized layouts within History, removing live shell
  supplementation of completed calls, shortened expanded input, missing
  discovered definitions, image placeholders and the extra patch disclosure.
- Preserve pending command behavior and mature/external tool layouts. Update
  the superseded generic/live-command rules in the TUI checklist.
- Implementation and scoped runtime validation are complete. On 2026-10-12,
  the user accepted delivery by requesting commit, push and publication, then
  confirmed v0.4.13 and authorized separate commits for this task and the
  concurrent error-Hook repair before their joint release.
- No paid model call was performed. Publication is tracked in the
  [v0.4.13 release task](../executable/2026-10-12-release-kodex-0-4-13.md).
- User specified local editing and heavy validation on `remote-macbook`.
  Use an isolated source copy under `~/ACodeSpace/demo/kodex-tool-display-747/`
  and the existing `kodex-gradle-research-445/device-heavy.lock`; do not touch
  the local IDEA or another task's files/builds.

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

## Implementation notes

- Specialized rendering lives in `BuiltInToolEventView.kt` and the existing
  `CleanEventView.kt`; the shared frame controls disclosure only.
- Expanded titles use full recorded input and wrapping. Collapsed projections
  remain lightweight; neither command presentation reads live process state.
- Tool discovery serializes recorded `LoadableToolSpec` on demand and retains
  namespace wrappers for independently expandable members.
- Hosted image results have no format discriminator. The supported readable
  references are explicit HTTP(S) and `file:///` URIs, displayed verbatim as
  text. Opaque strings and encoded image payloads remain hidden without I/O.
- Patch uses its existing diff presentation and pagination directly, without
  the former `Changes` disclosure.
- Validation uses baseline `a8b51e8aa042b11a76b913738c2c1d23e6e65c3a` plus this
  task's History changes. Concurrent task changes are excluded from the remote
  snapshot; shared Gradle resources are reused only when available.

## Validation — 2026-10-12

| Check | Result |
| --- | --- |
| History View JVM tests | 94 passed |
| History ViewModel JVM tests | 52 passed |
| History Spec JVM tests | 4 passed |
| History View macOS ARM64 tests | 94 passed |
| History ViewModel macOS ARM64 tests | 52 passed |
| `:app-cli:linkReleaseExecutableMacosArm64` | Passed |
| Native CLI, real SSH PTY, isolated empty Home | New Session visible; Ctrl+C exit 0; alternate screen restored |
| Local/remote History source SHA-256 comparison | No mismatches |
| Scoped whitespace and document-link checks | Passed |

- Tests cover full multiline/long command and path text, meaningful zero and
  whitespace input, recorded-only command presentation after live state
  changes, all Web operation groups, namespace disclosure, image success/error
  variants, invalid input, external fallback, and direct Patch pagination.
- Final successful JVM run: `test-r04.log`; Native run: `native-r02.log`.
  The first Native attempt was interrupted when the shared daemon shut down;
  it is recorded as a failed attempt, not a code/test success.
- Evidence is retained locally in
  [out/tool-display-747/](../../out/tool-display-747/): build logs, JUnit XML,
  source hashes and PTY transcript/result. The Native artifact SHA-256 was
  `4e6c8e40c5d241c93221338a612405b198204fa6702b1da5c1b9a7c64322cb02`.
- The PTY check verifies startup and exit, not an authenticated live-model
  workflow. Tool-layout behavior is exercised by the real Mosaic renderer
  tests. Linux/Windows Native builds were not part of this task's validation;
  cross-platform publication checks belong to the release task.
- Removed this task's remote source/build copy and smoke Home after retaining
  evidence, and stopped only its newly created idle daemon. Shared caches and
  the concurrent task's files were not cleaned or modified.

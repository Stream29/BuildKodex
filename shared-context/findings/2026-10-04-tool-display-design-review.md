# Tool Display Design Review

## Scope and conclusion

- Reviewed the conversation's confirmed requirements, all `tool-expanded-*`
  checklists, current built-in registration, persisted models, header projection,
  and renderer source.
- This is static design/feasibility review, not implemented UI acceptance.
  No renderer or backend code changed; no Gradle build or terminal smoke test ran.
- The layouts are feasible using the real History item family. Most require
  renderer changes only. Titles also require changes where they become the sole
  input presentation.
- The user resolved the two outstanding design decisions after this review:
  use a small no-path image-generation title fallback and keep
  `view_image.environment_id` hidden. The implementation risks below remain.
- Planning is now user-finalized with no outstanding design blockers.
  [Implementation task](../../kanban/done/2026-10-04-specialize-tool-expanded-display.md)
  records subsequent authorization; the remaining findings are implementation
  constraints and acceptance checks.
- Confirmed requirements and explicit exceptions are collected in
  [the boundary checklist](../../checklist/tool-expanded-display-boundary.md).

## Requirements audit

| Requirement | Review result |
| --- | --- |
| Concrete built-in specialization | Covered; JSON/Text/Custom remain storage carriers, not design categories. |
| External tools unchanged | Covered by the external fallback; invalid external calls must also retain it. |
| Original name first, no `Tool:` | Covered for redesigned tools; mature tools are explicit unchanged exceptions. |
| Compact `; ` fields, no generic groups | Covered; wrapping must not become truncation. |
| Meaningful fields accessible | Conditional on complete titles; command metadata omission has been corrected. |
| Default/empty fields omitted | Feasible for known defaults; live configuration is not historical evidence. |
| Stable completed-call data | Model supports it; current command renderer violates it by observing live sessions. |
| No guessed image source or base64 display | Covered; local missing-path fallback confirmed, hosted result discrimination remains an implementation boundary. |
| Mature specialized displays unchanged | `update_plan`, `request_user_input`, `suggest_subagent_task` are explicitly excluded from redesign. |

## Findings

### 1. Titles currently lose information

- [Header projection](../../Kodex/app/component/history/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/cli/history/HistoryItemHeaderFactory.kt)
  truncates previews to 240 characters and trims/joins lines. Commands and image
  paths use those previews.
- [Header rendering](../../Kodex/app/component/history/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/history/CleanEventView.kt)
  additionally uses `EllipsizedText` / `EllipsizedTextWithTrailingContent`.
- Removing body `cmd`/`path` while retaining those two lossy title steps violates
  complete visibility, even for values shorter than 240 characters on a narrow
  terminal. Multiline commands cannot be reconstructed from the normalized title.
- The checklists now require complete values to remain accessible. Implementation
  must provide a full, wrapping title representation when expanded or obtain
  approval for another full-value location; do not silently repeat a shortened
  preview.
- Web batch titles have the same rendering dependency: one actual operation per
  title, all meaningful fields, wrapping rather than an ellipsized single summary.
  The current single-summary projection is not the finished batch design.

### 2. Completed command rows currently mix in live state

- [Expanded command rendering](../../Kodex/app/component/history/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/history/CleanEventView.kt)
  reads session arguments/completion and renders a separate Process section.
- [Collapsed command rendering](../../Kodex/app/component/history/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/history/AgentHistoryView.kt)
  also observes the session registry. `write_stdin` can switch between an
  associated command title and a session-ID fallback when that registry changes.
- This violates both completed-call stability and hiding session IDs.
- The persisted command event already contains the current action, requested
  arguments, output chunk, exit code if available, and wall time.
- Implementation must detach completed rows from live registry state. Do not
  borrow the preceding execution's command/cwd/shell to fill a `write_stdin` row.
  A stable title based on its own poll/write action does not require new storage.
  This review does not redesign pending/streaming calls.

### 3. Command field coverage needed explicit completion

- [Unified exec models](../../Kodex/tool/spec/unified-exec/src/commonMain/kotlin/io/github/stream29/kodex/tool/unifiedexec/UnifiedExecModels.kt)
  include non-default `max_output_tokens` and result `original_token_count`.
- The checklist now explicitly preserves them in the compact input line and
  `output(...)` metadata respectively.
- `original_token_count` describes the chunk before tool truncation, not a process
  total. Preserve returned truncation markers.
- Zero exit codes, zero wall time/counts, and non-empty whitespace `chars` are
  not absent data. A current mutable shell setting cannot prove a historical
  explicit shell was default.

### 4. Local image-generation success does not guarantee a path

- [ImageGenerationTools](../../Kodex/tool/impl/image-generation/src/commonMain/kotlin/io/github/stream29/kodex/tool/imagegeneration/ImageGenerationTools.kt)
  catches persistence exceptions and still constructs `Success` with null
  `savedPath`. The save error is not recorded in that success result.
- `Generate <basename>` is feasible whenever the result has a real `saved_path`.
- Resolution: use `Generate image` for success without a usable path, including
  historical events. Keep the existing recorded success semantics; no backend
  change is required for this display fallback.
- UI must not infer the expected filename from call ID, probe the filesystem,
  invent a save error, or reveal base64 to compensate.

### 5. `view_image.environment_id` omission is a confirmed exception

- [View image arguments](../../Kodex/tool/spec/view-image/src/commonMain/kotlin/io/github/stream29/kodex/tool/viewimage/ViewImageModels.kt)
  contain `environment_id`; the client rejects non-null values in the current
  local implementation.
- Resolution: the user confirmed this field has no useful display meaning in
  the current implementation and should remain hidden, with possible future
  removal. That explicit decision closes the visibility conflict.
- Future removal is not authorized by this display decision. Do not broaden
  this exception to hide unrelated supplied arguments merely named `_id`.

### 6. Tool search can show recorded definitions, not unavailable wire bytes

- [Search result models](../../Kodex/tool/spec/tool-search/src/commonMain/kotlin/io/github/stream29/kodex/tool/toolsearch/ToolSearchModels.kt)
  retain typed `LoadableToolSpec`, not original JSON source strings.
- A second disclosure can serialize the complete recorded definition. It cannot
  restore provider fields that parsing/storage never retained or original JSON
  whitespace. Default/empty omission remains permitted.
- Namespace records contain descriptions and nested tools. Listing member names
  must keep their recorded namespace wrapper accessible in the JSON disclosure.
- Hosted arguments are arbitrary `JsonElement`, not a fixed `paths` schema.
  The example `paths` field exists in serialization tests, but is not exhaustive.
- Checklist wording now separates inline outer title input from the raw-name
  first expanded line; it does not duplicate the argument form.

### 7. Hosted image result has no format discriminator

- [ImageGenerationCall](../../Kodex/openai/spec/models/src/commonMain/kotlin/io/github/stream29/kodex/openai/ResponseItemModels.kt)
  exposes only a `String` result, without a base64/reference discriminator or
  `saved_path`.
- The agreed “omit encoding, retain readable references” rule requires an explicit
  supported reference-format boundary. Do not infer a real local file from an
  arbitrary string or from the current working directory.
- No image I/O is required by the design. Reference text, when established by its
  recorded format, is shown as text only. Do not use current placeholder comments
  as proof that every string is a readable reference.

## Remaining tools and feasibility

| Tool/group | Assessment |
| --- | --- |
| `clock.curr_time`, `get_context_remaining` | Existing recorded text can be rendered directly; examples now match integer formatting. |
| `web.run` | `SearchCommands` covers all ten operation groups; no output/schema change required. All sports/finance/weather qualifiers must survive title construction. |
| `apply_patch` | Existing presentation includes file operations, chunk context, workdir/environment and errors. Remove Changes folding, retain paging. Production presentation does not call the unused `compactPatchContext` helper. |
| `update_plan`, `request_user_input`, `suggest_subagent_task` | Keep unchanged; do not reopen their field/layout decisions under generic rules. |
| `hosted_web_search` | Existing action fields suffice; no separate output exists. Null/Other actions cannot produce invented URLs, queries, or result bodies. |
| Invalid built-in calls | Original strings/structured search arguments and message are retained. Dispatch only positively identified built-ins to the specialized invalid layout; unknown/external names keep existing fallback. |
| External/MCP | No redesign required. Discovery of external definitions is distinct from executing an external tool. |

## Implementation checks to carry forward

- Narrow terminal and over-240-character command/path/query; multiline and
  whitespace-only meaningful input; no inaccessible omitted values.
- Combined Web operation types, multiple entries per type, every optional
  qualifier, and null action/empty result cases.
- Same completed command row before/after process completion, registry removal,
  another poll and history reload; unchanged recorded content and stable title.
- Commands with/without exit code, non-default output limit, original token
  count and tool-truncated output.
- Image success with path, success without path, failure, non-default detail,
  and typed/unsupported hosted reference formats; no UI filesystem/network I/O.
- Tool-search namespace/member definitions, hosted arbitrary arguments,
  independent second disclosure and empty results.
- Patch over 200 lines, pagination continuity, all metadata, failure, move and
  chunk context; unchanged mature/external displays.

## Validation performed

- Read all eleven tool-specific checklists and real current source paths.
- Cross-checked the fixed-tool registry and relevant persisted models.
- Reviewed changed documents against confirmed requirements; syntax/whitespace
  and local-link checks are recorded with this review handoff.
- No claim of runtime rendering, terminal-width or lifecycle test success.

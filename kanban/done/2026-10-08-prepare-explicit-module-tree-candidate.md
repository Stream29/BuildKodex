# PREPARATION COMPLETE

# Task Tree

- `Prepare the explicit settings-local DSL against the accepted project graph`()
- `Compare explicit declarations with the current static build-script inventory`()
- `Hand off the exact candidate and project IDs to the coordinator`()

Preparation is complete. Integration, runtime checks and independent acceptance
belong to the parent task after its SDK source-root gate, not this worker.

# Details

- Parent: [Gradle development experience](../executable/2026-10-02-rescue-gradle-development-experience.md).
- This is isolated preparation, not integration before the SDK gate. Own only
  candidate `file:///tmp/kodex-module-tree-candidate-20261008/settings.gradle.kts`
  and this handoff. Current main binary baseline is `c0ba86cc`; coordinator alone
  integrates, builds, reviews, commits and pushes.
- Follow the approved DSL exactly: top-level `moduleTree(name)` always includes
  its root, even if a namespace without build script. Nested scope's same-name
  overload includes itself only if it is a real build-script project; otherwise
  a physical namespace. `module(name)` is a leaf. Explicitly declare all nodes;
  no `walkTopDown` or inferred child discovery during production settings.
- Preserve flat IDs, directories, all 203 non-root build-script projects, 12
  original namespace roots and implicit root: 216 registered projects.
  Source/target declarations, repositories, versions and kRPC remain untouched.
- Minimal settings-local path scope/DSL only; no plugin, project registry,
  provider/facade or second configuration API. Do not claim Sync speedup from
  maintainability; actual equivalent model and measurements gate adoption.
- No worker build/tests/IDE/network mutation/Git/device use; user is gaming on
  local. Preparation does not start profile, build-logic or compatibility migration.

## Exact handoff

- First prepared candidate: [settings:1–534](file:///tmp/kodex-module-tree-candidate-20261008/first-settings.gradle.kts#L1).
  SHA256 `01abc722070176acdc19b2bed95e5503c41b43de481c4739daea550be3cda24b`.
- Accepted input: [production settings:1–77](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/settings.gradle.kts#L1),
  SHA256 `78fbc45a63250ce92cbce9d87fc21f0c1e5031248c60a3b0d8335000db0ed074`.
  Rechecked unchanged after candidate preparation; `c0ba86cc` identity is the
  supplied accepted baseline, not a new Git verification by this worker.
- Design: [parent decisions:261–279](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/executable/2026-10-02-rescue-gradle-development-experience.md#L261).
  Only discovery functions/calls are replaced; adding the explicit `Settings`
  import is the sole prefix difference. Original first 47 lines compare
  byte-identical after removing that import from the candidate.
- Only this handoff and the isolated candidate were written, using `apply_patch`.
  No production source, SDK, other document or generated manifest was changed.
  No build, tests, Gradle, IDE, device, process control, network writes or Git.
  Candidate is intentionally retained for coordinator review; no extra temporary
  files or persistent resources were created.

## Node and leaf semantics

- Settings-local `ModuleTreeScope` holds `Settings` and path segments only.
  Top-level `moduleTree(String, block)` always registers its root, including
  the 12 existing namespace roots. `:` remains the implicit Kodex root.
- Nested member `moduleTree(String, block)` registers itself only when its
  explicit directory has `build.gradle.kts`; otherwise it is a physical
  namespace. Children are only the declarations in its body.
- Member `module(String)` requires an actual `build.gradle.kts` and registers
  one leaf. A node with its own build script and declared children uses one
  `moduleTree`, not a duplicate same-name leaf.
- Explicit paths resolve under `Settings.settingsDir`, the current original
  settings-file base. Flat IDs join segments with `-`, preserving directories.
  Names must be lower-case alphanumeric segments with single interior hyphens;
  empty names, `/`, `\`, `:`, `.` and `..` cannot redirect the path.
- Registration checks `Settings.findProject` before `include`/`projectDir`
  assignment. Repeated registered physical paths and colliding flat IDs fail
  before any descriptor override.
- Guard boundary: repeated *unregistered pure namespace* declarations are not
  recorded by `findProject`; duplicate namespace declarations with disjoint
  children are not independently rejected at runtime. This static body has no
  repeated physical declaration or flat project-ID collision. No additional
  namespace registry/global graph manager was introduced; coordinator review
  must explicitly consider this boundary against the parent duplicate rule.
- Production reads directory existence and build-script presence only for the
  explicit nodes/leaves; no recursive discovery, source-tree probing, plugin,
  provider wrapper, KMP configuration probe or alternative configuration API.

## Static inventory and model boundaries

- Intermediate coordinator delta: candidate SHA256 was
  `95e2e4961b8922e79b3145b5e4cc8e170d5384b05c36fcbea11928cca7bbe46c`.
  The original `01abc…` preparation remains separately frozen above.
  One settings-local `MutableSet<String>` records declared physical paths and
  is passed through the existing scope; it rejects duplicate pure namespaces
  before registration. `findProject` still guards distinct physical paths that
  collide in flat project IDs. This closes the parent's duplicate-path rule
  without a registry plugin, second module graph or recursive discovery.
  This was not the final Kotlin settings language implementation.

- Subsequent user-approved root `Settings.moduleTree` extension resolves the
  actual script-member shadowing failure without changing call syntax.
  Final `3bbb370af24830df1a1fb935a9fbb6f09a88a410b089a1a1d090d7b3ac11fe48`
  has exact actual-model equality, eight negative cases, representative tasks
  and independent review completed in the
  [integration task](2026-10-08-integrate-explicit-module-tree.md).
  This preparation child is complete; integration/commit and GUI measurement
  remain separate owners, not evidence claimed by the preparing worker.

- Read-only shell file enumeration and textual parsing only; no candidate
  evaluation, Kotlin compilation, Python tests or other runtime tests.
- Enumerated only the original 13 root trees, excluding `build` output.
  External builds and `buildSrc` are outside those trees. Compared the sorted
  real paths from the original discovery predicate with explicit declaration
  paths having actual `build.gradle.kts`: **no added or missing path**.
- All 318 explicit physical nodes exist: 202 real leaves and 116 tree nodes.
  Trees comprise 13 roots and 103 nested namespaces. The only current real
  tree node is `integration-test`; it is one of the 203 non-root real projects,
  not a thirteenth empty namespace project. There are currently no nested
  real build-script nodes with build-script descendants.
- Registered inventory is 203 real projects + 12 namespace roots + implicit
  root = **216 unique IDs**, listed below. Static ID/directory mismatch against
  the current model rule: **none found**. No comparison to the obsolete
  142-project research graph is used.
- Repository configuration for all three fork packages, canonical catalog, root name,
  source/target/build scripts and kRPC patch are untouched. Explicit physical
  hierarchy includes `spec`, `impl`, component namespaces and decorator trees;
  none of those pure nested namespaces becomes an extra project.
- This is not an actual Gradle model PASS. Coordinator owns SDK source-root
  admission, independent exact-candidate review, and comparison of actual
  216-project model, source hierarchy, tasks/targets, plugin `CodeSource` and
  resolved dependency closure. Integrate only after the SDK source gate.
  No Sync/navigation/incremental behavior or performance benefit is claimed.
- Intentional behavior change outside the fixed current inventory: newly added
  or undeclared physical build scripts will not be auto-included; missing
  explicit directories/leaf build scripts fail rather than silently disappear.

## Exact expected registered project IDs

Sorted IDs below include implicit `:`; project directories are the explicit
segment hierarchy in the candidate, not inferred by splitting the flat IDs.
The handoff list was textually compared with the candidate-derived static
inventory: exact match, 216 entries, no duplicate IDs.

```text
:
:agent-context
:agent-context-impl-agents-md-filesystem
:agent-context-impl-prefix
:agent-context-impl-prefix-render
:agent-context-impl-skill-filesystem
:agent-context-impl-skill-render
:agent-context-spec-agents-md
:agent-context-spec-available-skill
:agent-context-spec-contract
:agent-context-spec-prefix
:agent-context-spec-prompt-dsl
:agent-context-spec-skill
:agent-runtime
:agent-runtime-impl-composition
:agent-runtime-impl-decorator-compact
:agent-runtime-impl-decorator-steer
:agent-runtime-impl-decorator-tool
:agent-runtime-spec-contract
:agent-runtime-spec-decorator-compact
:agent-runtime-spec-decorator-steer
:agent-runtime-spec-decorator-tool
:agent-session
:agent-session-impl-filesystem
:agent-session-impl-in-memory
:agent-session-spec-contract
:agent-session-test
:agent-state
:agent-state-impl-state
:agent-state-spec-context-window
:agent-state-spec-contract
:agent-state-test
:agent-storage
:agent-storage-impl-filesystem
:agent-storage-impl-filesystem-layout
:agent-storage-impl-in-memory
:agent-storage-spec-clean-models
:agent-storage-spec-contract
:agent-storage-spec-contract-ext
:app
:app-cli
:app-component-account-usage-impl-view
:app-component-account-usage-impl-viewmodel
:app-component-account-usage-spec
:app-component-agent-impl-view
:app-component-agent-spec
:app-component-application-preferences-impl-view
:app-component-application-preferences-impl-viewmodel
:app-component-application-preferences-spec
:app-component-authentication-settings-impl-view
:app-component-authentication-settings-impl-viewmodel
:app-component-authentication-settings-spec
:app-component-composer-impl-view
:app-component-composer-impl-viewmodel
:app-component-composer-spec
:app-component-context-source-settings-impl-view
:app-component-context-source-settings-impl-viewmodel
:app-component-context-source-settings-spec
:app-component-history-impl-view
:app-component-history-impl-viewmodel
:app-component-history-index-impl-view
:app-component-history-index-impl-viewmodel
:app-component-history-index-spec
:app-component-history-spec
:app-component-hook-settings-impl-view
:app-component-hook-settings-impl-viewmodel
:app-component-hook-settings-spec
:app-component-mcp-settings-impl-view
:app-component-mcp-settings-impl-viewmodel
:app-component-mcp-settings-spec
:app-component-new-session-defaults-impl-view
:app-component-new-session-defaults-impl-viewmodel
:app-component-new-session-defaults-spec
:app-component-new-session-impl-view
:app-component-new-session-impl-viewmodel
:app-component-new-session-spec
:app-component-openai-login-impl-view
:app-component-openai-login-impl-viewmodel
:app-component-openai-login-spec
:app-component-path-picker-impl-view
:app-component-path-picker-impl-viewmodel
:app-component-path-picker-spec
:app-component-request-user-input-impl-view
:app-component-request-user-input-impl-viewmodel
:app-component-request-user-input-spec
:app-component-runtime-configuration-impl-view
:app-component-runtime-configuration-impl-viewmodel
:app-component-runtime-configuration-spec
:app-component-session-catalog-impl-view
:app-component-session-catalog-impl-viewmodel
:app-component-session-catalog-spec
:app-component-session-delete-impl-view
:app-component-session-delete-impl-viewmodel
:app-component-session-delete-spec
:app-component-session-rename-impl-view
:app-component-session-rename-impl-viewmodel
:app-component-session-rename-spec
:app-component-session-settings-impl-view
:app-component-session-settings-impl-viewmodel
:app-component-session-settings-spec
:app-component-session-sidebar-impl-view
:app-component-session-sidebar-spec
:app-component-session-tab-bar-impl-view
:app-component-session-tab-bar-spec
:app-component-session-title-settings-impl-view
:app-component-session-title-settings-impl-viewmodel
:app-component-session-title-settings-spec
:app-component-settings-impl-view
:app-component-settings-impl-viewmodel
:app-component-settings-spec
:app-component-suggest-subagent-task-impl-view
:app-component-suggest-subagent-task-impl-viewmodel
:app-component-suggest-subagent-task-spec
:app-component-usage-reset-impl-view
:app-component-usage-reset-impl-viewmodel
:app-component-usage-reset-spec
:app-component-working-directory-impl-view
:app-component-working-directory-impl-viewmodel
:app-component-working-directory-spec
:app-impl-application
:app-impl-rpc
:app-impl-session
:app-impl-view
:app-migration-impl
:app-migration-spec
:app-settings-impl-filesystem
:app-settings-spec-persistence
:app-spec-application
:app-spec-session
:app-test-support-rpc
:app-view-components
:hook
:hook-impl-notification
:hook-spec-notification
:integration-test
:mcp
:mcp-impl-composition
:mcp-spec-contract
:mcp-spec-stdio
:mcp-spec-streamable-http
:openai
:openai-impl-account-usage
:openai-impl-client
:openai-impl-client-test
:openai-impl-codex-cli-storage
:openai-impl-model-catalog
:openai-spec-account-usage
:openai-spec-client
:openai-spec-client-test
:openai-spec-codex-cli-storage
:openai-spec-json-codec
:openai-spec-model-catalog
:openai-spec-models
:rpc
:rpc-impl-client
:rpc-impl-in-memory
:rpc-impl-krpc-utils-patch
:rpc-impl-server
:rpc-spec-contract
:rpc-spec-models
:tool
:tool-impl-apply-patch
:tool-impl-builder
:tool-impl-current-time
:tool-impl-get-context-remaining
:tool-impl-image-generation
:tool-impl-multi-agent
:tool-impl-plan
:tool-impl-request-user-input
:tool-impl-tool-search
:tool-impl-unified-exec
:tool-impl-view-image
:tool-impl-web-run
:tool-spec-apply-patch
:tool-spec-builder
:tool-spec-contract
:tool-spec-current-time
:tool-spec-get-context-remaining
:tool-spec-image-generation
:tool-spec-multi-agent
:tool-spec-plan
:tool-spec-request-user-input
:tool-spec-tool-search
:tool-spec-unified-exec
:tool-spec-view-image
:tool-spec-web-run
:utils
:utils-coroutines-spec
:utils-external-url-impl
:utils-external-url-spec
:utils-filesystem-lease-impl
:utils-filesystem-lease-spec
:utils-images-codec-impl
:utils-images-codec-spec
:utils-images-spec
:utils-kodex-home-spec
:utils-kotlinx-io-coroutines-impl
:utils-kotlinx-io-coroutines-spec
:utils-kotlinx-io-serialization-spec
:utils-ktor-client-ext-impl
:utils-ktor-client-ext-spec
:utils-logging-impl
:utils-logging-spec
:utils-os-environment-spec
:utils-patch-impl
:utils-patch-spec
:utils-process-client-impl
:utils-process-client-spec
:utils-read-write-mutex-impl
:utils-read-write-mutex-spec
:utils-rpc-exception-spec
:utils-search-index-impl
:utils-search-index-spec
:utils-shell-client-impl
:utils-shell-client-spec
:utils-terminal-text-spec
```

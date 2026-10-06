# Task Tree

- `Trace exact UI implementations and production consumers`()
- `Move LazyList into existing components and Patch into History View`()
- `Preserve original rendering, algorithms and applicable tests`()
- `Publish source and dependency handoff`()
- `Receive independent scoped acceptance with the parent batch`()

# Details

- [主任务](2026-10-07-cutover-ui-foundations-and-home-bootstrap.md)；用户已授权实施。
- Own `app/contract/lazy-list/src` → `app/view/components/src` 的所有真实代码/tests。
  同路径冲突先报告，不复制新公共状态，不造 LazyList VM/provider。
- Own `app/view/patch/src` → `app/component/history/impl/view/src` 的 patch package
  与 tests。纯 LineDiff/Presentation 保留 commonMain，renderer 在 mosaicMain。
  保留原包名、操作、200-line paging、expansion、失败/完成/elapsed UI。
- 唯一 production consumer 是 History，性能 fixture 由协调者换项目坐标；
  禁止另建 Patch component/spec/controller 或依赖旧 View wrapper。
- 主协调者拥有全部 Gradle 及旧 build 删除；不碰 History 主 renderer/VM/spec、
  Root/Agent、外层公共规范与其他工作线文件。
- 完整保留原测试并补实际 UI 输入边界如有缺口，禁止占位输出或删除原性能不变量。
- 不运行 Gradle/IDE、提交、推送或切分支；用 apply_patch 修改内容。
  本子任务 handoff 给出迁移路径、实际 helper/FQCN 和新增依赖需求。

## Integration handoff

### Coordinator closure

- Final components JVM 165 cases and History View 71 cases passed; the original
  migrated source/test bytes and 72 scoped cases remain intact. Coordinator added
  two standalone remount/focus ablation cases, not a new controller or VM.
- Explicitly enabled Integration Patch probe passed with one repetition after
  matching its release/layout-frame fixture to the existing Patch renderer suites;
  all original projection/wrapping/row-count assertions remain.
- Retired build entries and all production/test consumers now directly use the
  existing components/History projects. Native compilation and current CLI passed.
  Full evidence/limits: [parent](2026-10-07-cutover-ui-foundations-and-home-bootstrap.md).
  Independent scoped acceptance passed under conditional commit authorization;
  components/History View and the explicitly enabled probe were freshly rerun.
  Local commits only.

- Source handoff **READY**: all 23 source/test files moved and originals deleted.
  Compared each destination byte-for-byte with baseline
  `8297d38873e654ce2fce3c78bde905db6f491017` on shared `refactor/spec`;
  zero collisions, same source sets/FQCN/API/KDoc/algorithms.
  This was a source-only worker handoff; centralized results supersede that status above.
- LazyList: 17 files moved from `app/contract/lazy-list/src` to
  `app/view/components/src`, with source sets and
  `io.github.stream29.kodex.cli.components` unchanged.
- Patch: six files moved from `app/view/patch/src` to
  `app/component/history/impl/view/src`, with source sets and
  `io.github.stream29.kodex.cli.patch` unchanged.
- No Gradle, IDE, host, other History, Home, resource, commit or branch operation
  belongs to this lane.

### Main-lane dependency work

- `app/view/components`: move/add the old LazyList
  `commonMain` dependencies `api(libs.mosaic.runtime)` and
  `api(libs.kotlinx.coroutines.core)`; existing `mosaicMain` declarations
  cannot satisfy imports in newly populated `commonMain`.
  Remove `api(project(":app-contract-lazy-list"))` from `mosaicMain`.
  Existing `mosaicTest` already supplies `libs.mosaic.testing`.
- `app/component/history/impl/view`: remove its `mosaicMain`
  `implementation(project(":app-view-patch"))`; add
  `implementation(project(":utils-terminal-text-spec"))` there for
  `takeFirstFittingTerminalWidth` and `terminalCellWidth`.
  Existing shared components/runtime/testing dependencies cover the rest.
- Patch `commonMain` public signatures need clean models and
  `:utils-patch-spec`. They are already transitively exported by existing
  `api(project(":app-component-history-spec"))` →
  `api(project(":agent-storage-spec-clean-models"))` →
  `api(project(":utils-patch-spec"))`.
  Preserve that path; no `:utils-patch-impl` or new wrapper dependency is needed.
- `integration-test/build.gradle.kts:51`: replace
  `implementation(project(":app-view-patch"))` with
  `implementation(project(":app-component-history-impl-view"))`.
  `PatchRendererPerformanceProbeTest.kt` imports remain valid unchanged.
- `CleanEventView.kt` is the sole production caller of
  `PendingPatchToolEventView` and `StablePatchToolEventView`; same FQCN makes
  it a local same-project consumer without a source edit.
- Remove the old LazyList dependency from History spec as its separate lane
  removes renderer types. Retarget Session Tab Bar's
  `implementation(project(":app-contract-lazy-list"))` to components
  (deduplicate if already present).
- Main owns deletion of both old build files and root/discovered project entries.

### Preserved API and helpers

- Components retain `LazyListScopeMarker`, `LazyListScope`, `items`,
  `LazyListItemInfo`, `LazyListLayoutInfo`, `LazyListState`,
  `rememberLazyListState`, `LazyColumn`, `ScrollableState` and its factory,
  `rememberScrollableState`, `ScrollOrientation`, `ScrollInputSource`,
  `ScrollInteraction`, `ScrollInteractionSource`, `MutableScrollInteractionSource`,
  `ScrollState`, `rememberScrollState`, `scrollable`, `scrollablePaging`,
  `horizontalScroll`, and `verticalScroll`.
- Patch retains `PatchPresentation`, `PatchPresentationAction`,
  `PatchPresentationTarget` (including `SingleFile`/`FileCount`),
  `PatchPresentationStatus`, `PatchPresentationLine`, `PatchPresentationLineKind`,
  `toPendingPatchPresentation`, `toStablePatchPresentation`,
  `PendingPatchToolEventView`, and `StablePatchToolEventView`.
- Existing `cli.history` read-only typography/rows and `cli.settings`
  `SettingsStyles.kt` remain single implementations in components. History/Index
  and multiple Settings renderers use them; none are copied or cut into a View cycle.
- No resource files exist under either retiring `src` tree. Existing Docs and
  Integration recordings/fixtures remain outside this lane and unchanged by it.

### Full moved-file manifest

- LazyList old prefix: `Kodex/app/contract/lazy-list/src/`.
- LazyList new prefix: `Kodex/app/view/components/src/`.
- Each row below keeps the exact suffix
  `<sourceSet>/kotlin/io/github/stream29/kodex/cli/components/<filename>`.

| Source set | Filename |
| --- | --- |
| `commonMain` | `LazyListScope.kt` |
| `commonMain` | `LazyListState.kt` |
| `commonMain` | `ScrollableState.kt` |
| `mosaicMain` | `HorizontalScroll.kt` |
| `mosaicMain` | `LazyColumn.kt` |
| `mosaicMain` | `LazyColumnBeyondBoundsLayout.kt` |
| `mosaicMain` | `LazyColumnBringIntoViewModifier.kt` |
| `mosaicMain` | `LazyColumnPrefetch.kt` |
| `mosaicMain` | `LazyColumnRemeasurementModifier.kt` |
| `mosaicMain` | `Scrollable.kt` |
| `mosaicMain` | `ScrollablePaging.kt` |
| `mosaicMain` | `VerticalScroll.kt` |
| `mosaicMain` | `VerticalScrollBeyondBoundsLayout.kt` |
| `mosaicMain` | `VerticalScrollBringIntoViewModifier.kt` |
| `mosaicTest` | `HorizontalScrollTest.kt` |
| `mosaicTest` | `ScrollableStateTest.kt` |
| `mosaicTest` | `VerticalScrollTest.kt` |

- Patch old prefix: `Kodex/app/view/patch/src/`.
- Patch new prefix: `Kodex/app/component/history/impl/view/src/`.
- Each row below keeps the exact suffix
  `<sourceSet>/kotlin/io/github/stream29/kodex/cli/patch/<filename>`.

| Source set | Filename |
| --- | --- |
| `commonMain` | `LineDiff.kt` |
| `commonMain` | `PatchPresentation.kt` |
| `commonTest` | `LineDiffTest.kt` |
| `commonTest` | `PatchPresentationTest.kt` |
| `mosaicMain` | `PatchToolEventView.kt` |
| `mosaicTest` | `PatchToolEventViewTest.kt` |

### Verification and central test readiness

- Passed static checks: all 23 destination files byte-identical to baseline
  (4,372 lines); all 23 original paths absent; both old `src` trees contain
  no files; scoped `git diff --check` clean.
- All pre-existing components source/tests byte-identical to baseline,
  including `LazyColumnTest.kt`. No helper or test invariant was ablated.
- Components focused suites are ready: `HorizontalScrollTest` (3),
  `ScrollableStateTest` (11), `VerticalScrollTest` (7), existing
  `LazyColumnTest` (35). Original wheel bursts, keys/anchors, variable measure,
  predictive premeasure/reuse, bounded composition, focus search/relocation,
  pointer capture and paging assertions remain.
- History View Patch suites are ready: `LineDiffTest` (4),
  `PatchPresentationTest` (5), `PatchToolEventViewTest` (7).
  All 72 cases across these seven suites are retained, not executed here.
  Patch tests retain real mouse expansion, narrow/wide-grapheme output,
  ANSI status, elapsed header, structured success/failure diffs, 200-line
  pages and the at-most-five-Text-node invariant.
- Production files are unchanged apart from their paths: scroll consumption,
  key/interval mapping, measure, beyond-bounds, focus, prefetch cancellation,
  complete patch data and renderer paging/expansion/error/elapsed behavior
  are therefore preserved at source level. Runtime confirmation is central.
- No supplemental tests were added: original renderer-input tests already
  exercise the migrated operations; this lane changes no algorithm or business wire.
- Central validation still needs affected JVM suites, Integration perf probe,
  Native main compilation, dependency graph/old entry retirement and actual CLI.
  Coordinator Gradle/host edits visible in the shared worktree are not lane
  changes and have not been validated by this lane.
- No temporary files, background jobs, PTYs or shared compute/device resources
  were created; no cleanup operation or build was run.

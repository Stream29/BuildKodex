REVIEW READY

# Task Tree

- `Verify the pinned source baseline and frozen paths`()
- `Trace actual UI helpers, moved Patch and Home startup resource contracts`()
- `Review scope exceptions, consumers and lifetime evidence`()
- `Publish the independently evidenced report`()

# Details

## Baseline, authorization and checkpoints

- Independent read-only reacceptance of `Kodex/refactor/spec`, HEAD
  `6b7129fa0e719dde8f88a0034b9d572679984a03`; comparison baseline `8297d388`
  (`test: cover frontend exact targets and child ownership`).
- Inner `git status --porcelain=v1` was empty at entry and after inspection;
  HEAD remained pinned. Outer repository already contains unrelated changes and
  concurrent reports. This lane writes only this child report with `apply_patch`.
- Loaded [parent scope/verdict rules](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-07-final-reaccept-spec-impl-repository.md#L1),
  Draft, applicable skills and frontend, spec/impl, Home and coroutine-lifecycle
  checklists. Did not inherit coordinator/previous reviewer “pass”.
- Checkpoint 1: actual three build projects and Home spec/factory identified.
- Checkpoint 2: 17 LazyList/scroll and six Patch source/test blobs verified at new
  paths; all 16 frozen Home paths/blobs and application version unchanged.
- Final checkpoint: actual production consumers and test assertions traced;
  two confirmed baseline lease failures remain unfixed. Ready for asynchronous
  coordinator reading, not a request to execute the pending repair.
- No build, test execution, CLI, actual filesystem fixture, IDE, network, device,
  commit/push/switch or shared-resource operation. No temporary files created.

## Project coverage and verdict

| Actual project / counted scope | Grade | Contract → implementation → production consumer → test evidence |
| --- | --- | --- |
| `:app-view-components` / **1** | **D** — justified shared UI infrastructure, not missing MVVM | `io.github.stream29.kodex.cli.components.{LazyListScope,LazyListState,ScrollableState,ScrollState}` → actual `LazyColumn` measure/subcompose/input and eager scroll modifiers → History, Index, picker, catalog, sidebar, Settings, tab bar → actual Mosaic geometry/input/remount suites; [build](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/view/components/build.gradle.kts#L1), [algorithm](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/view/components/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/components/LazyColumn.kt#L114), [History caller](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/history/AgentHistoryView.kt#L202), [test](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/view/components/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/components/LazyColumnTest.kt#L33). |
| `:app-migration-spec` / **1** | **R** — real declarations and resource identity taken over | `io.github.stream29.kodex.app.migration.{MigrationVersion,Migration,KodexHomeVersionException,KodexHomeLayoutException,KodexHomeHandle}` → directly implemented retained lease, original factory returns interface → CLI/Application consume same FQCN → original version suite and actual-factory lifecycle tests; [spec build](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/migration/spec/build.gradle.kts#L1), [handle](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/migration/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/migration/KodexHomeHandle.kt#L6), [factory test](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/migration/impl/src/commonTest/kotlin/io/github/stream29/kodex/app/migration/KodexHomeHandleLifecycleTest.kt#L35). R describes takeover/documentation, not resolution of the B1 resource defects below. |
| `:app-migration-impl` / **1** | **B1** — confirmed baseline resource failures, cross-lane deduplicated | `CoroutineScope.prepareKodexHome` → `HeldLeaseKodexHomeHandle : KodexHomeHandle`, generated current version and frozen registry → CLI prepare before backend, `withKodexApplication` releases last → generic startup tests and real-FS fault-injection test source. [Factory/held resource](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/migration/impl/src/commonMain/kotlin/io/github/stream29/kodex/app/migration/KodexHomeMigration.kt#L20), [orphan test](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/migration/impl/src/commonTest/kotlin/io/github/stream29/kodex/app/migration/KodexHomeHandleLifecycleTest.kt#L270), [cleanup test](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/migration/impl/src/commonTest/kotlin/io/github/stream29/kodex/app/migration/KodexHomeHandleLifecycleTest.kt#L232). |
| Dedicated `cli.patch` in `:app-component-history-impl-view` / **0 additional projects** | **R** — real consumer-local cutover | `io.github.stream29.kodex.cli.patch.{PatchPresentation,PatchPresentationLine,…}` and `diffPatchLines` → pending/stable renderer → clean-event dispatch and History item expansion → common diff/presentation and Mosaic click/paging tests. [Build](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/impl/view/build.gradle.kts#L7), [consumer](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/history/CleanEventView.kt#L136), [200-line rendering test](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/impl/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/patch/PatchToolEventViewTest.kt#L203). Frontend lane owns/counts the full History View project. |

- Inventory derived from tracked build files, not directory names or historical
  project counts. Components has 60 tracked source/test files; critical-path review
  covers scrolling, framework boundary, shared visual helpers and their consumers.
- `D` is the explicit renderer-infrastructure exception in
  [frontend boundary](file:///home/stream/ACodeSpace/push/BuildKodex/checklist/frontend-application-boundary.md#L39);
  do not manufacture a LazyList/Patch VM or empty spec to pair folders.
- No independent new B1 root, B2 behavioral suspicion or unassigned project in this
  lane. Runtime/platform gaps below remain; R is static critical-path review only.

## UI infrastructure: original algorithm really moved

- Compared full Git blobs using `git ls-tree` and `git rev-parse <ref>:<path>`.
  All **17** `8297d388:app/contract/lazy-list/src/**` files have identical blobs
  under `HEAD:app/view/components/src/**` with the same suffix and package.
  This is real code/test movement, not a forwarded old implementation.
- Exact moved inventory:
  - `commonMain`: `LazyListScope.kt`, `LazyListState.kt`, `ScrollableState.kt`.
  - `mosaicMain`: `LazyColumn.kt`, `LazyColumnBeyondBoundsLayout.kt`,
    `LazyColumnBringIntoViewModifier.kt`, `LazyColumnPrefetch.kt`,
    `LazyColumnRemeasurementModifier.kt`, `Scrollable.kt`, `ScrollablePaging.kt`,
    `VerticalScroll.kt`, `VerticalScrollBeyondBoundsLayout.kt`,
    `VerticalScrollBringIntoViewModifier.kt`, `HorizontalScroll.kt`.
  - `mosaicTest`: `ScrollableStateTest.kt`, `VerticalScrollTest.kt`,
    `HorizontalScrollTest.kt`; the existing components `LazyColumnTest.kt`
    remains, plus new `LazyColumnRemountTest.kt`.
- HEAD has no tracked file under old `app/contract/lazy-list`, `app/view/patch`
  or `app/migration/contract`. Tracked `.kt`/`.kts` searches have no old
  `app-contract-lazy-list`, `app-view-patch`, `app-migration-contract` references.
  [Settings discovery](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/settings.gradle.kts#L34)
  derives coordinates from actual build directories, excluding `build`.
- `LazyListScope` stores intervals, bounds total count, resolves logical/reverse
  indexes, caches nearby keys and binary-searches intervals;
  [real provider](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/view/components/src/commonMain/kotlin/io/github/stream29/kodex/cli/components/LazyListScope.kt#L106).
- `LazyListState` implements signed scrolling with a measured-window fast path,
  force-remeasure fallback, pending anchor intent, stable-key restoration and
  attach/detach of the actual renderer's remeasurement;
  [scroll](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/view/components/src/commonMain/kotlin/io/github/stream29/kodex/cli/components/LazyListState.kt#L91),
  [attachment/anchor](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/view/components/src/commonMain/kotlin/io/github/stream29/kodex/cli/components/LazyListState.kt#L171).
- `LazyColumn` genuinely subcomposes variable-height rows by stable keys, rejects
  unbounded height, measures overscan/tail fill, clamps consumption, publishes real
  visible geometry and schedules predictive rows. Its prefetch uses cancellable
  stages and disposes handles on plan replacement/unmount;
  [measure](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/view/components/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/components/LazyColumn.kt#L122),
  [geometry/prefetch](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/view/components/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/components/LazyColumn.kt#L307),
  [prefetch lifetime](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/view/components/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/components/LazyColumnPrefetch.kt#L104).
- `ScrollState` clamps with widened arithmetic; `ScrollableState(...)` directly
  constructs `DefaultScrollableState`, validates consumed direction/magnitude
  and resets transient progress in `finally`. Wheel/paging publish only committed
  nonzero interactions and otherwise bubble;
  [state/factory](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/view/components/src/commonMain/kotlin/io/github/stream29/kodex/cli/components/ScrollableState.kt#L84),
  [wheel](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/view/components/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/components/Scrollable.kt#L17),
  [paging](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/view/components/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/components/ScrollablePaging.kt#L17).
- Common source's Compose runtime annotations/state are legitimate widget machinery
  in a **view project**, not leakage into a component spec/VM. Mosaic layout,
  terminal events and modifiers are in `mosaicMain`.
  [View convention](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/buildSrc/src/main/kotlin/kodex.kmp-view.gradle.kts#L1)
  applies CLI/Compose and centralized
  [JVM + four Native Mosaic hierarchy](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/buildSrc/src/main/kotlin/KodexHostKmp.kt#L36).
  There is no new fake framework-free VM/controller.
- Shared theme/pressable/popup/input/terminal text and read-only History/Index
  typography remain visual infrastructure. Read-only request-user-input/suggestion
  row projection does not own pending submission state;
  [shared rows](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/view/components/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/history/RequestUserInputHistoryView.kt#L17),
  [shared text](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/view/components/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/history/ReadOnlyHistoryText.kt#L20).

### Test reach and minimal ablation

- Original Mosaic suites use the actual renderer, not only DTO calls:
  variable-height partial rows, prepend/reorder keys, reverse layout, shrink/resize,
  same-frame wheel burst, predictive reuse, focus relocation and bounded composition.
  [Geometry suite](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/view/components/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/components/LazyColumnTest.kt#L33),
  [prefetch assertions](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/view/components/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/components/LazyColumnTest.kt#L522),
  [input assertions](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/view/components/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/components/ScrollableStateTest.kt#L64).
- New standalone remount tests explicitly distinguish focus-induced relocation
  from preservation of a partial reverse-list item with retained `LazyListState`;
  [test](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/view/components/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/components/LazyColumnRemountTest.kt#L17).
  They are **not** full History VM/host acceptance. Frontend lane owns that review.
- Ablation: removing old projects is safe because their real algorithms/tests are
  now here and consumers depend directly on components. Removing `LazyListState`
  or the measure/prefetch/input bridge would remove actual rendering responsibilities;
  retain them, do not replace them with a formal spec wrapper.

## Patch: actual History consumer and complete presentation

- All **six** original `app/view/patch/src/**` blobs match at
  `app/component/history/impl/view/src/**` (same suffix/FQCN):
  `LineDiff.kt`, `PatchPresentation.kt`, two common tests,
  `PatchToolEventView.kt`, `PatchToolEventViewTest.kt`.
- `Patch.toPendingPatchPresentation()` /
  `StablePatchToolEvent.toStablePatchPresentation()` preserve structured add/delete/
  rename/update hunks, contexts, working directory/environment, raw tool name and
  failure text. They do not truncate the source presentation to 200 lines;
  [projection](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/impl/view/src/commonMain/kotlin/io/github/stream29/kodex/cli/patch/PatchPresentation.kt#L74).
- `diffPatchLines` uses prefix/suffix/patience anchors, bounded dynamic LCS and
  large-input removal/addition fallback. `compactPatchContext` is not called by
  this presentation path; don't infer content loss from that helper's existence;
  [algorithm](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/impl/view/src/commonMain/kotlin/io/github/stream29/kodex/cli/patch/LineDiff.kt#L60).
- `PendingPatchToolEventView` owns local renderer expansion; stable renderer accepts
  paired external expansion/toggle or local fallback. Real History dispatch passes
  the existing item's expansion binding and elapsed suffix, while pending dispatch
  directly passes `diff`;
  [renderer](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/patch/PatchToolEventView.kt#L34),
  [stable caller](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/history/CleanEventView.kt#L136),
  [pending caller](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/history/CleanEventView.kt#L192).
- Two expansion levels precede body display; `DefaultPatchLinePageSize = 200`,
  each Show More increments by 200 up to full presentation size. Body wraps only
  currently displayed lines into one styled Text at finite terminal width;
  [paging](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/patch/PatchToolEventView.kt#L170),
  [body](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/patch/PatchToolEventView.kt#L228).
- History View common source now directly depends on clean models and
  `:utils-patch-spec`; Mosaic source depends on components. No external patch-view
  project, controller, alias or forward layer remains. `utils-patch-spec` is the
  actual parsed Patch model, not a revived UI patch project.
- Tests distinguish pure edit-script/projection checks from real rendering:
  common tests cover repeated/reordered/large diffs and success/failure/hunks;
  Mosaic tests send actual pointer motion/press/release for expansion, colors,
  narrow wrapping and 2,000-line paging. The latter asserts 204 then 404 rendered
  rows, 1801 then 1601 remaining, and at most five Text nodes.
  [Pure tests](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/impl/view/src/commonTest/kotlin/io/github/stream29/kodex/cli/patch/LineDiffTest.kt#L49),
  [renderer/paging](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/impl/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/patch/PatchToolEventViewTest.kt#L203),
  [real dispatch test](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/impl/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/history/CleanEventViewTest.kt#L654).
- Limits: first two display pages are asserted, not all pages/Native terminal
  timing; full source presentation is still eagerly generated. This is not a
  constant-memory diff or full CLI end-to-end performance certification.
- Ablation: delete only the retired patch project (already done); retain the moved
  projection/diff/renderer, which carries real semantics and styles. Adding a
  Patch VM/spec would not preserve any missing responsibility.

## Home: real contract takeover and production ownership

- Original `MigrationVersion` and tests moved `contract → spec`, retaining parser,
  integer comparison and constructor validation; change adds meaningful KDoc,
  including constructor `@throws`, not a second version model;
  [version](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/migration/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/migration/MigrationVersion.kt#L3),
  [actual test](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/migration/spec/src/commonTest/kotlin/io/github/stream29/kodex/app/migration/MigrationVersionTest.kt#L8).
- Original ordinary `Migration(toVersion, suspend (Path, CoroutineFileSystem) -> Unit)`
  and original version/layout exception FQCNs now live uniquely in spec.
  `Migration` KDoc owns ordering, successful-version commit, failure/cancellation,
  no rollback/cursor and historical action reentry semantics;
  [entry](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/migration/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/migration/Migration.kt#L6),
  [exceptions](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/migration/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/migration/KodexHomeVersionException.kt#L3).
- Spec directly `api`s `:utils-kotlinx-io-coroutines-spec`, which declares
  [actual `CoroutineFileSystem`](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/kotlinx-io-coroutines/spec/src/commonMain/kotlin/io/github/stream29/kodex/utils/kotlinxiocoroutines/CoroutineFileSystem.kt#L22).
  Implementation adds filesystem impl privately for `SystemCoroutineFileSystem`;
  spec does not route through the concrete FS project.
- Sole `KodexHomeHandle : AutoCloseable` exposes home/version/close/closeAndJoin.
  Original concrete class was replaced with internal `HeldLeaseKodexHomeHandle`
  directly holding the original lease and implementing this interface. Factory
  returns it on both matching-version and post-upgrade paths. No factory interface,
  Home manager/preparer, version-provider, parallel handle or forward wrapper.
- Actual chain:
  [CLI prepare](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/cli/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/Main.kt#L12)
  → public `CoroutineScope.prepareKodexHome`
  → internal coordinator obtains initial read, exclusive write if required, final read
  → `HeldLeaseKodexHomeHandle`
  → [Application version/root use](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/application/src/commonMain/kotlin/io/github/stream29/kodex/cli/app/Application.kt#L55)
  → [last NonCancellable release](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/application/src/commonMain/kotlin/io/github/stream29/kodex/cli/app/Application.kt#L180).
  Prepare failure prevents opening backend/settings. Logging uses default Home;
  structured data uses supplied handle root.
- Current version stays generated:
  [Gradle `project.version` task/source registration](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/migration/impl/build.gradle.kts#L7)
  → [generated constant task](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/buildSrc/src/main/kotlin/GenerateKodexVersion.kt#L15)
  → `CurrentKodexApplicationVersion` parses that constant. Coordinates remain
  [0.4.8](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/buildSrc/src/main/kotlin/KodexHostKmp.kt#L8);
  generator/coordinates diff against `8297d388` is empty.
- Coordinator still validates registry, rejects downgrade, avoids Session scan on
  exact match, validates only unversioned layout, selects `stored < target <= current`,
  writes a target only after action success, writes no-op tail version and checks
  final retained-read version. No startup algorithm rewrite was hidden in the move;
  [actual coordinator](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/migration/impl/src/commonMain/kotlin/io/github/stream29/kodex/app/migration/KodexHomeMigration.kt#L75).

### KDoc / throwing behavior / actual test limits

- `KodexHomeHandle` documents snapshot version, explicit owner, shared-reference
  release, request-only close, cancellable wait and known lack of atomic handoff/
  cleanup-failure surfacing. This is substantive documentation of **current** behavior,
  not permission to consider lifecycle SOP satisfied.
- Named Home rejection conditions have `@throws` both on normative handle KDoc
  and the actual factory: version/layout, registry order, missing owner Job,
  cancellation. Version constructor throws is on its actual spec constructor;
  closeAndJoin cancellation is on its throwing spec operation.
- Actual generic tests cover new/unversioned Home, unknown/legacy preservation,
  invalid latest pointer, matching-version no-scan, ordered/future migration,
  completed-target retention on failure, newer/non-string version and unordered
  registry;
  [suite](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/migration/impl/src/commonTest/kotlin/io/github/stream29/kodex/app/migration/KodexHomeMigrationTest.kt#L18).
- Lifecycle tests use public factory/interface and delegated real isolated system
  FS, not fake Home handles: restart/idempotent close, shared references, owner
  cancellation, cancelled wait, action cancellation, read-failure cause,
  noncanonical/non-regular version and invalid unversioned layout;
  [suite](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/migration/impl/src/commonTest/kotlin/io/github/stream29/kodex/app/migration/KodexHomeHandleLifecycleTest.kt#L35).
- Do not turn KDoc into test evidence: generic/lifecycle suites do **not** directly
  exercise final-read disappeared/changed version, no-Job receiver, duplicate target
  specifically, migration-start callback failure, every metadata failure or
  successful action + cleanup failure. These are coverage limits, not demonstrated
  new bugs. Raw filesystem/lease/action failures remain propagated as documented;
  no guarantee that all I/O becomes a Home exception.

## B1 Home impacts — confirmed baseline-unfixed, count root in other-utils

### HOME-LEASE-HANDOFF: owner published before resource ownership is installed

- [Renewable acquire](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/filesystem-lease/impl/src/commonMain/kotlin/io/github/stream29/kodex/utils/filesystemlease/FileSystemLeaseImpl.kt#L90)
  writes the owner before constructing the structured renewable lease;
  [read path](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/filesystem-lease/impl/src/commonMain/kotlin/io/github/stream29/kodex/utils/filesystemlease/FileSystemReadLeaseImpl.kt#L56)
  reaches it under acquisition guard. Home's first acquire is before its local
  `try/finally`; there is no returned lease to close if publication is cancelled.
- [Actual Home characterization](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/migration/impl/src/commonTest/kotlin/io/github/stream29/kodex/app/migration/KodexHomeHandleLifecycleTest.kt#L270)
  writes actual `.read.lock` bytes then suspends/cancels, waits for operation and
  owner completion, and asserts **one orphan read owner remains** and guard is gone.
  Consequence: failed startup can leave a live-looking Home read owner until
  expiration/stale cleanup, obstructing exclusive Home preparation. No handle
  was handed to the caller to repair it.
- History: root lease and Home cleanup logic unchanged versus `8297d388`;
  new test in `6b7129fa` records an existing defect. Green characterization would
  mean the defect remains, not that cancellation safety passed.
- Minimal proposal, only if separately authorized: repair original lease publication/
  handoff with exact-owner cleanup and finite NonCancellable wait; retain original
  factory, lock format and ownership identity. Change characterization to desired
  “no orphan after owner completion” assertion and first verify failure.
  Do not add a Home manager or edit released migration paths.

### HOME-LEASE-CLEANUP: completed wait does not observe failed release

- [Held handle](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/migration/impl/src/commonMain/kotlin/io/github/stream29/kodex/app/migration/KodexHomeMigration.kt#L29)
  and initial/write/final failure cleanup use `close(); Job.join()`.
  The lease support extension does the same;
  [support](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/filesystem-lease/impl/src/commonMain/kotlin/io/github/stream29/kodex/utils/filesystemlease/FileSystemLeaseSupport.kt#L23).
  Waiting finishes without surfacing the background cleanup exception.
- [Actual Home failure test](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/migration/impl/src/commonTest/kotlin/io/github/stream29/kodex/app/migration/KodexHomeHandleLifecycleTest.kt#L232)
  injects primary action failure and write-owner delete failure, asserts original
  primary identity, observes cleanup only in background handler, asserts **empty
  suppressed exceptions**, old version and surviving write owner.
- Application's finalizer can attach a cleanup failure **only if**
  `homeHandle.closeAndJoin()` throws it. Its catch/suppressed structure does not
  solve the underlying unobserved background failure. Successful wait/owner
  completion is not proof owner file deletion succeeded.
- History/test limit: baseline-unfixed, not a migration regression. Existing
  ordinary close/owner-cancel tests prove release in the success case only.
  Characterization is not a primary+cleanup correctness test.
- Minimal proposal: expose the original lease's awaited cleanup result at its real
  boundary, then have original Home waits preserve primary and attach cleanup
  failure; successful action + failed cleanup must become observable. No second
  failure authority, wrapper release or parallel resource model.
- Cross-lane root ownership:
  [other-utils report](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-07-final-audit-other-utils.md#L1).
  These two Home impacts do **not** add two independent defects to the repository
  count. [Home/lease repair task](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-07-plan-home-lease-handoff-and-cleanup.md#L1);
  its proposals are not implemented at pinned HEAD.

## Frozen 16 paths/bytes: independently verified, not rewritten

- Selected every original `app/migration/impl` tracked `v*` source/test/fixture
  plus `KodexHomeMigrations.kt`: exactly **16** paths. Compared full old/new Git
  blob IDs at the **same** path; all equal, none moved/deleted. Also checked current
  inventory: no additional frozen path or replacement registry.
- Table lists abbreviated blob identifiers for reproducibility; full equality,
  not prefix comparison, was performed. Each link names the exact preserved file.

| Frozen file URI | Same blob (`8297d388` = pinned HEAD) |
| --- | --- |
| [Registry](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/migration/impl/src/commonMain/kotlin/io/github/stream29/kodex/app/migration/KodexHomeMigrations.kt#L9) | `ee60b8bc5a2a` |
| [v0_3_3 source](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/migration/impl/src/commonMain/kotlin/io/github/stream29/kodex/app/migration/v0_3_3/MigrateToV0_3_3.kt#L1) | `42fdda56a949` |
| [v0_3_5 frozen skill](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/migration/impl/src/commonMain/kotlin/io/github/stream29/kodex/app/migration/v0_3_5/KodexHomeSkill.kt#L1) | `90ed4069e693` |
| [v0_3_5 source](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/migration/impl/src/commonMain/kotlin/io/github/stream29/kodex/app/migration/v0_3_5/MigrateToV0_3_5.kt#L1) | `1df0eb0c2e3c` |
| [v0_4_3 frozen skill](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/migration/impl/src/commonMain/kotlin/io/github/stream29/kodex/app/migration/v0_4_3/KodexHomeSkill.kt#L1) | `6817c54f57ff` |
| [v0_4_3 source](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/migration/impl/src/commonMain/kotlin/io/github/stream29/kodex/app/migration/v0_4_3/MigrateToV0_4_3.kt#L1) | `a699ef34b394` |
| [v0_4_5 source](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/migration/impl/src/commonMain/kotlin/io/github/stream29/kodex/app/migration/v0_4_5/MigrateToV0_4_5.kt#L1) | `9b7bc79eb17b` |
| [v0_4_7 source](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/migration/impl/src/commonMain/kotlin/io/github/stream29/kodex/app/migration/v0_4_7/MigrateToV0_4_7.kt#L1) | `b29bc7c2f98a` |
| [v0_4_7 codec](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/migration/impl/src/commonMain/kotlin/io/github/stream29/kodex/app/migration/v0_4_7/SettingsCodec.kt#L1) | `b7620f5412b8` |
| [v0_3_3 tests](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/migration/impl/src/commonTest/kotlin/io/github/stream29/kodex/app/migration/v0_3_3/MigrateToV0_3_3Test.kt#L24) | `9fb7dfcc1ce6` |
| [v0_3_5 tests](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/migration/impl/src/commonTest/kotlin/io/github/stream29/kodex/app/migration/v0_3_5/MigrateToV0_3_5Test.kt#L21) | `436ed0e457a1` |
| [v0_4_3 tests](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/migration/impl/src/commonTest/kotlin/io/github/stream29/kodex/app/migration/v0_4_3/MigrateToV0_4_3Test.kt#L34) | `66a0a6c334f2` |
| [v0_4_5 tests](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/migration/impl/src/commonTest/kotlin/io/github/stream29/kodex/app/migration/v0_4_5/MigrateToV0_4_5Test.kt#L20) | `c5a518db5f78` |
| [v0_4_7 tests](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/migration/impl/src/commonTest/kotlin/io/github/stream29/kodex/app/migration/v0_4_7/MigrateToV0_4_7Test.kt#L32) | `1ae5ea6061bd` |
| [v0_4_7 fixture](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/migration/impl/src/commonTest/kotlin/io/github/stream29/kodex/app/migration/v0_4_7/SettingsFixture.kt#L1) | `d3d4b49fced5` |
| [v0_3_3 JVM performance probe](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/migration/impl/src/jvmTest/kotlin/io/github/stream29/kodex/app/migration/v0_3_3/MigrateToV0_3_3PerformanceProbeTest.kt#L50) | `9ea7cb24ab38` |

- Frozen registry still selects only 0.3.3, 0.3.5, 0.4.3, 0.4.5, 0.4.7.
  Historical test source preserves slim patch records, bundled skill replacement/
  neighbors, future activation guards, interruptions/reentry, token-count wrapping,
  settings split/credentials/conflict checks and multi-version upgrade into actual
  new stores; [multi-version consumer test](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/migration/impl/src/commonTest/kotlin/io/github/stream29/kodex/app/migration/v0_4_7/MigrateToV0_4_7Test.kt#L248).
- This comparison preserves historical evidence; it is **not** a new run of those
  fixtures or a claim all possible released migration failure states were reaudited.
  No version bump/new entry/serializer rewrite authorized or performed.

## Final acceptance boundary

- **Physical closure:** observed for owned UI primitives/Patch/Home declarations;
  original source/tests moved, retired coordinates absent, frozen history preserved.
- **Real takeover:** observed. UI is genuine Mosaic infrastructure, Patch remains
  actual consumer-local renderer, Home spec is consumed by original factory,
  direct retained lease and production host; no speculative pairing abstractions.
- **Behavioral completion:** **not all green**. Home lifetime retains two confirmed
  B1 baseline failures; deduplicate both against other-utils. Honest KDoc and
  characterization tests do not fix them.
- **Execution acceptance:** no new JVM/Native/UI/CLI/FS run. Static source/test/
  Gradle wiring is reviewed; full Native terminal and release runtime validation
  remain coordinator/user-gated. Other 203 projects and full History ownership are
  not silently certified by this lane.

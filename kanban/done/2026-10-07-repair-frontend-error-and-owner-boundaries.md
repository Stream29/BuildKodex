SCOPED REPAIR SEALED FOR E2E — full E2E acceptance pending

- 协调者：RPC 85、Application 51、根 View 54、Catalog View 25、
  History VM 50 项 JVM 通过。原 parent/error/CAS/准入和外部写完成修复
  获对应独立静态支持，不把已有导航用例推广为全部 admission 已闭合。
  [最终统一证据](2026-10-07-plan-spec-impl-closure-and-consistency.md#final-scoped-closure-evidence)。

# Task Tree

- `Trace real root error outlets and draft parent ownership`()
- `Repair confirmed owners and add discriminating B2 gates`()
- `Add genuine renderer and gated source regression tests`()
- `Publish integration handoff for centralized tests and independent review`()
- `Receive authorization to seal and commit the scoped repair`()

# Details

## Early integration handoff

- Constructor changed: `RpcSessionDraft(initialSettings, views, ownerScope: CoroutineScope)`;
  no ownerless overload. Production consumer: `RpcNewSessionViewModel` passes its existing
  `local` child scope. Direct tests in `RpcSessionViewsTest` (three sites) and
  `RpcSettingsTest` (one site) pass the existing `frontend` scope.
- Queue changed: `SettingsUpdateQueue.submit(...): Boolean` means actual admission,
  not persistence; `RpcGlobalEditor.acceptWrite` and defaults admission consume it.
  Terminal operation cancellation stays cancellation; stop admission and release pending closures,
  distinguish successful drain from terminal cleanup.
- F1 uses the existing CLI coroutine exception boundary for throwing History/catalog
  operations, not a new error owner. Synchronous History request must launch in the captured
  renderer scope to reach that boundary. Catalog gets an explicit failure callback with a
  throwing default; root passes no duplicate reporter. Cancellation always rethrows.
- New tests live in RPC/New Session/History test sets and Application JVM root-UI tests.
  Coordinator owns all Gradle changes, including any test dependencies these tests need.
  No obsolete Session Settings constructor lines were edited here; the new real source/VM
  gates use the current `createSessionSettingsViewModel(SessionSettingsDependencies(...), scope)` API.
- Central tests and independent acceptance: **PENDING**. Static interleavings are not runtime
  reproduction; experiments below are expectation gates to execute centrally.

### Coordinator cross-file integration

- `app/test-support/rpc/.../RpcFrontendFixture.kt` is outside this lane. `startRpcFrontendFixture`
  must own a dedicated frontend child Job, and in its existing `finally` shutdown Sessions,
  cancel/join that frontend owner before leaving `withInMemoryRpc`. Otherwise `finish.await()`
  returning normally can wait forever on unclosed factory drafts (the original New Session
  owner was already a Job child; parenting Composer makes its lifecycle correct, not magically
  self-closing on normal scope-body return). Add/retain `fixture.closeAndJoin()` with an
  unclosed factory draft as a real shutdown gate. Do not cancel backend before Session cleanup.
- Root API adds optional `onOperationFailure: (Throwable) -> Unit = { throw it }` before
  the existing final `openMcpUrl` parameter. Existing trailing MCP opener lambdas stay valid;
  named arguments recommended for any positional fourth argument. Catalog adds optional
  `onFailure: (Throwable) -> Unit = { throw it }`. Root forwards the same reporter once.
- Queue `close` callback now receives `drained: Boolean`; this is cleanup even after failure.
  `RpcGlobalEditor` uses `updates.close { mcp.close() }`, not `mcp::close`. Normal close drains;
  worker cancellation cancels pending closures and yields false, never a success receipt.
- Actual root UI gates are in `app/impl/application/src/jvmTest/.../ApplicationRootBoundaryTest.kt`
  so they can construct the original internal Application owner, not a fake navigation VM or
  new public factory solely for tests. **Coordinator Gradle action required:** add
  `jvmTest.dependencies { implementation(project(":app-impl-view")); implementation(libs.mosaic.testing) }`
  in `app/impl/application/build.gradle.kts`. These are test-only edges, not a main graph cycle.
  Apply `kotlin("plugin.compose")` there for the JVM test's composable lambdas:
  `kodex.kmp-viewmodel` itself does not apply that compiler plugin. Keep root View dependencies
  confined to `jvmTest`, not `commonMain`; no new main-source view ownership is introduced.
  If transitive compile visibility is insufficient, add direct test dependencies for
  `app-component-session-tab-bar-spec`, `app-component-agent-spec`, `agent-storage-spec-clean-models`,
  `rpc-impl-client` and `rpc-spec-contract` as needed; all Gradle edits remain main-owned.

## Authorized scope

- 用户授权修复后重新审查；[主任务](2026-10-07-plan-spec-impl-closure-and-consistency.md)，
  基线 `6b7129fa`。[前端报告](../done/2026-10-07-final-audit-frontend-ownership.md)
  F1/F2 已确认；F3/F4 与 Settings B2-D/B2-Q 先复现，不宣称已发生用户故障。
- 本线独占 `app/impl/{rpc,view,application}`、New Session impl/viewmodel、
  History impl/viewmodel/spec 及相应专属测试。可以改 Session Catalog View 的
  真实错误出口与 KDoc，不能改 Session Settings 兼容 API（另一线）。
- 原 History/catalog request/revert/fork/open 失败必须通过准确 owner 的现有错误
  能力报告一次；保留取消传播、捕获 target、不重放。不得新造 Failure VM、
  ErrorController 或复制 Composer 算法。
- RpcSessionDraft 原 scope 成为原 supplied owner 的 child；父 owner cancel/join
  关闭 Composer 与 observers，显式关闭仍只关闭自己的 child，borrowed backend
  不关闭。所有真实构造方直接适配，不加 owner-less 兼容构造函数。
- gate 原 History source read，超过 buffer 容量导航，断言最新 target 送达与
  stale ack 不清新 effect；若复现，原 single owner 内收敛/admit，不用无界 journal。
- 真实 draft/source 的 observer 延迟字段覆盖、真实 SettingsUpdateQueue 单次
  command cancellation 与仍活着 parent admission 分别测试；不靠 fake CAS。
  若复现，只修原 edit/admission 边界，保留 accepted-vs-persisted/FIFO 语义。
- equal Session twins 用真实 root tab/deletion 输入复现后修 referential lookup，
  不改真实 Session 身份或增加 ID/registry。
- 不构建、不提交/推送、不触碰其他线/IDE；只写自己的 handoff。
  完成时列实际 API、所有消费者、真实输入测试与仍未覆盖的路径；中央测试待执行。

## Source handoff

- **F1:** History request/revert-and-edit/fork/open reach the captured root renderer reporter
  once; cancellation rethrows. Default reporter throws to the existing CLI coroutine logger,
  not an invented Agent notification or a new failure state. Catalog open/fork invokes the same
  root callback; no false assertion that registry/Agent had already reported. Unrelated caught
  Agent confirmation/resume/Stop reporting is unchanged.
  [Root](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/SessionTreeCliScreen.kt#L935),
  [Catalog](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-catalog/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/sessioncatalog/SessionCatalogPopup.kt#L59).
- **F2:** the original draft SupervisorJob is a child of the mandatory supplied Job; its context
  retains the supplied dispatcher. Original close runs on owner completion; settings edits and
  materialization check the original owner. Parent join includes Composer observers; explicit
  close does not close views, Stop runtime, or cancel siblings. No ownerless overload.
  [Draft](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/rpc/src/commonMain/kotlin/io/github/stream29/kodex/cli/rpc/RpcSessionDraft.kt#L34),
  [Production caller](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/new-session/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/cli/newsession/NewSessionViewModel.kt#L32).
- **F3 candidate:** original History loop retains exactly one newest navigation plus bounded
  wake markers. A full command buffer does not discard that slot; stale in-flight identity and
  effect acknowledgments remain guarded. No journal, per-request launch, replacement VM or paging
  bound change.
  [Original owner](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/cli/history/AgentHistoryViewModel.kt#L739).
- **B2-D candidate:** source compares only changed fields with its captured baseline inside
  actual `draft.edit` mutex, then merges those fields into latest settings. Unrelated tier/name/
  runtime fields survive; same-field conflicts return false; source close during mutex wait
  cannot produce a late accepted edit. No Session Settings component/compatibility API edit.
  [Draft source](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/rpc/src/commonMain/kotlin/io/github/stream29/kodex/cli/rpc/RpcSessionSettings.kt#L156).
- **B2-Q candidate:** terminal worker cancellation propagates and cancels pending command
  closures. Later real global/default admission consumes Boolean rejection and publishes the
  existing sanitized global failure; cancellation itself is not business failure. Close cleanup
  receives false on terminal cancellation, true only on normal FIFO drain. Accepted remains
  distinct from persisted.
  [Queue](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/rpc/src/commonMain/kotlin/io/github/stream29/kodex/app/settings/SettingsUpdateQueue.kt#L29).
- **F4 candidate:** original tab identity retention uses pair-list referential lookup/pruning;
  Compose Session keys compare exact references, not equality. Deletion retains the selected
  survivor with `===`. Session identities and existing UI tab identity allocation are unchanged;
  no new registry or persistent ID.
  [Selection](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/application/src/commonMain/kotlin/io/github/stream29/kodex/cli/app/ApplicationViewModel.kt#L414),
  [Renderer keys](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/SessionTabRenderState.kt#L41).

## Central gates — all execution PENDING

| Suite / location | Genuine gate and expected assertion |
| --- | --- |
| [applicationRootBoundaryTest](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/application/src/jvmTest/kotlin/io/github/stream29/kodex/cli/app/ApplicationRootBoundaryTest.kt#L46) | Eight full-root input cases: real RPC revert/fork/open-after-fork fault, command cancellation, actual Agent validation rejection, catalog RPC open/fork failures, equal Session twins click/select/delete. Original internal Application and real backend/JSON/children; no isolated VM claimed as end-to-end. |
| [historyViewportAndScrollEffectTest](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/impl/viewmodel/src/commonTest/kotlin/io/github/stream29/kodex/cli/history/HistoryViewportAndScrollEffectTest.kt#L44) | Gate original index.getExact(7), send 256 alternating navigations plus final item/latest, release, require newest effect and stale-ack immunity. Existing repeated-target, late/unmounted, nonce/revert/close and viewport tests retained. |
| [draftAndSettingsBoundaryTest](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/rpc/src/commonTest/kotlin/io/github/stream29/kodex/cli/rpc/DraftAndSettingsBoundaryTest.kt#L25) | Seven real-chain cases: draft parent join/observer closure; explicit child-only close while actual backend turn runs; actual source observer paused with real VM executing changed-field merge/same-field conflict; real global/default CAS throws command-local cancellation with parent alive, later admission reports rejection; real VM close while draft materialize holds original mutex. |
| [settingsUpdateQueueTest](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/rpc/src/commonTest/kotlin/io/github/stream29/kodex/app/settings/SettingsUpdateQueueTest.kt#L18) | Terminal command cancellation with parent alive, queued closures dropped honestly, subsequent admission false/error, cleanup flag false; separate whole-parent cancellation. Original ordinary-error continuation, normal FIFO close-drain retained. |
| [newSessionViewModelTest](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/new-session/impl/viewmodel/src/commonTest/kotlin/io/github/stream29/kodex/cli/newsession/NewSessionViewModelTest.kt#L13) | Actual factory supplied-parent cancellation without close; fixture.closeAndJoin with unclosed factory draft (requires coordinator fixture integration). Existing materialization/explicit-close tests retained. |
| Catalog popup and Settings Login lifetime | Existing real-input failed-open case now asserts exact reporter invocation; identity churn assertion follows referential pair retention only. No obsolete Session Settings constructor edits. |

- B2-D/Q/F3/F4 have **not been runtime reproduced here**. Candidate corrections address the
  exact static interleavings; acceptance requires central failing-control + repaired behavior,
  not a characterization-green or merely compiling a new test.
- To run a baseline failing control, restore only the suspect behavior blocks while retaining
  the mandatory owner constructor/new error callback test APIs. New queue receipt assertions
  cannot compile against the old Unit API; use the real global/default owner gate or preserve
  the Boolean signature during behavioral ablation. Restore candidate code afterward. If the
  intended failure cannot be reproduced, do not declare that B2 closed or loosen the gate.
- Root faults are injected at real RPC command transport (not fake success). Request validation
  deliberately passes a stale generation to the actual captured Agent validator. Equal twins are
  test-only delegating factory/registry handles over real Sessions; concrete production Sessions
  still use reference equality. The root test reporter is the injected endpoint; the production
  default CLI logging connection is statically traced, not a fresh terminal-log capture.
- Existing renderer late-mount/repeated-target/nonce tests and all old recordings are retained.
  No new native terminal recording, full CLI startup run, JS/Native certification or independent
  acceptance is claimed. No Gradle execution/edits, IDE/device/resource-control operations,
  commits, pushes or branches were performed by this lane.
- Scoped `git diff --check` and constructor/consumer search completed; no scoped whitespace
  errors, ownerless real draft callers, or unadapted production queue consumers found.
  This is static evidence only. New JVM root-test dependencies, fixture integration and all
  centralized executions remain coordinator-owned **PENDING**.

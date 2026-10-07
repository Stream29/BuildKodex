SCOPED REPAIR SEALED FOR E2E — full E2E acceptance pending

- 协调者：真实 draft/materialize 与 nonce 门闩包含于 RPC 85 / History 50 项
  通过；第二独立轮次支持 S1/S2 静态修正，不声称作者提前写出的 red 预测已执行。
  无 nonce 外部写责任保留，外部写完成交错的旧字节 red 与修复后 green 见
  [最终统一证据](2026-10-07-plan-spec-impl-closure-and-consistency.md#final-scoped-closure-evidence)。

# Task Tree

- `Read fixed frontend interleaving suspicions`()
- `Add actual-source draft and saturated nonce regression gates`()
- `Repair statically confirmed gates at their original mutation owners`()
- `Publish deterministic gates for centralized validation and delta review`()
- `Receive authorization to seal and commit the scoped repair`()

# Details

- 用户授权修复后重新审查；[第一轮独立复审](../done/2026-10-07-recheck-frontend-contract-closure.md)
  的 S1/S2 是有具体时序的疑点，不是已经运行失败。
- 独占 RpcSessionDraft、RpcSessionSettings、History spec/VM 及对应测试；其他
  frontend root 与 fixture 已验证增量不重写。
- 真实 materialize 持锁后 source 准入，再成功变为 nonwritable；核验原 source
  约定 false，而不是泛捕全部 IllegalStateException。修正只能在原 draft lock。
- 真实 read gate、饱和 wake buffer、cacheNonce 改变和新 Latest 共存；
  核验新代次 effect 不被迟到 Invalidate 撤回，原 child/window 身份不越代。
- 不新增 draft/History owner、resolver、通用 retry、journal 或配置副本。
- 不运行构建/设备，不提交/推送；测试由协调者集中执行，固定报告不改写。

## Integration handoff

- **Early REDTESTREADY checkpoint:** regression bodies were published before production
  edits, so the coordinator could execute red controls asynchronously. Production fixes are
  now applied; no tests/builds have been run by this lane. The failures below remain predictions
  from actual code gates, not a claim that centralized red/green executions passed.
- S1 tests in
  [DraftAndSettingsBoundaryTest](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/rpc/src/commonTest/kotlin/io/github/stream29/kodex/cli/rpc/DraftAndSettingsBoundaryTest.kt#L164):
  `materialize makes queued actual draft source update nonwritable without failure` and
  `materialize makes queued actual draft Settings VM update nonwritable without failure`.
  Actual JSON RPC `createSession` blocks while materialize holds the original mutex; actual
  source configuration update is admitted and suspended on that mutex, without VM/source close.
  Release creation, await real materialization and recorded source outcome; expect false,
  reporter zero, unchanged full draft/binding and direct backend settings timeline values.
  Current `draft.edit` checks createdIndex before the source transform, so expected red is
  `IllegalStateException` instead of false (VM variant also reports it).
- S2 test in
  [HistoryViewportAndScrollEffectTest](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/impl/viewmodel/src/commonTest/kotlin/io/github/stream29/kodex/cli/history/HistoryViewportAndScrollEffectTest.kt#L110):
  `saturated navigation followed by nonce withdrawal preserves new Latest after queued invalidation drains`.
  Actual `getExact(7)` is blocked; 256 requests fill wake buffer. The original nonce collector
  executes withdrawal and suspends sending Invalidate (controlled owner scheduler), then Latest
  is requested at nonce 41. An actual UpdateLatestTurn/duration command queued behind that
  suspended sender provides the FIFO drain barrier; assertions read final steady effect/window,
  not the first transient effect. Expect nonce 41 Latest pending until exact ack, original
  child cancelled/not reused and old timestamp read rejected. Current late Invalidate
  unconditionally withdraws navigation after Latest publication: expected red is null final effect.
- **Coordinator-only dependency action:** History viewmodel `commonTest` now uses
  `StandardTestDispatcher`/`TestCoroutineScheduler`; add
  `implementation(libs.kotlinx.coroutines.test)` in its existing build file if not otherwise
  provided. This lane does not edit Gradle. RPC suite already uses these test APIs.
- Retained controls: saturated nonnonce item/latest, nonce withdrawal, repeated exact targets,
  late mount/ack, sparse viewport/visibleChunks/follow, same-field conflict/unrelated-field merge
  and close-while-CAS-waits remain unchanged.

## Final source handoff

- **S1 actual gate:** source admission is outside the draft mutex, while `createdIndex` is
  committed inside real materialize before unlocking. Previously source transform could never
  express rejection because `edit` threw first. The original mutex now admits
  `internal suspend fun tryEdit((KodexAgentSettings) -> KodexAgentSettings?): Boolean`:
  closed/materialized returns false, null transform returns false without mutation, otherwise
  commits latest settings and explicit-name tracking together. Only `RpcDraftSettingsSource.change`
  uses it; public `edit`, `clearExplicitThreadName`, materialize and all factories are unchanged.
  General late edits still throw ISE; no ISE catch or compatibility overload was added.
  [Original mutation owner](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/rpc/src/commonMain/kotlin/io/github/stream29/kodex/cli/rpc/RpcSessionDraft.kt#L82),
  [Source consumer](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/rpc/src/commonMain/kotlin/io/github/stream29/kodex/cli/rpc/RpcSessionSettings.kt#L187).
- Changed-field baseline/revision checks and latest-field merge are retained. A cancelled caller
  waiting on the original lock still gets CE, not false or reporter failure; new regression
  `caller cancellation while actual draft source waits on materialize lock remains cancellation`
  checks this and full unchanged settings.
  [Cancellation test](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/rpc/src/commonTest/kotlin/io/github/stream29/kodex/cli/rpc/DraftAndSettingsBoundaryTest.kt#L241).
- **S2 actual gate:** the retained intent runs immediately after a command, before the next
  receive; therefore prioritizing only the queued Invalidate branch would be insufficient.
  The original History owner synchronizes nonce/generation both before consuming a command and
  before retained navigation. This releases old children first. Internal intents, Invalidate
  and paging demands capture generation; outdated Invalidate/paging is discarded. New-nonce
  navigation survives withdrawal of an older generation; already-applied Invalidate is
  idempotent and does not recreate the destination or clear its exact pending effect.
  Structural batches check current nonce before materializing children. If nonce advances
  during generation loading, synchronization advances to that latest generation; this is not
  error retry, command replay or a navigation journal.
  [Withdrawal/intent identity](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/cli/history/AgentHistoryViewModel.kt#L234),
  [Generation priority](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/cli/history/AgentHistoryViewModel.kt#L759),
  [Effect contract](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/history/contract/AgentHistoryViewModel.kt#L121).
- **External API delta: none.** `createAgentHistoryViewModel(source, ownerScope, running)`,
  `RpcSessionDraft(initialSettings, views, ownerScope)`, Settings dependencies/factory and
  public window/effect/ack types/signatures remain unchanged. No new source/owner/resolver,
  renderer authority, notification retry, fixture, project, or Session Settings compatibility
  field was added.
- Action identity remains exact instance + generation: same target requests are distinct,
  stale ack cannot consume a newer effect, old-window viewport/follow/page commands cannot gain
  admission. Original sparse/chunk retention and nonnonce destructive invalidation semantics
  remain. S2 starts with the original lazy tail child and finishes after the turn FIFO barrier;
  old payload job must be cancelled, old timestamp access rejected, replacement tail must be a
  different instance whose actual payload/timestamp operations have the current load context.
- Test cleanup is in `finally`: release gates, cancel/join actual operation jobs, close local
  VM/source/draft owners; History closes only its child scope, pumps cancelled owner to completion,
  then joins the repository. Borrowed backend/views are not closed by draft/source production
  changes. No temporary files or managed resources were created by this lane.
- Static scoped `git diff --check` and typed operation/command searches passed. This is not
  compilation or runtime acceptance. Only the six named production/test files plus this handoff
  were edited; root/startup, parent-close fixture, Gradle/Mosaic UI, lease/cache lanes, frozen
  snapshot and independent reports were not changed. No Gradle/build/IDE/network/fixture run,
  process/resource control, commit/push/switch was performed.

## Central validation list — READY, execution PENDING

- Run both S1 nonwritable tests through the real RPC source and Settings VM; check false,
  zero reporter calls and unchanged full local/persisted configuration.
- Run S1 caller-cancellation test plus existing close-CAS, same-field conflict and unrelated-field
  merge controls. Existing parent-close/explicit-edit tests remain separate coordinator gates.
- Run S2 saturated nonce test through the final FIFO barrier, plus retained saturated nonnonce,
  nonce-withdrawal, repeat/late-mount/ack, sparse/visibleChunks and follow controls.
- Then centralized compilation/platform matrix and independent review of these deltas.
  This lane finishes without waiting for builds or altering any immutable report. If a red
  checkpoint was not captured centrally, use the frozen pre-delta source as the behavioral
  control; do not relabel this static confirmation as a measured failing reproduction.

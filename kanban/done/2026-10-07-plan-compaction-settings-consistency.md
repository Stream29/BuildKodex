# Task Tree

- `Trace compaction snapshot, concurrent settings and checkpoint writes`()
- `Receive repair authorization for latest-settings preservation`()
- `if (authorizedToExecute()) {`
  - `Make the deterministic overwrite probe assert the desired invariant`()
  - `Repair the original checkpoint write inside the existing Mutex`()
  - `Verify normal, concurrent, failure and cancellation paths`()
  - `Validate storage and RPC observation without wire changes`()
  - `Receive authorization to seal and commit the scoped repair`()
- `}`

# Details

- 协调者：State 66、Session filesystem 109、RPC server 135 项 JVM 通过。
  缓存失败重扫后的准入由实际 SafeRw fail-closed，真实 Session/CAS 与重开
  用例通过；两个独立轮次支持该静态修复，范围边界见
  [最终统一证据](2026-10-07-plan-spec-impl-closure-and-consistency.md#final-scoped-closure-evidence)。

- [主计划](2026-10-07-plan-spec-impl-closure-and-consistency.md)；
  [现有串行化](../../checklist/agent-state-mutation-serialization.md)。
  上次 AgentState 迁移明确只复现故障；用户现授权修复后重新审查。

## Actual defect and unique owner

- 原 `KodexAgentState` 是唯一实际 owner，不新增 CompactionCoordinator、
  SettingsMergeService 或配置镜像。
- [compact](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-state/impl/state/src/commonMain/kotlin/io/github/stream29/kodex/agentstate/impl/KodexAgentStateImpl.kt#L288)
  在短 Mutex 内捕获 request snapshot，网络等待释放锁，允许 settings/CAS 成功提交。
  [checkpoint](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-state/impl/state/src/commonMain/kotlin/io/github/stream29/kodex/agentstate/impl/KodexAgentStateImpl.kt#L632)
  仍用开始时完整 settings，覆盖已接受写入。
- [门闩复现](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-state/impl/state/src/commonTest/kotlin/io/github/stream29/kodex/agentstate/impl/AgentStateHardCutoverTest.kt#L112)
  证明 CAS 已返回成功并落盘的 `threadName`，在 checkpoint 被旧值替换。

## Proposed checkpoint rule

- 网络请求仍使用开始时固定 settings/message snapshot，不重发/重算已在执行的请求。
- checkpoint 原 `writeMutex` 内重新读取实际 storage 最新 settings，把它作为
  原 `appendCompaction` 的配置基底；只由原存储 helper 更新其既有新 window 元数据。
  用户配置与自动标题等期间成功接受的值均保留，不按字段建立第二权威。
- 不把 Mutex 延长到网络等待，不禁止已允许的运行期 settings/CAS；
  不增加自动 retry、补偿写入、新 RPC 命令、配置 revision 或 serializer 字段。
- 原 `Compacting` ownership 检查、单次 checkpoint、latestIndex/state 最终恢复、
  失败/取消路径和工具 pending/work/index 语义保留。
- 若其他可并发路径改变了压缩的实际历史目标，需先用真实门闩确认，不能把
  保留 settings 的修复扩展为新的跨 storage 事务或恢复协议。

## Work and gates

- 本线：AgentState spec/KDoc、唯一 State impl、针对性 common tests；
  只有原 `appendCompaction` 确实缺少所需能力时才修改其真实 helper。
  同线负责真实 storage cached State 的 publication+cleanup、
  settings append+timestamp 疑点：先故障注入复现，确认后只修原 cache/index
  投影最终一致性；不改变 write/revert/latest 格式、冻结 layout 或重做事务算法。
  协调者负责 RPC/Session 集成与规范同步。
- 将现有 characterization 的旧值断言改成“latest 值保留”先红后绿；
  保留原 accepted index/checkpoint index/state 顺序和真实 CAS。
- 增补 rename、model/effort/tier、其他用户配置多次更新；检查相等与 stale CAS、
  checkpoint 写入竞争次序、网络失败/取消，以及新 window 元数据仍只更新一次。
- 验证普通与强制 compact、RequestResponse/steer/tool 的旧不变量，
  RPC settings timeline/catalog/name 订阅最终观察到实际保留值，不靠缓存掩盖覆盖。
- State、Storage contract-ext/in-memory/filesystem、Runtime compact/composition、
  Session、RPC server 与前端真实观察定向回归；CLI/Integration 编译、Native 主源码。
- 原 persisted/RPC 形状与 spec 领域身份不变；用户现授权执行此修复。
- 本线不改 RPC server/前端、构建/IDE/其他线；涉及装配依赖在本文件
  `Integration handoff` 报告，中央验证与独立复验待协调者。

## Integration handoff

- Status: **SCOPED SOURCE HANDOFF READY; Central PENDING** for compilation,
  executable tests, B2 confirmation/repair and independent acceptance.
  No Gradle, IDE, commit, branch change or shared-resource acquisition by this line;
  no temporary files/resources were acquired. Other active worktree changes left alone.
- APIs/moves: **none**. Original State factory/spec, CAS and `appendCompaction`
  signatures remain. Only the private checkpoint's captured-settings parameter was
  removed. [Original checkpoint](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-state/impl/state/src/commonMain/kotlin/io/github/stream29/kodex/agentstate/impl/KodexAgentStateImpl.kt#L630)
  now reads `storage.settings[storage.latestIndex()]` inside existing `writeMutex`.
  The helper still owns only its existing window-lineage patch. Request
  model/tools/input/window metadata remain the initial snapshot.
- [Spec KDoc](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-state/spec/contract/src/commonMain/kotlin/io/github/stream29/kodex/agentstate/contract/KodexAgentState.kt#L222)
  reflects the repair. Storage helper/serialization/layout/latest/write/revert
  sources, migrations and frozen files are unchanged by this line.
- Existing tests: `AgentStateHardCutoverTest` now asserts **during**, not **before**,
  after accepted CAS/checkpoint; `KodexAgentStateImplTest` also asserts the
  accepted title at the final checkpoint. Neither was executed here; red/green
  execution remains central, not claimed from editing an assertion.
- [CompactionSettingsConsistencyTest](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-state/impl/state/src/commonTest/kotlin/io/github/stream29/kodex/agentstate/impl/CompactionSettingsConsistencyTest.kt#L43):
  four deterministic cases: forced/direct compaction with three accepted writes
  (automatic/manual title, model, tier, effort, questions, cwd, instructions,
  limit/routing), equal/stale CAS, exact checkpoint/timestamp indices and one
  window advance; network failure/cancellation preserves accepted settings and
  restores stable ownership without a checkpoint. Request snapshot/order checked.
- [StateFileSystemPublicationTest](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-state/impl/state/src/commonTest/kotlin/io/github/stream29/kodex/agentstate/impl/StateFileSystemPublicationTest.kt#L47):
  seven real-filesystem cases; original State/CAS and actual filesystem storage,
  controlled `CoroutineFileSystem` faults/gates, no fake CAS. Temporary test
  directories use bypass cleanup and child operations are joined.
  - Four `B2 timestamp ... resyncs durable settings with cas=...` cases cover
    unconditional/CAS writes, actual calling-Job cancellation/IOException,
    cleanup suppressed errors, raw numbered records/pointers, legal history,
    desired immediate State tail, stale CAS and next explicit append.
  - `B2 record publication then cleanup failure ...` faults the temp delete
    **after** the actual settings record move; checks durable data, State tail,
    propagated failure and immutable earlier records on the next explicit CAS.
  - Two `checkpoint waits for admitted filesystem CAS with cancelled=...`
    cases gate timestamp I/O while CAS holds original State mutex, return the
    network result, then release CAS. Checkpoint/cancelled checkpoint must wait,
    preserve accepted settings, and retain exact history/index order.
- Central dependency need: State common tests need `:agent-storage-impl-filesystem`
  for real filesystem faults; only main may edit Gradle. Existing coroutine-IO
  dependency supplies controlled filesystem wrappers.
- Scope discovery: actual `CachedIndexVersionedImpl.set` and `FileSystemAgentStorage.cached`
  are **not** under owned `agent-storage/{spec,impl}`. They are internal in
  [Session-owned cache](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-session/impl/filesystem/src/commonMain/kotlin/io/github/stream29/kodex/agentsession/filesystem/CachedAgentStorage.kt#L256).
  This line will not edit Session source/tests or expose/move internals.
  Main must supply the real Session-cache integration experiment/repair if needed.
- B2-3: direct-disk publication plus cleanup failure already has a real storage test;
  cache concealment/record overwrite remains **unconfirmed by execution**.
  Suggested central experiment: one-shot delete failure after settings entry move,
  inspect raw exact record/pointer, actual cache tail/nonce/value, State tail and
  subsequent explicit CAS. Only if confirmed: reconcile the original cache on
  uncertain delegate outcome while preserving thrown primary/suppressed errors;
  never swallow cleanup failure or retry/overwrite a numbered record.
- B2-4: **not repaired speculatively**. No authorized local test execution was
  available, so the five `B2 ...` filesystem tests above are deliberately
  desired-invariant confirmation gates, **expected to fail at State tail** on the
  untouched [appendSettings](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-state/impl/state/src/commonMain/kotlin/io/github/stream29/kodex/agentstate/impl/KodexAgentStateImpl.kt#L445).
  Do not classify this handoff as all-green or weaken those assertions.
  If central execution confirms: resync `latestIndex` in the original append
  boundary's bounded `NonCancellable` finalization; preserve `Compacting`/
  `RequestResponse` (do not restore stable State there), rethrow the original
  write failure/cancellation and suppress a secondary resync failure.
  No rollback, automatic retry, transaction, new revision or success reinterpretation.
- Risk disposition: compaction B1 has scoped source repair awaiting execution;
  B2-3 actual-cache experiment and B2-4 confirmation/repair remain **OPEN**.
- RPC seam (main-owned): gate real compaction transport, commit multiple settings
  CAS/title updates while `Compacting`, release, and assert settings timeline,
  name/catalog subscriptions and runtime latest index retain accepted values.
  Failed/cancelled timestamp writes must expose durable settings via corrected tail,
  without reporting CAS success or inventing a revision/wire field.

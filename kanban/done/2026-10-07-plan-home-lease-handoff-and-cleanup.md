SCOPED REPAIR SEALED FOR E2E — full E2E acceptance pending

- 协调者：Lease 54、Home 74 项 JVM 通过；真实隔离 CLI 正常退出且无 owner
  文件。共享引用/交接修复获独立静态支持，原失败快照不改写；整批结论见
  [最终统一证据](2026-10-07-plan-spec-impl-closure-and-consistency.md#final-scoped-closure-evidence)。

# Task Tree

- `Trace lease publication, actual owners and Home cleanup callers`()
- `Receive repair authorization for actual handoff and cleanup rules`()
- `if (authorizedToExecute()) {`
  - `Turn baseline fault probes into desired-invariant failing tests`()
  - `Repair the existing lease implementation and real waiting contract`()
  - `Adapt Home cleanup and hand off other await-release consumers`()
  - `Statically re-review ordinary lifetime, fault matrix and frozen paths`()
  - `Hand off for centralized CLI and regression acceptance`()
  - `Receive authorization to seal and commit the scoped repair`()
- `}`

# Details

- [主计划](2026-10-07-plan-spec-impl-closure-and-consistency.md)。
  用户已明确授权修复后重新审查；仅修原交接/清理规则，冻结协议不变。

## Confirmed baseline defects — 6b7129fa

- [lease 发布终验](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-07-final-audit-other-utils.md#L548)
  写 owner 后才建立 owner-bound renewable lease；期间取消可能留锁。
  原实际 Home 复现
  在原路径写入后挂起，取消取得操作，实际 orphan 仍存在。
- [等待清理终验](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-07-final-audit-other-utils.md#L573)
  仅 `Job.join()`，不返回背景 cleanup failure；Home 的实际 action 失败测试
  已复现“主异常保留，但清理失败不成为 suppressed”。
- 两项是确定性基线缺陷，不是契约移动引入；本批已把原断言改成修复后的不变量。
  未运行 red/green；实际执行保留给协调者，原终验记录保持固定。

## Authorized repair rules

- 发布 owner 到返回实际 lease 的取得操作必须有取消交接清理：未交付时删除准确
  本次 owner，有限 `NonCancellable` 内等待；失败保留原始异常，附加清理失败。
  不删其他 owner/未知文件，不把“存在锁”当本次 owner 的证明。
- 保留 `FileSystemLease : CoroutineScope` 的真实资源身份和原工厂。
  把已有实现端 `closeAndJoin` 变成实际 spec 操作，由原 lease 实现直接实现：
  请求取消、等待本实例清理完成，并返回该清理结果；删除被接管的旧扩展，不加 wrapper。
- `close()` 仍只请求关闭；`closeAndJoin()` 的调用者取消只停止等待，不撤销释放。
  重复关闭不重复扣共享引用；再次等待可观察相同完成结果。
- 有主失败时准确附加 cleanup failure；无主失败时等待关闭应暴露 cleanup failure。
  同一异常不自我 suppressed；不能让背景 handler 和同步路径制造两份释放操作。
- 共享读 handle 只释放自己的引用；最后一个引用负责实际 owner 清理。
  owner 结束必须等原 child 清理，不把显式关闭变成取消整个应用 owner。
- 不增加 LeaseManager、HomePreparer、结果 journal 或全局失败总线。
  若现有 child 完成路径无法支持上述契约，先报告准确问题，不另建平行资源模型。

## Ownership and validation

- 本线：`utils/filesystem-lease/{spec,impl}`、Home 非冻结 startup/handle 等待、
  专属 fault tests；协调者更新其他准确消费者与资源规范。
- 必测：exclusive/read/write/guard 发布前后取消、交付后 owner 取消、
  共享读引用重复 close、lease-loss/renew 退出、正常与清理失败等待、
  primary+cleanup identity/suppressed、成功动作+cleanup 失败可见。
- 保留原进程互斥、lease 过期和 Home 版本/跨版本 suites；故障 fixture 使用
  准确门闩与隔离系统 FS，不用 sleep 或用户 Home。
- 不修改任何冻结 `v*` 源/fixture/registry/路径、版本号、heartbeat 格式或锁文件协议；
  “实际无法删除”仍可留锁并报告失败，不承诺物理 FS 故障下锁必然消失。
- 下游 JVM：filesystem lease、Home、Session filesystem、RPC server、
  Application/Session/Root；CLI/Integration 编译、Linux x64 与隔离启动/关闭/重开。
- 用户修复授权覆盖上述真实不变量。实现线只改 lease、非冻结 Home startup/
  handle 和专属测试；其他 closeAndJoin 消费方由协调者按 handoff 直接适配。
- 不运行 Gradle、修改其他线/主记录、提交或操作设备；完成后写本文件
  `Integration handoff`（实际 API、改动/测试清单、未闭合项），集中验证待协调者。

## Integration handoff — early API/error contract

- `FileSystemLease.closeAndJoin(): Unit` is now a **spec member**, same call name;
  the old impl extension is removed. Factories and heartbeat serialization stay unchanged.
- Each renewable lease/shared handle retains one release result. `close()` requests
  only its child cancellation; `closeAndJoin()` joins that child and observes the
  saved failure. Cancelling a waiter never cancels cleanup; repeated waits report
  the same original failure without releasing twice. Background cleanup is not a
  separate throwing/error-handler path.
- Publication failure/cancellation closes only the exact `(pid, acquiredAt)` owner;
  guard/resource handoff failures await undelivered leases. Necessary cleanup uses
  finite `NonCancellable`; primary failure is retained with distinct cleanup suppressed.
  Physical deletion failure may leave the owner file; no physical-success guarantee.
- Main adaptation: replace lease `close(); coroutineContext.job.join()` in
  `agent-session/impl/filesystem` (repository/entry/catalog temporary leases and
  acquisition failure paths) with the spec member, preserving primary errors.
  Home implementation is this lane; Application/CLI/test-fixture finalizers,
  lifecycle checklist and Gradle are main-owned. No import of the removed internal
  extension is needed.
- Test inventory implemented in source: isolated real-FS publication cancellation for
  exclusive/read/write/guard, foreign identity preservation, guard cleanup failure
  after owner publication, shared repeated close/last release, cancelled waiter +
  repeated failure, parent cleanup gate, renewal/lease-loss, Home primary+cleanup
  and successful action+cleanup failure. Existing process and frozen suites retained.
- Baseline fault assertions are ported to repaired invariants; no red/green execution
  claimed because builds/tests are explicitly reserved for centralized validation.

## Final integration handoff — implementation and self re-review

- Status: **READY — CENTRAL TESTS PENDING** (`readyCENTRALTESTSPENDING`).
  This lane finished implementation and static self re-review independently;
  it does not claim centralized execution or independent acceptance.
- Source baseline remains `6b7129fa0e719dde8f88a0034b9d572679984a03`.
  Shared-tree changes from other lanes were not attributed to this lane.
  Only the ten scoped source/test files below and this task were edited here.

### Actual API and ownership

- [Sole lease contract](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/filesystem-lease/spec/src/commonMain/kotlin/io/github/stream29/kodex/utils/filesystemlease/FileSystemLease.kt#L14)
  owns `close()` and `suspend closeAndJoin()`. The obsolete impl extension is gone;
  factory names, returned resource identity, owner heartbeat schema and paths remain.
- [Original renewable lease](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/filesystem-lease/impl/src/commonMain/kotlin/io/github/stream29/kodex/utils/filesystemlease/FileSystemLeaseImpl.kt#L38)
  attaches its one lifetime child **before publishing bytes**. Publication and
  terminal results are in-memory values, not jobs owning another resource or a journal.
  Failed/cancelled acquisition closes and joins this child before propagating failure.
- [Exact owner release and cleanup](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/filesystem-lease/impl/src/commonMain/kotlin/io/github/stream29/kodex/utils/filesystemlease/FileSystemLeaseSupport.kt#L24)
  uses the existing `(pid, acquiredAt)` check; foreign/malformed bytes are not proof
  of ownership. Physical release/heartbeat-temporary cleanup has a 10-second
  cooperative deadline; outer release waits have 30-second deadlines.
  Only necessary cleanup is masked.
- Concurrent integration's explicit `Dispatchers.Default` release deadline was
  retained. Dispatcher switching/return is nested **inside** `NonCancellable`,
  preventing prompt cancellation from discarding the saved cleanup result.
  Ordinary acquisition, heartbeat waits and use are not masked.
- [Read reference owner and actual handles](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/filesystem-lease/impl/src/commonMain/kotlin/io/github/stream29/kodex/utils/filesystemlease/FileSystemReadLeaseImpl.kt#L87)
  retain the existing map/mutex/count. Each handle's one child releases its one
  reference, and only the last reference closes/awaits the shared owner.
  Lease loss/renewal failure also ends resource children, not the parent owner.
- [Write acquisition](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/filesystem-lease/impl/src/commonMain/kotlin/io/github/stream29/kodex/utils/filesystemlease/FileSystemWriteLeaseImpl.kt#L24)
  retains an acquired-but-undelivered lease across guard cleanup/reader admission.
  Read acquisition does the same before installing its shared reference.
  A guard cleanup failure cannot silently discard an acquired owner.
- [Home startup and finalizers](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/migration/impl/src/commonMain/kotlin/io/github/stream29/kodex/app/migration/KodexHomeMigration.kt#L80)
  use the member on initial/write/final retained reads, keep original primary errors,
  suppress distinct cleanup errors, and expose cleanup failure after successful work.
  Active caller/owner/retained-lease checks precede Home handle return.
  [Home spec](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/migration/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/migration/KodexHomeHandle.kt#L31)
  now documents these observed results rather than the old join-only limitation.

### Test inventory and centralized targets

- [New lease lifecycle suite](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/filesystem-lease/impl/src/commonTest/kotlin/io/github/stream29/kodex/utils/filesystemlease/FileSystemLeaseLifecycleTest.kt#L38):
  39 named cases after matrix expansion; isolated system FS and exact gates.
  - Exclusive/read/write/guard before/after publication cancellation; foreign replacement;
    exact primary+cleanup identity, including caller cancellation.
  - Owner cancellation while publication is suspended, with cleanup gate proving
    parent completion waits; guard failure/cancellation after successful owner acquisition.
  - Cancelled close waiter followed by repeated failure observation; shared repeated
    close and last-reference failure; explicit close leaves parent/sibling alive.
  - Self-suppression rejection, unknown malformed publication retained, cleanup
    `CancellationException` saved as failure rather than mistaken for close.
  - Lease loss for exclusive/read/write cancels child consumers and preserves
    foreign owner; renewal primary+release failure. Renewal uses a manually advanced
    `UnconfinedTestDispatcher`, not sleep or automatic advancement over real FS cleanup.
- [Home lifecycle suite](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/migration/impl/src/commonTest/kotlin/io/github/stream29/kodex/app/migration/KodexHomeHandleLifecycleTest.kt#L232):
  two baseline characterizations now assert no orphan / exact suppressed cleanup.
  Four additional cases cover successful migration+failed cleanup, repeated last
  reference failure, version-read cause+cleanup, and owner cancellation during the
  matching-version read. Existing public-factory lifetime cases remain.
- [Original lease suite](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/filesystem-lease/impl/src/commonTest/kotlin/io/github/stream29/kodex/utils/filesystemlease/FileSystemLeaseImplTest.kt#L25)
  retains all six ordinary cases; its helper now observes the member result.
  JVM cross-process lease and generic/frozen Home migration suites are unchanged.
- Coordinator targets: filesystem lease + migration JVM tests, Session filesystem,
  RPC server and Application/Session/Root regression; CLI/Integration compilation,
  Linux x64 and isolated startup/close/reopen. Other consumers/Gradle/checklists
  remain main-owned. No imports of the old extension are required; the now-unused
  filesystem-lease impl dependency on `utils-coroutines-spec` can be removed by main.

### Static checks and remaining boundary

- Re-read the scoped source/test diff and lifecycle/Home requirements; checked
  publication ownership, guarded handoff, one result per handle, last-reference
  release, primary/suppressed identity and lease-loss child termination.
  `git diff --check` for owned paths passed.
- Compared every frozen `v*` source/test/fixture and `KodexHomeMigrations.kt`
  against `6b7129fa` at its **same path**, using baseline blob versus working-file
  hash: **16 files, zero mismatches**. No new frozen path, registry or version file
  appeared; version/generator changes were absent. No other migration model changed.
- Original other-utils/UI/Home audits were not rewritten. This is a new repair
  handoff, not retroactive editing of baseline evidence.
- No Gradle/build/test execution, IDE/resource/network operation, commit, push or
  branch operation was performed by this lane. No temporary files/resources remain.
- Physical delete failure or unidentifiable partial publication can still leave
  bytes/owner on disk, with an observable failure. Cooperative timeout cannot forcibly
  interrupt a non-cooperative OS syscall. The existing heartbeat/stale-acquisition
  protocol is retained, not replaced with an atomic compare-and-delete protocol.
  Independent review and centralized results remain pending; do not mark the whole
  migration or repository accepted from this handoff.

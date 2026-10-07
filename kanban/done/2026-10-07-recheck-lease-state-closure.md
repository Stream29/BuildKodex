REVIEW READY — SCOPED STATIC RE-REVIEW; NOT ACCEPTED

- 审查脚本已完成并归档；本报告结论对应第一固定快照，后续修复证据见
  [当前主任务](2026-10-07-plan-spec-impl-closure-and-consistency.md)。

# Task Tree

- `Independently review fixed lease, Home, State and cache ownership()`
- `Check every repair against baseline and genuine regression evidence()`
- `Report remaining defects, unsupported guarantees and minimal ablations()`

# Details

- 用户授权修复后重新审查；本任务是独立只读复审，不授权改代码或提交。
- 原基线 `6b7129fa`；固定修复快照、manifest 和集中测试证据由
  [修复主任务](2026-10-07-plan-spec-impl-closure-and-consistency.md)统一记录。
- 独占审查：filesystem-lease、Home 非冻结启动/契约、AgentState 与 Session
  文件缓存、存储部分写入与清理；读取消费者，不与实现者共用结论。
- 核对真实 scope/Job、准确 owner 身份、父 join、primary/suppressed、已发布
  但未交付、共享引用、最新 settings 和非事务写入结果。冻结 16 文件只读。
- 仅修改本文件报告；不构建、运行 fixture、操作 IDE/设备或修改原审查报告。
- 分别报告确认缺陷、需复现疑点、已知限制、实际审查覆盖；列真实文件 URI、
  行号与最小去层候选。快照不含 Git 历史，历史只从原仓库读固定基线。

## Independent report

### Verdict and evidence boundary

- Independent reviewer; I did **not** implement these repairs.
- The original publication/handoff, awaited cleanup-result, compaction-settings,
  and ordinary partial-settings-write repairs have meaningful source support.
  This is **not** scoped acceptance: B1–B3 below remain source-confirmed closure
  blockers. Their controlled executions have **not** been performed by me.
- Reviewed filesystem-lease spec/impl, Home spec/non-frozen preparation,
  AgentState contract/state implementation, Session filesystem/cache, and related
  filesystem timeline outcomes. Read production callers and relevant tests;
  did not review all 206 projects or certify the other review lanes.
- No Gradle, runtime test, fixture execution, IDE, network, device/resource
  operation, commit, push or branch operation. Only this report was edited.
  No temporary files were created; the coordinator owns the shared snapshot.
- Implementer/central handoffs are evidence of reported work, not independent
  runtime results. Test source can demonstrate an assertion/gate and baseline
  failure mechanism, but cannot establish a red/green run.

### Frozen provenance

- Read current project code **only** from
  [frozen snapshot](file:///tmp/kodex-closure-review-6b7129fa-20261007/).
  Source links below use the requested original repository URI; line numbers
  refer to the frozen files, not a subsequently changing working tree.
- [Manifest](file:///tmp/kodex-closure-review-6b7129fa-20261007/REVIEW_SOURCE_MANIFEST.json):
  SHA-256 `3d661ec7abdf3e33a8de2b103ad7a19b8504cfd2b8a1168f00980fbd7765335e`.
  Independently recomputed every listed file's SHA-256:
  **1290 entries, zero mismatches**.
- Scoped hash inventory: filesystem-lease 10 files, migration 26, agent-state 26,
  Session filesystem 7, agent-storage 80: **149 files**, all matched.
  Reproducible aggregate SHA-256 of sorted UTF-8
  `relativePath + NUL + manifestSha256 + LF` entries:
  `24adc66558c21df3768ea4a5d7bc57bfbbe32cc4a4842850729bcaadc21c1674`.
  Hash coverage is not a claim of equally deep semantic review of every file.
- History was read only through pinned Git objects in the original repository:
  `6b7129fa0e719dde8f88a0034b9d572679984a03`.
  Compared baseline blobs with snapshot bytes, not dirty working-source diffs.
- **16 frozen paths/byte contents unchanged**, with no added frozen path:
  registry; eight `v0_*` production files; six common-test codec/fixture/test
  files; one JVM performance probe. The
  [registry](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/migration/impl/src/commonMain/kotlin/io/github/stream29/kodex/app/migration/KodexHomeMigrations.kt#L9)
  retains targets `0.3.3`, `0.3.5`, `0.4.3`, `0.4.5`, `0.4.7`, in that order.
- Version-bearing `KodexHostKmp.kt`, `kodex.kmp-shared.gradle.kts`, generator,
  migration build script, `gradle.properties` and dependency version catalog
  also match baseline. Application version remains `0.4.8`; no version bump,
  registry activation or serializer-model change was found in this repair scope.
- Used locally cached **pinned kotlinx-coroutines 1.11.0 source** to verify
  `AbstractCoroutine` parent attachment and `CoroutineStart.UNDISPATCHED`
  cancelled-before-start behavior, and pinned kotlinx-io 0.9.0 source to verify
  `FileSystem.atomicMove` replaces an existing destination. No dependency
  download or execution.

### Confirmed blockers

#### B1 — P1: failed cache resync still admits stale CAS and record replacement

- Responsibility: `io.github.stream29.kodex.agentsession.filesystem.CachedIndexVersionedImpl.set`.
  [Failure resync](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-session/impl/filesystem/src/commonMain/kotlin/io/github/stream29/kodex/agentsession/filesystem/CachedAgentStorage.kt#L258)
  suppresses a scan failure/timeout and rethrows the write failure, but leaves
  the previous index list and an active, writable cache. It records no
  uncertainty that prevents subsequent append admission.
- Actual chain: `FileSystemKodexSessionRepository.open` →
  `FileSystemKodexAgentSession` → `FileSystemAgentStorage.cached` →
  `KodexAgentStateImpl.compareAndSetSettings` → `appendSettings` →
  `CachedIndexVersionedImpl.set` →
  `FileSystemIndexVersioned.setUnsafe`.
  [CAS](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-state/impl/state/src/commonMain/kotlin/io/github/stream29/kodex/agentstate/impl/KodexAgentStateImpl.kt#L438)
  trusts the cached tail/current value; the
  [unsafe filesystem write](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-storage/impl/filesystem/src/commonMain/kotlin/io/github/stream29/kodex/agentstorage/filesystem/FileSystemIndexVersioned.kt#L216)
  explicitly does not enforce append-only order.
- Source-determined sequence:
  1. Initialization is at index 0. Actual settings `1.json` is moved into place;
     its temporary-delete cleanup throws A, before timestamp append.
  2. Fault the ensuing settings-directory enumeration with B. The new resync
     catches B before replacing indexes. A is returned with B suppressed.
  3. State's subsequent `storage.latestIndex()` reads cached tails, not disk;
     it remains 0. Settings at cached index 0 remain the old value.
  4. `compareAndSetSettings(old, replacement)` therefore matches and appends at
     1. The cache check compares 1 against stale 0; `setUnsafe` replaces the
     already durable `1.json`. With faults removed, timestamp commits and CAS
     returns **true**, despite `old` no longer being the latest durable settings.
- This is an incomplete closure of the inherited cache-concealment defect,
  not a claim that the new happy resync branch always overwrites records.
  Reporting the secondary failure does not make the next write safe.
- Minimum controlled reproducer: extend the existing **real** filesystem
  publication/delete-failure gate with one failing `list(settingsDirectory)`,
  then use the actual cached Session/runtime CAS. Assert original A/B identity,
  raw `1.json` bytes unchanged, and no stale-CAS success or index reuse.
  Do not substitute a fake CAS. Not executed here.
- Required boundary: after an unresolved delegate outcome, the original
  timeline must not admit a write based on its stale cache. This can be a
  fail-closed admission rule; it does not require retry, rollback or a new
  transaction/service. The new unconditional reconciliation wording at
  [State latestIndex](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-state/spec/contract/src/commonMain/kotlin/io/github/stream29/kodex/agentstate/contract/KodexAgentState.kt#L145)
  also needs to distinguish successful resync from a reported resync failure.

#### B2 — P2: bounded shared-handle cleanup can abandon its only reference release

- Responsibility:
  `io.github.stream29.kodex.utils.filesystemlease.SharedReadLease` /
  `SharedReadLeases.release`.
  [Handle finally](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/filesystem-lease/impl/src/commonMain/kotlin/io/github/stream29/kodex/utils/filesystemlease/FileSystemReadLeaseImpl.kt#L135)
  invokes release inside the new 30-second deadline.
  [Release](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/filesystem-lease/impl/src/commonMain/kotlin/io/github/stream29/kodex/utils/filesystemlease/FileSystemReadLeaseImpl.kt#L89)
  decrements only **after** acquiring the global shared-read mutex.
- That same mutex is held across guarded, suspending filesystem acquisition
  for any directory, at
  [acquire lines 45–85](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/filesystem-lease/impl/src/commonMain/kotlin/io/github/stream29/kodex/utils/filesystemlease/FileSystemReadLeaseImpl.kt#L45).
  A slow acquisition of B can therefore prevent release of unrelated A.
- Source-determined sequence: keep the parent active; open the sole read handle
  for A; gate acquisition of B while it holds the shared mutex; explicitly close
  A. Its timed mutex wait is cancelled **before decrement**. A's handle saves
  that timeout and completes, but A's renewable owner remains active with
  `references == 1`. After releasing B's gate, repeated `A.closeAndJoin()` only
  replays the saved timeout. A subsequent A handle increments to 2 and closes
  back to 1: the abandoned reference keeps A's heartbeat/lock alive until parent
  cancellation.
- Finite cleanup is desirable, but the timeout cannot silently discard the only
  reference-release action. This breaks the exact-reference lifecycle behind
  [lease spec lines 9–10 and 19–23](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/filesystem-lease/spec/src/commonMain/kotlin/io/github/stream29/kodex/utils/filesystemlease/FileSystemLease.kt#L7).
  Unlike baseline's indefinitely waiting release, the new deadline makes this
  abandoned-reference outcome reachable.
- Minimum reproducer: two isolated directories, one real-FS publication gate
  for B, active parent, A close exceeding its actual cleanup deadline, then
  repeat close and reacquire/close A. Check raw owner files and actual handle
  Jobs. Existing repeated-close tests are uncontended and do not cover this.
  The timed execution was not run here.
- Required boundary: reference accounting must have a terminal, exactly-once
  owner even when its cleanup wait times out; do not solve this by retrying
  `closeAndJoin` or introducing a parallel lease manager.

#### B3 — P2: CLI logging-start failure loses its primary when Home release fails

- Actual caller `io.github.stream29.kodex.cli.app.main`:
  [logging catch](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/cli/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/Main.kt#L23)
  calls `homeHandle.closeAndJoin()` before reporting the original logging
  failure, without a masked cleanup context or primary/suppressed handling.
- Now `HeldLeaseKodexHomeHandle.closeAndJoin` →
  `SharedReadLease.closeAndJoin` can throw the saved cleanup failure. If logging
  fails with A and last-read-owner deletion fails with B, B escapes and the
  logging diagnostic at line 27 is never reached; A is lost. An already
  cancelled caller can likewise stop this wait.
- The caller is unchanged from baseline, but the repaired throwing close
  contract makes this an unadapted production error boundary. The ordinary
  Application finalizer **is** adapted; logging failure happens before entering
  `withKodexApplication`, so it does not benefit from that finalizer.
- Minimum reproducer: isolate logging startup failure and retained Home owner
  deletion failure in a controlled CLI startup harness; assert original failure
  remains primary/diagnosed and exact cleanup is suppressed, with cleanup awaited.
  Application's `onPhase` seam starts later and cannot test this earlier branch.
  No user Home/logging operations were performed here.
- This directly contradicts the
  [primary-error lifecycle SOP](file:///home/stream/ACodeSpace/push/BuildKodex/checklist/coroutine-resource-lifecycle.md#L10),
  not a request to change Home or logging policy.

### Supported scoped repairs and actual call chains

#### Lease and Home

- Contract is the original
  `io.github.stream29.kodex.utils.filesystemlease.FileSystemLease`; the real
  `CoroutineScope.FileSystemLease`, `FileSystemReadLease`,
  `FileSystemWriteLease` factories return it. No manager, marker interface or
  forwarding resource replacement was added.
- [RenewableFileSystemLease lifetime](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/filesystem-lease/impl/src/commonMain/kotlin/io/github/stream29/kodex/utils/filesystemlease/FileSystemLeaseImpl.kt#L39)
  is the **actual** owner-bound launch Job. Parent attachment precedes its body,
  and its body publishes the initial heartbeat. There is no separate
  `scopeJob` created first with a cleanup launch installed later.
- Cancelled-before-launch was considered, not assumed safe from an active check:
  pinned `UNDISPATCHED` still enters an already-cancelled child. Here the
  `ensureActive` is **inside** try/finally, so publication is skipped and the
  result/cleanup path still executes. The tests cover cancellation during
  publication, not a dedicated constructor-before-launch race; this part is
  supported by actual code plus the pinned library source, not a runtime result.
- `publication: CompletableDeferred<Result<Unit>>` gates acquired return;
  `completion: CompletableDeferred<Result<Unit>>` retains terminal errors that
  `Job.join()` cannot report. They are unparented **value latches**, not alternate
  resource scopes. Plain Result preserves injected failure identity instead of
  imposing a library exception-copying path.
- [Acquisition catch](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/filesystem-lease/impl/src/commonMain/kotlin/io/github/stream29/kodex/utils/filesystemlease/FileSystemLeaseImpl.kt#L148)
  awaits undelivered cleanup. Read/write keep the acquired owner across guard
  cleanup, so guard failure cannot discard it. Caller/receiver active checks
  precede acquisition return. Home additionally checks retained-lease activity.
- [Release identity](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/filesystem-lease/impl/src/commonMain/kotlin/io/github/stream29/kodex/utils/filesystemlease/FileSystemLeaseSupport.kt#L134)
  compares `(pid, acquiredAt)`, not mere path existence. Foreign identity is
  retained; malformed bytes fail closed and become cleanup failure rather than
  guessed ownership. No cleanup retry or foreign-file sweep was introduced.
- [Cleanup helper](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/filesystem-lease/impl/src/commonMain/kotlin/io/github/stream29/kodex/utils/filesystemlease/FileSystemLeaseSupport.kt#L24)
  keeps dispatcher return inside `NonCancellable`; physical cleanup has a
  cooperative 10-second limit and outer waits 30 seconds. Ordinary acquisition
  is cancellable. A stuck non-cooperative OS call is not forcibly bounded.
- Closing a handle cancels only its resource child. On renewal failure/loss,
  that child cancels its own consumers and exits; its parent is not cancelled.
  Saved cleanup `CancellationException` is a failure, not automatically an
  ordinary close. On direct renewal IO + release failure, the original renewal
  error is primary with distinct release error suppressed.
- Shared read accounting remains the existing map/mutex/count. Uncontended
  repeated close releases once; non-final handles do not close other refs.
  `owners[owner.key] === owner` prevents an old generation's reference release
  removing a newer map entry. B2 prevents claiming this invariant for every
  timeout/contention path.
- Home chain:
  `io.github.stream29.kodex.cli.app.main` →
  `io.github.stream29.kodex.app.migration.prepareKodexHome` →
  initial shared read/version check → optional exclusive preparation →
  final read/version check → `HeldLeaseKodexHomeHandle` →
  `withKodexApplication` → backend/RPC/frontend owners.
  [Preparation finalizers](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/migration/impl/src/commonMain/kotlin/io/github/stream29/kodex/app/migration/KodexHomeMigration.kt#L80)
  expose cleanup failure after successful work and suppress it onto an existing
  action/version failure. No handle is returned on these failures.
- [Application final release](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/application/src/commonMain/kotlin/io/github/stream29/kodex/cli/app/Application.kt#L160)
  follows frontend/backend closure and awaits Home in a non-cancellable
  finalizer. B3 is the separate earlier logging branch.
- Matching versions avoid Session scanning; unversioned layout validation still
  checks the six timelines and numbered-tail/latest agreement before version
  registration. Preparation remains non-transactional. Backend production
  [repository factory](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/server/src/commonMain/kotlin/io/github/stream29/kodex/rpc/server/BackendServices.kt#L71)
  is now installed lazily in the original host, rather than invoked merely to
  construct Application. This is caller tracing, not acceptance of the complete
  Backend lane.

#### State, Session cache and storage outcomes

- Actual State identity:
  `io.github.stream29.kodex.agentstate.contract.KodexAgentState` →
  `io.github.stream29.kodex.agentstate.impl.KodexAgentState` (scope extension) →
  private `KodexAgentStateImpl`, owned below Session.
  [Session assembly](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-session/impl/filesystem/src/commonMain/kotlin/io/github/stream29/kodex/agentsession/filesystem/FileSystemKodexAgentSession.kt#L27)
  uses that spec type and builds the production runtime from it.
- Request admission still fixes the initial snapshot index; settings/history
  reads for the active request use that immutable append-only prefix. Settings
  edits do not change the request's model/tools/input. They share the original
  Mutex with response-header commits and keep `RequestResponse` published.
- [Compaction start/commit](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-state/impl/state/src/commonMain/kotlin/io/github/stream29/kodex/agentstate/impl/KodexAgentStateImpl.kt#L633)
  retains initial settings/index for remote input, releases the Mutex during
  remote wait, and reads **latest settings under the same Mutex** for checkpoint.
  The original
  [appendCompaction](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-storage/spec/contract-ext/src/commonMain/kotlin/io/github/stream29/kodex/agentstorage/contract/ext/CompactionStorage.kt#L13)
  patches only window lineage and writes the original point/output sequence.
  No second merge authority, serializer field, re-request or retry was added.
- [appendSettings failure boundary](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-state/impl/state/src/commonMain/kotlin/io/github/stream29/kodex/agentstate/impl/KodexAgentStateImpl.kt#L448)
  boundedly refreshes global tail after a partial write, rethrows the original
  IO/cancellation with resync failure suppressed, and **does not restore stable
  state there**. Thus an admitted settings failure during `Compacting` or
  `RequestResponse` does not steal the active operation's phase.
- After settings record publication followed by temporary-delete failure,
  successful cache resync scans numbered records, invalidates only the attempted
  index value, replaces index metadata, and exposes the new tail without changing
  cache nonce. Earlier immutable records are not rewritten. Next explicit CAS
  compares against the durable new settings; the failed CAS itself never returns
  true. This support is conditional on successful resync; B1 is the failed case.
- Settings success followed by timestamp failure legitimately leaves a sparse
  settings record without a timestamp at that index. The correction publishes
  that durable settings tail; it neither pretends cross-timeline atomicity nor
  compensates/retries the write. A later explicit update uses a new index.
- Production RPC chain:
  `BackendSettingsTimelineRpc.compareAndSet` →
  `BackendSessionHost.inSession` → runtime delegate → State's real CAS;
  [timeline observers](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/server/src/commonMain/kotlin/io/github/stream29/kodex/rpc/server/BackendTimelines.kt#L29)
  observe actual Session cache metadata. There is no lock-outside read/write
  imitation of CAS or replacement success result.
- Session delete/catalog temporary operations now use
  [useAndRelease](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-session/impl/filesystem/src/commonMain/kotlin/io/github/stream29/kodex/agentsession/filesystem/FileSystemKodexSessionRepository.kt#L428):
  the operation is a child of the acquired lease, and awaited release failure
  is surfaced or suppressed onto the operation failure. Async/Job exception
  recovery may copy a cancellation; retained cause and exact injected cleanup
  matter, not false assertions that the library never copies exceptions.

### Test evidence and missing cases

- [Lease lifecycle suite](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/filesystem-lease/impl/src/commonTest/kotlin/io/github/stream29/kodex/utils/filesystemlease/FileSystemLeaseLifecycleTest.kt#L38)
  exercises actual isolated system filesystem operations before/after
  exclusive/read/write/guard publication, caller/owner cancellation, foreign
  replacement, guard cleanup after owner publication, exact errors, repeated
  close, cancelled waiter, parent-join cleanup gate and lease-loss child exit.
  The baseline writes before attaching the cleanup owner; the post-publication
  gate therefore targets a real baseline failure, not a fake lease.
- [Home lifecycle suite](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/migration/impl/src/commonTest/kotlin/io/github/stream29/kodex/app/migration/KodexHomeHandleLifecycleTest.kt#L232)
  checks real publication cancellation, migration primary + cleanup, successful
  action + failed cleanup, repeated last-ref failure, version cause + cleanup,
  and owner cancellation at matching-version read.
- [Compaction consistency suite](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-state/impl/state/src/commonTest/kotlin/io/github/stream29/kodex/agentstate/impl/CompactionSettingsConsistencyTest.kt#L49)
  uses original State/real CAS with controlled transport: direct/forced compact,
  multiple title/model/config updates, equal/stale CAS, one lineage advance,
  and failure/cancellation without checkpoint. Baseline captured-settings
  checkpoint necessarily fails the new retained-settings assertion.
- [State publication suite](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-state/impl/state/src/commonTest/kotlin/io/github/stream29/kodex/agentstate/impl/StateFileSystemPublicationTest.kt#L53)
  uses actual filesystem storage and original State/CAS. Its four timestamp
  IO/actual-calling-Job cancellation cases assert durable settings, original
  error/cleanup, tail, unchanged legal history and next explicit append.
  It also tests record-move-then-delete failure and checkpoint waiting behind
  admitted filesystem CAS. These are not fake-CAS tests.
- [Cache regression](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-session/impl/filesystem/src/commonTest/kotlin/io/github/stream29/kodex/agentsession/filesystem/CachedAgentStorageTest.kt#L42)
  actually publishes a numbered record before failing temporary deletion;
  repaired assertions expose it and reject replacing its index. The baseline
  omits cache metadata publication after that failure. This fixture covers a
  real cached timeline, not the full cached-Session/runtime CAS chain.
- [RPC compaction regression](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/server/src/commonTest/kotlin/io/github/stream29/kodex/rpc/server/BackendRuntimeRpcTest.kt#L208)
  crosses the real RPC adapter, performs original CAS during controlled
  compaction, and waits for the settings-point index, not the output index.
  It does not by itself cover filesystem failures or catalog/name subscriptions.
- Missing: B1 secondary resync failure and actual cached-State stale CAS; B2
  mutex/deadline contention and old/new shared-generation overlap; B3 startup
  dual failure; settings failure while an active request/compaction retains
  phase; cached Session/runtime cancellation after publication; IO failure in
  checkpoint's individual timeline commits; post-move cancellation and valid
  stale pointer recovery. Existing orderly repeat/renewal cases do not prove
  these paths. No runtime-green assertion is made for any target/platform.

### Suspected reproducer, inherited debt and normative gaps

- **S1 — concrete cancellation/pointer reproducer needed, inherited storage
  mechanism:** [setUnsafe lines 233–239](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-storage/impl/filesystem/src/commonMain/kotlin/io/github/stream29/kodex/agentstorage/filesystem/FileSystemIndexVersioned.kt#L233)
  restores previous latest on any failed record move. Actual blocking
  [atomicMove dispatcher handoff](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/kotlinx-io-coroutines/impl/src/blockingMain/kotlin/io/github/stream29/kodex/utils/kotlinxiocoroutines/BlockingCoroutineFileSystem.kt#L26)
  can complete the OS move before cancellation is observed on return.
  Expected source outcome for that interleaving: numbered record survives,
  `latest.json` is rolled back to a **valid older record**. Successful new cache
  resync sees the new number but does not repair that pointer. Direct storage
  and catalog accept valid older pointers without scanning
  ([catalog repair](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-session/impl/filesystem/src/commonMain/kotlin/io/github/stream29/kodex/agentsession/filesystem/FileSystemKodexSessionRepository.kt#L450)).
  Gate after the actual record move, cancel the calling Job, inspect raw
  numbered file/pointer and cache/catalog separately. Not reproduced here;
  the timestamp-cancellation test fails before writing its temporary and does
  **not** establish this post-move outcome as safe.
- **D1 — inherited production lease-loss fencing gap:** Session's actual scope
  owns lease and State/runtime as sibling child branches
  ([assembly lines 33–57](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-session/impl/filesystem/src/commonMain/kotlin/io/github/stream29/kodex/agentsession/filesystem/FileSystemKodexAgentSession.kt#L33)).
  Lease loss now ends lease descendants, but does not end this Session scope or
  sibling runtime/cache. The close-on-Session-completion hook is the reverse
  direction. Likewise Home users are not lease descendants. Do not interpret
  the lease child-loss test as production Session write fencing after takeover.
  This topology is inherited, not a new parent-cancellation guarantee in spec.
- **D2 — inherited recovery-error/finite-cleanup limitations:** State's
  [finishCompaction, finishRequestResponse and mutate finalizers](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-state/impl/state/src/commonMain/kotlin/io/github/stream29/kodex/agentstate/impl/KodexAgentStateImpl.kt#L669) remain
  unbounded non-cancellable reads and can replace an earlier primary with a
  recovery failure. `startCompaction` publishes `Compacting` before its settings
  read and before entering compact's try/finally; a failed initial settings read
  can retain that phase. Session's
  [initial lease-acquisition catch](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-session/impl/filesystem/src/commonMain/kotlin/io/github/stream29/kodex/agentsession/filesystem/FileSystemKodexAgentSession.kt#L40)
  still invokes cancellable `scope.cancelAndJoin()`. Ordinary Session shutdown
  waits its scope Job but does not call the privately retained lease's result
  observer; do not infer saved cleanup-failure propagation from parent join.
  These are not corrected by the new narrow settings-resync catch; no broader
  primary/phase recovery pass is claimed.
- **D3 — inherited lease protocol boundary:** exact-owner read followed by
  delete, and heartbeat read followed by replacement, are not atomic
  compare-and-delete/update. A foreign replacement *between* those operations
  is outside the new before-read foreign-identity tests. Physical delete or
  malformed publication failure may retain bytes. No new foreign-owner sweep,
  retry, quarantine transaction or protocol was introduced.
- **Normative reconciliation remains coordinator-owned:** the
  [Home SOP's former no-extra-release-method rule](file:///home/stream/ACodeSpace/push/BuildKodex/checklist/kodex-home.md#L135)
  is superseded for this authorized repair by the actual spec member
  `closeAndJoin`; it is stale, not a reason to create another contract.
  [State SOP lines 8–10](file:///home/stream/ACodeSpace/push/BuildKodex/checklist/agent-state-mutation-serialization.md#L8)
  still require “storage transaction” and describe captured-settings overwrite.
  The repaired spec explicitly says non-transactional writes/latest checkpoint
  settings. Those SOP statements were not silently treated as current behavior.
- Spec `@throws` on lease/Home close correctly exposes cancellation and saved
  renewal/release failure. Home version/layout and State invalid-transition/
  storage behavior are represented by the original contracts, not only impl
  comments. Conversely, unconditional durable-tail recovery must not be claimed
  when resync itself fails (B1), nor successful recovery/finite completion for
  inherited D2. No new DTO/serialized revision was required.
- **Unverified targets:** centralized JVM suites, actual JS/Native resource
  behavior, process expiry/handoff, CLI/Integration compilation, startup/
  shutdown/reopen, and full subscription/catalog results. Existing process and
  frozen migration tests remain in snapshot; preservation is not re-execution.

### Minimum ablation: remove only responsibility-free structure

- Keep the new lease `closeAndJoin` member: deleting it loses the **actual**
  saved-release-result observation required by Home and Session callers.
  `Job.join()` alone does not recover that responsibility.
- Keep publication and completion Result latches: deleting publication permits
  returned-before-acquired handoff; deleting completion loses release/renewal
  errors after child completion. They do not justify an alternate resource
  interface, manager, journal or failure bus.
- Keep the actual owner-bound launch/finally, exact-owner comparison and
  acquired-but-undelivered local: deleting each respectively loses structural
  cleanup, foreign-owner safety, or guard-failure handoff cleanup.
- Keep shared references and generation identity guard: deleting them breaks
  legitimate same-process read sharing or lets an old generation remove a new
  map entry. B2 concerns finishing their existing responsibility, not replacing
  this model.
- Keep State's existing Mutex and latest-settings checkpoint read: removing
  either reintroduces an actual lost-update/CAS boundary; no merge service is
  needed. Keep cache resync and State tail publication as **different** existing
  responsibilities: numbered timeline metadata versus Agent global observation.
  B1 requires safe failure admission, not deleting either responsibility.
- Keep `SafeRw`: it is used for Session cache index reads/writes; removing it
  destroys a real concurrency boundary. `HeldLeaseKodexHomeHandle` translates a
  prepared Home/version plus lease into the original domain resource; it is not
  an empty new factory/controller layer.
- Actual safe removal candidate: unused
  [filesystem-lease impl dependency on utils-coroutines-spec](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/filesystem-lease/impl/build.gradle.kts#L12).
  No scoped production/test source imports it after eliminating the separate
  supervisor scope. Remove only that obsolete dependency if desired, not the
  live child-scope utility used by Session/State. No edits made.

### Independent completion

- Source review and provenance checks complete; report ready for asynchronous
  coordinator consumption. **Do not mark Lease/State closure accepted** until
  B1–B3 are dispositioned with the real invariants and centralized evidence.
- Supported repairs above remain scoped positives. S1, D1–D3 and unexecuted
  targets are explicitly separate; neither this report nor another lane's green
  build constitutes a whole-repository pass.

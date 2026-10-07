# REVIEW READY — independent non-Native delta review; scoped static repairs supported

- 本轮审查已完成，结论对应第二固定快照；History 新 B2 的后续门闩/修复不
  改写本报告；当前证据见 [主任务](2026-10-07-plan-spec-impl-closure-and-consistency.md)。

# Task Tree

- `Independently compare post-review ownership deltas with the first snapshot`()
- `Trace actual cached admission, read references and frontend generation ordering`()
- `Report remaining blockers without inheriting central test results`()

# Details

- 用户授权修复后重新审查；本任务只读，不参加实施或构建。
- 第二固定快照：`file:///tmp/kodex-closure-delta-review-20261007/`。
  1292 文件；manifest SHA-256
  `13a0843b6d57557bf1c98ad2ad940b77015a91c5469b8a130a38324ca513894f`。
  `DELTA_FROM_FIRST_REVIEW.json` 列 37 个增量路径，第一快照不改写。
- 独占：缓存不确定结果的 fail-closed 准入、共享读引用与每目录获取协调、
  repository 初始扫描、draft nonwritable/History nonce、CLI 日志失败清理，
  空 fixture 与 Koin marker 退役、fixture 关闭和测试元数据修正。
- 仅编辑本报告；不构建/修改源码、不操作 Home/设备/进程或提交。列准确
  contract/真实 factory/caller、锁顺序、primary/结果与残余复现门槛。
- 原复审未通过结论作为历史证据，不由实现者删除；新修复需独立判断。

## Independent report

### Verdict and evidence boundary

- Independent reviewer, not an implementer. **No newly confirmed B1 in the
  reviewed non-Native delta.** The first reports' cache-admission, abandoned
  shared-reference, logging-primary and failed-scan-child defects have actual
  static repairs, not just renamed fields or relaxed tests.
- S1's nonwritable draft result and S2's **queued nonce `Invalidate`** ordering
  have supported repairs. This does **not** establish that every destructive
  History producer is generation-safe: **B2-H1** below retains a specific
  `ExternalWriteFinished` interleaving. The original pointer-cancellation
  suspicion remains separate.
- `R` = independently reviewed static repair/call chain/test intent;
  `B1` = confirmed defect, including static proof; `B2` = concrete suspicion
  requiring its stated gate; `D` = retained debt/limit; `U` = unverified execution.
  An `R` is not an independently executed test pass.
- The user reports centralized **54 lease / 109 Session / 45 History / 4 CLI /
  85 RPC** green results. I did not execute or independently verify those
  test-result artifacts. They are **reported central evidence**, not my passes.
- This report does not approve all 203/206 projects, Native actuals, all Home/
  State fault paths, or the complete migration task. Native terminal/actual
  acceptance remains with its separate reviewer; common/JVM/JS adaptations
  below are checked for non-Native consistency.
- Only this report was written with `apply_patch`. No source fix, other document/
  report edit, build, IDE, runtime/FS fixture, child-process, device, network,
  user-Home, commit/push/branch operation. Read-only hashing/diff/source commands
  created no task temporary files or retained resources. Shared snapshots belong
  to the coordinator and remain available to the other reviewer.

### Provenance and exact reviewed inventory

- Current code exclusively:
  [second fixed snapshot](file:///tmp/kodex-closure-delta-review-20261007/).
  Independently rehashed **1292/1292 entries**, zero missing/mismatched files.
  Manifest SHA-256:
  `13a0843b6d57557bf1c98ad2ad940b77015a91c5469b8a130a38324ca513894f`.
  Only unlisted artifacts are the two expected
  `REVIEW_SOURCE_MANIFEST.json` and `DELTA_FROM_FIRST_REVIEW.json`.
- Independently checked the first snapshot: **1290/1290 matched**;
  manifest SHA-256
  `3d661ec7abdf3e33a8de2b103ad7a19b8504cfd2b8a1168f00980fbd7765335e`;
  its sole unlisted artifact is its manifest.
  Recomputed the manifest-map difference: **exactly the same 37 paths and
  previous/current hashes** as the delta artifact, including additions/deletions.
- Source links below are **original `file:///.../BuildKodex/Kodex/` URIs**.
  Their line numbers refer to second-snapshot bytes. Replace that URI prefix
  with `file:///tmp/kodex-closure-delta-review-20261007/` to read the fixed file.
  Deleted marker/coordinate descriptions refer to the first snapshot,
  not files claimed to exist in the current tree.
- Before/after comparisons use those two frozen snapshots. Fixed Git history
  is only `6b7129fa0e719dde8f88a0034b9d572679984a03`; no dirty working-source
  substitution. Independently compared the **16 frozen migration paths/bytes**
  against that Git object: all identical, no frozen path in this delta.
- Loaded project/associated skills, Draft, main executable plan, this task,
  lifecycle/Home/State/frontend/spec-impl and RPC Session/settings guidance,
  and the four first-round reports. Those reports are historical findings,
  not automatically inherited acceptance.

Finding lineage (the earlier reports remain unchanged):

| First-review finding | Current scoped disposition |
| --- | --- |
| [Lease/State B1 cache stale authority:78][first-storage] | R — original cache lock and fail-closed admission |
| [Lease/State B2 lost decrement:120][first-storage-ref] | R — synchronous accounting, bounded physical release |
| [Lease/State B3 logging A lost:155][first-logging] | R — actual startup caller/helper; real-Home double-fault U |
| [Backend B1 failed-scan child][first-backend] | R — scan before attachment, real factory retry test |
| [Frontend S1/S2:258–291][first-frontend] | R — conditional draft edit / nonce Invalidate ordering; B2-H1 is a different producer |
| [Platform B2-N1:203][first-platform] | R — non-Native exact raw-session consumer trace; actual/platform verdict separate |

All **33 non-Native delta paths** were examined, grouped below. Counts include
deleted files; the remaining four paths are Linux Curl source/test and POSIX/
MinGW Process actuals, reserved for Native review.

| Delta group | Exact file basenames/topics reviewed | Count / disposition |
| --- | --- | --- |
| Cached Session/State | `CachedAgentStorage.kt`, its test, `FileSystemKodexSessionRepository.kt`, its test, State contract `KodexAgentState.kt` | 5 — R admission/scan; D/U below |
| CLI startup | `Main.kt`, new `LoggingStartup.kt`, new `LoggingStartupTest.kt` | 3 — R primary; diagnostic/real-Home U |
| History | VM, spec, `HistoryViewportAndScrollEffectTest.kt`, viewmodel build, deleted `HistoryViewModelKoin.kt` | 5 — R nonce/identity; B2-H1 |
| New Session marker | deleted `NewSessionViewModelKoin.kt` | 1 — R responsibility-free removal |
| Catalog assertion | `SessionCatalogPopupTest.kt` | 1 — R test-only exact-cause assertion |
| Draft/source/fixture | `RpcSessionDraft.kt`, `RpcSessionSettings.kt`, `DraftAndSettingsBoundaryTest.kt`, `RpcFrontendFixture.kt` | 4 — R original lock/fixture lifetime |
| Empty test coordinate | deleted host-test-support spec build; OpenAI client build | 2 — R direct test dependency |
| Shared read | `FileSystemReadLeaseImpl.kt`, new `FileSystemSharedReadAccountingTest.kt` | 2 — R accounting/retirement; 8 gate sources |
| Direct Process adaptations | common impl, spec, acquisition test, IO test, JVM actual, Node actual | 6 — R exact-session barrier; faults U |
| Pipe/recording adaptations | `PipeProcessSession.kt`, JVM lifetime test, new Node barrier test, `McpStdioContractTest.kt` | 4 — R consumer/fixture conformance |
| **Total** | No unreviewed non-Native delta path omitted | **33** |

### 1. Cache uncertainty — historical B1 statically repaired

- FQCNs: internal
  `io.github.stream29.kodex.agentsession.filesystem.CachedAgentStorage` /
  `CachedIndexVersionedImpl`; original State spec
  `io.github.stream29.kodex.agentstate.contract.KodexAgentState`.
  Actual chain remains repository `open` → [Session factory][session-factory]
  → `FileSystemAgentStorage.cached` → original State/runtime → real
  [State settings CAS][state-cas] → cache `set` →
  [filesystem `setUnsafe`][fs-write]. No fake CAS/second storage owner.
- **Before:** delegate publication and rescan were outside the cache write
  session. A failed rescan returned A/B but retained authoritative-looking old
  indexes; actual CAS could reuse and replace durable settings `1.json`.
  **After:** [cache:145–151,283–329][cache] has one `unresolvedFailure`, read and
  written only under the **original indexes `SafeRw`**. The write session covers
  append validation, delegate write, failure marking, bounded reconciliation
  and metadata publication. A failed numbered-record scan leaves admission
  closed until this Session/cache is closed and rebuilt.
- `latestIndex()`, floor/ceil, get/getExact, indexes/values all check authority
  under read sessions; even empty ranges cannot bypass it. Value reads retain
  that session through `getUnsafe`/cache loading. `set` and `revert` check under
  the write session before touching delegate bytes. Last-known `latestIndex`
  and `cacheNonce` flows remain observations, **not writable authority**.
  Corrected [State KDoc:145–162,313–338][state-spec] says that and documents
  the settings guards' `IllegalStateException`.
- A stays the original delegate Throwable; B is suppressed on A. Guard errors
  intentionally have **no cause pointing back to A**. State's follow-up
  tail-read can suppress that guard on A without forming A→guard→A.
  No implicit rescan/retry, rollback, new exception type or repository poisoning.
- Successful reconciliation clears uncertainty only after obtaining actual
  numbered indexes and replacing the cache. It invalidates the attempted
  value, preserves append nonce, then still throws A; it does not return CAS
  success for a partially failed write.
- Lock trace: State write Mutex → cache `SafeRw` → delegate filesystem calls.
  Reconciliation mutates its supplied list directly rather than reacquiring
  `SafeRw`; delegate enumeration/read/write do not call back into the cache.
  `valuesIn` computes its range locally rather than calling `indexesIn` under
  the held reader. No introduced reader→writer upgrade or RW reentry found.
  [Original SafeRw][safe-rw] remains a real production boundary.

**Test-source evidence, not execution:**

- [Timeline test:44–118][cache-test]: isolated system FS actually moves
  `5.json`, faults temporary deletion with A and gates subsequent list failure B.
  Queued read/append/revert remain incomplete while the write/reconciliation
  session is held, then reject. Exact A, B suppression, all query guards,
  last-known flows, no implicit scan, unchanged raw `5.json`, absent `6.json`
  and successful rebuilt-cache append are asserted. Fault wrapper at 462–503
  delegates real IO; it is not a fake storage result.
- [Repository test:97–194][repo-test] exercises **actual cached Session/runtime
  CAS**: publish settings 1, fail delete A/list B, reject stale/equal/durable
  expected-value CAS after one-shot faults disappear; preserve raw settings 1
  and absent settings 2/timestamp 1. Healthy sibling still updates under the
  same live repository owner. Closing **only the affected Session** and
  reopening reloads durable settings 1, rejects the old baseline and appends
  settings 2 normally. Wrapper: 894–937.
- Both suites parameterize IO and **injected CE**, not actual caller-Job
  cancellation in this new double-fault case. They check A identity and B
  suppression, not a separate whole-exception-graph cycle traversal. No-cycle
  support comes from guard construction just traced.
- Existing successful rescan, normal sparse queries/append/revert/nonce and
  metadata tests remain. **U:** actual cancellation after OS move, scan timeout,
  and active RequestResponse/Compacting double-fault combinations are not
  independently executed or all covered by these new cases.

### 2. Shared read — exact accounting separated from physical cleanup

- Original public `io.github.stream29.kodex.utils.filesystemlease.FileSystemLease`
  [contract:7–29][lease-spec] and `CoroutineScope.FileSystemReadLease` factory
  are unchanged. New `SharedReadInventory`, slot/entry and typed owner are
  **private in-memory accounting**, not a public manager, registry service,
  resource scope or parallel lease implementation.
- **Before:** one global Mutex surrounded all keys' filesystem acquisition;
  last-handle cleanup could time out waiting for it **before decrement**.
  Repeated close replayed the timeout, leaving a renewable reference.
  **After:** [read implementation:31–175][read-lease] CAS updates an immutable
  inventory of key pins/current-owner identity/per-owner references. Transforms
  allocate/copy memory, inspect Jobs/identity and return values; no suspend, IO,
  launch, close/join, handle construction or callback registration occurs
  inside a transform that can retry.
- Actual order:
  - synchronous CAS pin → that resolved key's acquisition Mutex;
  - CAS select/reserve → old exact raw `closeAndJoin`, if retiring;
  - original acquisition guard/owner publication → CAS install;
  - completion callback registration/handle construction → unlock → synchronous
    CAS unpin in `finally`.
  - release: synchronous CAS decrement → only for zero, bounded raw cleanup →
    identity/completion-checked retirement. **Release never takes the key Mutex.**
- Live same-parent duplicate reserves a reference before constructing its
  original handle. Another active parent gets `FileSystemLeaseInUseException`.
  Cancelled mutex waiter only drops its pin. Post-construction failed handoff
  closes/awaits that handle, whose single lifetime `finally` decrements exactly
  once. There is no timeout around that decrement.
- Pins include waiting/cancelled acquisitions until synchronous `finally`.
  `withSlot` removes a key only with zero pins and no owner. New same-key pin
  and retirement/unpin races linearize on the same inventory: they cannot
  produce two live acquisition Mutexes for one resolved key.
- Zero-reference/lost owner remains the slot's retirement barrier until its
  **raw lease Job actually completes**. `removeRetired` also requires
  `slot.owner === owner` and no remaining reference entry. New acquisition
  cannot publish at that path while old cleanup runs. A lost old owner's
  remaining handles stay independently counted by old object; their later
  zero cannot remove a replacement owner.
- Completion can invoke `removeRetired` inline, including immediately at
  callback registration. It performs only a fresh CAS transaction, not
  acquisition/IO. If nested/completing work changes the inventory before an
  outer CAS succeeds, the outer transform retries from the new snapshot.
  Retirement-before-last-release is harmless: later release removes references
  and explicitly rechecks retirement. Static transitions remove zero-count
  reference entries and idle slots; empty child lists alone would **not** prove
  private-map invariants.
- [Handle:205–245][read-lease] retains one owner-bound `UNDISPATCHED` lifetime,
  one saved `Result` latch, no detached release Job. Owner-loss ends its reference
  child, not parent. Failure identity/primary handling remains in the original
  raw lease/cleanup helper.
- [Support:24–47,190][lease-support] and raw lease are unchanged between
  snapshots. Physical cleanup is still **10 seconds**, close-wait **30 seconds**;
  this repair did not silently change physical cleanup to 30 seconds.
  Accounting never deadline-abandons a decrement; cooperative physical timeout
  remains an observable saved failure. Non-cooperative host calls are not
  forcibly interrupted by coroutine deadlines.

All eight new [real-FS gate test sources][accounting-test] were checked:

| Lines | Exact gate/test responsibility |
| --- | --- |
| 47–94 | `unrelated gated B publication cannot block A last release or abandon its reference`: A closes/reacquires/closes within **5 real seconds while B remains gated**; repeated close, raw files and live parent checked |
| 96–147 | `same-key concurrent acquisition shares one original owner and competing parent rejects`: one publication/raw identity, exact non-final/final semantics |
| 149–196 | `cancelled same-key waiter consumes no reference and leaves no orphan or idle owner job`: caller cancellation, live owner, later reacquisition |
| 198–239 | `cancelled receiver waiting on a contended key cannot publish or inherit another owner's lease`: receiver differs from still-live caller |
| 241–284 | `cancelled publication with same-key contender cleans undelivered owner before replacement`: real post-publication suspension, old deletion before next publication |
| 286–336 | `physical cleanup keeps its real deadline and saved timeout without retaining a shared reference`: real delete awaits cancellation; **15-real-second test bound** catches removing/raising the 10-second bound; exact repeated timeout and dead raw Job checked |
| 338–395 | `one-to-zero concurrent reacquire waits for old physical release and repeated old close cannot delete new owner`: new parent/key waiter blocked on actual old deletion |
| 397–471 | `owner-loss concurrent reacquire waits for retiring job and never deletes or reuses foreign read owner`: real foreign bytes/gated cleanup read; only renewal triggering uses virtual time |

- Headers cache previously resolved directories solely to position same-key
  waiters; guard/read-owner IO stays real. Timeout case deliberately retains
  failed-cleanup bytes, verifies acquisition rejects them, then deletes **its
  own fixture file** to demonstrate fresh admission. That is not production
  recovery or a foreign-owner sweep.
- **D/U:** no direct private-inventory size assertion or multithread CAS/
  callback stress execution here. Exact-owner read→delete is not atomic
  compare-and-delete; replacement **between** syscalls remains first-report
  protocol debt. All eight sources plus unchanged ordinary/lifecycle/process/
  Home regressions need central results matched to this snapshot; I ran none.

### 3. Real repository failed initial scan — historical Backend B1 repaired

- [Factory:469–489][repo-factory] computes fallible `initialEntries` before
  evaluating `supervisorChildScope()`. Previously named-argument evaluation
  attached that child first; failed lazy scan left it under a surviving,
  retryable Backend owner.
- Actual chain unchanged: `withKodexApplication` → `withBackendServices` →
  `BackendSessionHost.repository()` acquisition Mutex → original
  `CoroutineScope.FileSystemKodexSessionRepository`. It constructs the same
  repository with real receiver child; no extra Wrapper/Provider/second lazy
  repository. Root directories may already exist on failure; the fix concerns
  **unreturned Job attachment**, not filesystem transaction/rollback.
- [Test:195–221][repo-test] uses real FS factory, same parent, twice-failing
  sessions `list`, exact IO A, `owner.children.none()` after each failure,
  then healthy same-parent retry/list/close and no remaining child. This
  targets the first finding, unlike fake factory failure before child creation.
- **U:** test injects scan IO, not scan caller cancellation or encoded whole-host
  retry. Static evaluation order protects scan failures; unrelated partial
  constructor/OOM faults are not blanket-certified.

### 4. CLI logging startup — primary retained before Application ownership

- Actual entry [Main:10–24][main] calls
  `io.github.stream29.kodex.cli.app.initializeCliLogging(homeHandle)` before
  `withKodexApplication`. [Internal helper:13–36][logging] owns no logger/Job.
  On init A it awaits [actual Home `closeAndJoin` contract:63–78][home-spec] under NonCancellable,
  with Default dispatcher return **inside** that mask and finite 30-second wait;
  distinct cleanup B is suppressed, then original A reaches reporter.
  Success does not close Home.
- **Before:** direct `home.closeAndJoin()` could throw B/stop on caller
  cancellation before A's diagnostic. **After:** cleanup is a Result inside
  cleanup context, so cleanup throw/prompt dispatcher-return cancellation
  cannot replace A.
- Main prepares **default** `KodexHome`, so helper `initialize(home.home)`
  resolves to the same default root in production. This is not authority to
  redirect process logs with a future custom data-root entrypoint.
- [Four tests:22–111][logging-test] cover injected IO + cleanup IO, injected
  CE + cleanup IO, **actually cancelled caller Job** with gated cleanup IO,
  and success retaining Home. Actual-cancel test uses `awaitCancellation`,
  `operation.cancel`, awaits cleanup start, proves operation incomplete until
  release, then joins/checks reported CE/B.
- These are **fake Home-handle injection tests around the actual helper**,
  not real filesystem `prepareKodexHome` + logging-init + Home-delete
  double-fault execution. Earlier real Home lease tests are separate evidence.
- **D/U diagnostic limit:** default reporter prints only A's message, not
  suppressed B/stack trace. A/B are retained for an injected/other reporter,
  not newly made user-visible in that line. No new logger owner/Failure VM
  warranted. Reporting-sink failure/30-second timeout branch remain untested.

### 5. Frontend S1 — rejection inside original draft mutation lock

- FQCNs remain `io.github.stream29.kodex.cli.rpc.RpcSessionDraft`,
  `RpcDraftSettingsSource` and original
  [SessionSettingsDataSource spec:51–90][draft-spec], whose configuration command
  requires nonwritable/field-conflict false and caller/source cancellation CE.
  Application → [original factory:14–35][new-session-factory] → `RpcNewSessionViewModel`
  supplies draft local owner; persisted backend remains borrowed.
- **Before:** pre-lock editable check passed, source waited behind materialize,
  then public `draft.edit` threw “no longer editable”. **After:**
  [draft:82–100][draft] adds only internal conditional `tryEdit` under **the same
  Mutex**. Materialized/closed returns false there; nullable transform rejects
  field conflict without rewriting. `applyEdit` shares existing explicit-name/
  settings publication. Public late `edit` remains throwing.
- [Source:156–205][draft-source] retains revision admission → draft Mutex.
  Requested changed-field baseline is compared with **current locked draft**
  settings; unrelated model/tier/instructions/title survive. Source-owner loss
  rejects; caller cancellation waiting on Mutex remains CE. No broad
  IllegalStateException catch, retry, public conditional-editor API or new owner.
- [New tests:162–302][draft-test] gate actual `createSession` while
  `materialize()` holds original lock. Direct source and VM FIFO updates stay
  pending with VM/source alive, then return false/no reporter failure after
  successful materialization. Require unchanged local settings, unavailable
  source afterwards, public late-edit failure and actual backend settings.
  Separate case cancels **caller** waiting on lock and requires CE/no late edit.
- Metadata assertion now expects real backend `Session <index>`, nonblank
  generated turn/first-window IDs, `windowId == firstWindowId`, and equality
  against `initial.copy(those backend fields)`, checking **all remaining caller
  fields**. One case also reads actual settings RPC timeline. It does not
  equate initialized settings with uninitialized input DTO.
  [Backend create:49–50,148–154][backend-create] and storage initialization are
  generated-field authority, not the decorator.
- Existing delayed observer at 78–120 checks same-model conflict false and
  unrelated tier/instructions/title merge. Teardown explicitly runs paused
  scheduler cancellation before join. New materialize VM reporter continuation
  drains before assertion. These are scheduling/assertion corrections, not
  alternate CAS behavior.
- **U:** broader multi-field/rename/cwd delayed-observer and root caller-cancel+
  transport-fault combinations are not all supplied by this delta.

### 6. Frontend S2 — nonce ordering repaired; additional producer gate open

- Original spec
  `io.github.stream29.kodex.app.history.contract.AgentHistoryViewModel` /
  `HistoryScrollEffect` [124–130,207–216][history-spec]; factory
  `io.github.stream29.kodex.cli.history.createAgentHistoryViewModel`
  [908–921][history]. Production [RpcAgentViewModel:108–112][history-caller]
  supplies same binding's storage/latestIndex/state/index nonce and dedicated
  local child. No `AgentHistoryAggregate`, nonce resolver/framework manager.
- **Before:** nonce observer withdrew old intent but enqueued untyped
  `Invalidate`; retained new intent ran first, then unconditional replacement
  withdrew new effect. **After:** `HistoryNavigationIntent` carries nonce plus
  referential identity; Invalidate/paging capture generation.
  `synchronizeGeneration()` runs **before channel commands and retained
  navigation**, so new-nonce replacement precedes new-nonce intent.
- [History:104–109,234–273,480–539,759–803][history] preserves new-generation
  intent/effect, ignores late nonce Invalidate already synchronized, rejects old
  paging, rechecks nonce/closed after suspending reads before materialization/
  publication. Item context retains captured generation. One `pendingNavigation`
  latest-only slot remains; channel is bounded wake/ordered internal commands,
  not a journal.
- No-nonce fallback still increments window generation on destructive
  invalidation and withdraws prior intent. Identity-based window/viewport/
  exact-child and exact-effect CAS acknowledgment remain. Repeated destinations
  create distinct effects; close withdraws/ends local work. Renderer unmount
  does not consume VM effect.
- [New test:110–208][history-test] blocks VM `getExact(7)`, saturates 256 mixed
  requests, drives **actual nonce collector**, submits new Latest, and uses FIFO
  turn-duration barrier behind Invalidate before final assertion. Checks final
  nonce/20 child/Latest, old lazy child's cancelled Job/stale timestamp rejection,
  new child loading and stale ack not consuming final effect. Scheduler is
  explicitly both dispatcher **and context scheduler**, avoiding inherited
  scheduler mismatch.
- This is actual VM + **InMemory repository/supplied nonce flow**, not real FS/
  RPC nonce propagation or mounted UI. Retained tests at 209–342 cover repeated
  targets/no mount/close/no-nonce revert; 395–430 protects **exact Work Group
  instance and structural chunk**, not merely 19 child count. Renderer late-
  mount/effect tests are unchanged and unrun here; preservation is not a new pass.

#### B2-H1 — delayed ExternalWriteFinished may invalidate newer navigation

- Separate concrete producer [History:151–168][history] queues
  `ExternalWriteFinished(startIndex,endIndex)` with **no generation**.
  After synchronization, [811–822][history] can call
  `replaceWindow(command.endIndex, invalidate = true)` for destructive operation.
  Synchronization does not record that operation in `lastInvalidation`;
  already-current nonce replacement reaches [497–499][history] with no preserved
  generation and withdraws current navigation/releases current children again.
- Suspected sequence: blocked old seek + saturated buffer; actual destructive
  `runtime.modify/revert` changes cache nonce/emits ExternalWrite completion;
  new-nonce Latest is processed after earlier wake synchronizes nonce; effect
  publishes; delayed ExternalWriteFinished invalidates **same current nonce**
  again. Typed nonce Invalidate is fixed, but this other producer is not
  generation-tagged/deduplicated by the fix.
- **Not an observed failure/new B1 claim.** Minimum central gate: extend
  saturation case with cached source/real destructive operation (or first
  isolate actual VM state producer with controlled flows); hold completion
  command behind saturated wakes, release reads, then use FIFO barrier **after
  ExternalWriteFinished**, not only nonce Invalidate. Require new nonce/exact
  child/effect pending until exact ack; old child cannot reload.
  Preserve no-nonce destructive rewrite behavior—deleting that command loses
  a genuine responsibility.
- Smallest candidate is reconciliation/deduplication in original History owner
  with existing generation/operation information. No resolver, renderer
  workaround, retry or second authority justified.

### 7. Fixture close and responsibility-free removals

- [Catalog assertion:137–145][catalog-test] replaces collection equality with
  one reported Throwable and **exact deepest cause identity**. Opened target,
  retained error state and no-dismiss assertions remain. Type/coroutine
  recovery assertion correction, not product catch or “any failure passes”.
- [RpcFrontendFixture:74–120][frontend-fixture] attaches actual
  `frontendJob = SupervisorJob(connection Job)` for views/registry/drafts.
  Normal `finish` runs `sessions.shutdown()` then `frontendJob.cancelAndJoin`
  **before returning RPC connection block**, whose structured normal completion
  otherwise waits forever for live draft children.
- Backend owner spans entire fixture connection, then `withBackendServices`
  exits and Home closes. Draft/Session frontend closure does not close borrowed
  backend. Retained draft parent remains real; not a rollback to unparented
  product scopes. Production [Application:162–175][application] already closes
  frontend owner before surrounding RPC teardown.
- **U/D:** normal retained-draft close is statically corrected; unusual
  synchronous construction failure before new inner try/finally and fixture
  cleanup double-fault diagnostics are not certified by the change. Fixture
  is test support, not production owner API.
- Deleted `utils/host-test-support/spec/build.gradle.kts` only exported
  `libs.ktor.client.mock`, with no code/fixture. [OpenAI build:20–25][openai-build]
  now directly depends on that library in commonTest. No snapshot consumer
  still names old coordinate. Meaningful ablation, not missing spec.
- Deleted History/New Session Koin files were annotation-scanned marker classes
  with no assembly consumer in either snapshot. Original factories remain;
  snapshot-wide source/build search finds no marker/`AgentHistoryAggregate`
  reference. History build adds only commonTest `libs.kotlinx.coroutines.test`
  for controlled scheduler, not framework state leaking into spec.

### 8. Remaining non-Native Process/consumer adaptations

- Shared-contract consistency check for other lane, not Native certification.
  FQCN `io.github.stream29.kodex.utils.processclient.ProcessSession`
  [spec:37–61][process-spec] adds **original resource's** `closeAndJoin`:
  exact streams/process observer, no client/sibling join, cleanup despite
  cancelled caller, bounded cooperative waits, saved outcome.
- [Acquisition:69–114][process-common] awaits exact session on unclaimed
  rollback instead of only `close`; retains exit wait/primary suppression and
  owner-bound acquisition child. Recording MCP/acquisition/JVM gated fixtures
  implement same contract; synchronous recording close needs no artificial Job.
- [JVM:62–130][process-jvm] saves actual release Result, attempts termination/
  streams independently, joins exact cancellation guard/exit observer within
  NonCancellable IO/7-second wait. Racing close sees saved result. Blocking
  host `waitFor`/stream calls remain non-forcibly-interruptible.
- [Node:192–200][process-node] exposes existing raw guard/result under
  NonCancellable/7-second wait, never client join. [Pipe:113–125,226–255][pipe]
  awaits it on explicit close and session-end cleanup. Statically addresses
  first platform report's **B2-N1 exact raw barrier**; original session primary
  gets cleanup suppressed.
- [Process IO test:30–52][process-test] starts real direct siblings; repeated
  exact close must release streams/exit observer without ending client/owner/
  sibling. [Node tests:20–143][node-test] cover actual session-only cancel,
  normal close, unclaimed Shell handoff with real ChildProcess/gated **close-
  event delivery**, plus separate fake cleanup-failure case. Real gate checks
  destroyed streams/live parent, not numeric exit as proxy for raw cleanup.
- Node prototype interception at 182–223 is restricted to unique spawn marker,
  released/restored in finally; tests still require isolated execution to avoid
  global-prototype fixture interaction. Native actuals/faults belong to other report.
- **U:** OS force/stream/constructor faults, taskkill/helper timeout precedence,
  cancelled direct-client waits and platform repeated-timeout identity are not
  all established. Public contract does not make synchronous cleanup interruptible.

### Retained debts, specific gates and minimum ablation

- **B2-P1, inherited:** post-OS-record-move caller cancellation may roll back
  `latest.json` to valid older record while retaining newer numbered record.
  [setUnsafe:233–239][fs-write] and repository valid-pointer fast path unchanged;
  fail-closed admission does not repair that pointer. Gate actual move-before-
  dispatcher-return cancellation; inspect record/pointer/cache/catalog separately.
  Injected temp-delete CE is not that experiment.
- **D-State/lease:** first report's Session/Home lease-loss fencing topology,
  general State recovery-primary/phase gaps and non-transactional timeline writes
  are outside this repair. Owner identity before cleanup read is protected;
  inter-syscall foreign replacement is not.
- **D-doc:** Home SOP no-extra-release-method text and State SOP old transaction/
  captured-checkpoint description still conflict with repaired actual contracts.
  This report does not rewrite checklists; don't use stale wording to reject
  saved-result method or claim stronger transaction guarantees.
- Minimum meaningful ablation:
  - Remove uncertainty or move cache delegate IO outside SafeRw: stale admission
    returns. Retain both; no cache-health manager.
  - Remove exact accounting, per-key pins or completion/identity retirement:
    abandoned release, split locks or old/new owner race returns. A simpler
    single Mutex **with IO outside and equivalent reservations/retirement**
    could satisfy same responsibility; atomic inventory isn't an ideal-framework
    mandate.
  - Remove conditional draft edit/changed-field checks: false→throw after
    materialize/delayed lost edit returns. No public conditional editor or second CAS.
  - Remove History generation/identity guards: late withdrawal/stale child
    reuse returns. Keep one intent; B2-H1 needs existing producer reconciliation,
    not navigation service.
  - Remove CLI masked Result/primary capture: logging A loss returns. Helper
    can be inlined at same actual callsite; it is internal testability, not owner.
  - Remove fixture child without exact pre-connection cancellation: normal-close
    wait on drafts returns. Keep real lifetime; empty coordinate/Koin markers
    can stay removed.
- **Handoff:** ready for asynchronous reading; no wait for builds/other Sessions.
  Credit static repairs narrowly. Resolve or explicitly defer B2-H1/P1, and
  match central runtime evidence to this manifest; changed source needs new
  hash-identified delta review. No whole-repository approval.

### URI references

[first-storage]: file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-07-recheck-lease-state-closure.md#L79
[first-storage-ref]: file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-07-recheck-lease-state-closure.md#L123
[first-logging]: file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-07-recheck-lease-state-closure.md#L158
[first-backend]: file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-07-recheck-backend-protocol-closure.md#L136
[first-frontend]: file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-07-recheck-frontend-contract-closure.md#L261
[first-platform]: file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-07-recheck-platform-resource-closure.md#L206
[home-spec]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/migration/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/migration/KodexHomeHandle.kt#L63
[draft-spec]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-settings/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/settings/contract/SessionSettingsViewModel.kt#L51
[new-session-factory]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/new-session/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/cli/newsession/NewSessionViewModel.kt#L14
[cache]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-session/impl/filesystem/src/commonMain/kotlin/io/github/stream29/kodex/agentsession/filesystem/CachedAgentStorage.kt#L145
[cache-test]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-session/impl/filesystem/src/commonTest/kotlin/io/github/stream29/kodex/agentsession/filesystem/CachedAgentStorageTest.kt#L44
[repo-factory]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-session/impl/filesystem/src/commonMain/kotlin/io/github/stream29/kodex/agentsession/filesystem/FileSystemKodexSessionRepository.kt#L469
[repo-test]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-session/impl/filesystem/src/commonTest/kotlin/io/github/stream29/kodex/agentsession/filesystem/FileSystemKodexSessionRepositoryTest.kt#L97
[session-factory]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-session/impl/filesystem/src/commonMain/kotlin/io/github/stream29/kodex/agentsession/filesystem/FileSystemKodexAgentSession.kt#L27
[state-cas]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-state/impl/state/src/commonMain/kotlin/io/github/stream29/kodex/agentstate/impl/KodexAgentStateImpl.kt#L438
[state-spec]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-state/spec/contract/src/commonMain/kotlin/io/github/stream29/kodex/agentstate/contract/KodexAgentState.kt#L145
[fs-write]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-storage/impl/filesystem/src/commonMain/kotlin/io/github/stream29/kodex/agentstorage/filesystem/FileSystemIndexVersioned.kt#L216
[safe-rw]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/read-write-mutex/impl/src/commonMain/kotlin/io/github/stream29/kodex/utils/SafeRw.kt#L8
[lease-spec]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/filesystem-lease/spec/src/commonMain/kotlin/io/github/stream29/kodex/utils/filesystemlease/FileSystemLease.kt#L7
[read-lease]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/filesystem-lease/impl/src/commonMain/kotlin/io/github/stream29/kodex/utils/filesystemlease/FileSystemReadLeaseImpl.kt#L31
[lease-support]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/filesystem-lease/impl/src/commonMain/kotlin/io/github/stream29/kodex/utils/filesystemlease/FileSystemLeaseSupport.kt#L24
[accounting-test]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/filesystem-lease/impl/src/commonTest/kotlin/io/github/stream29/kodex/utils/filesystemlease/FileSystemSharedReadAccountingTest.kt#L38
[main]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/cli/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/Main.kt#L10
[logging]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/cli/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/LoggingStartup.kt#L13
[logging-test]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/cli/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/app/LoggingStartupTest.kt#L22
[draft]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/rpc/src/commonMain/kotlin/io/github/stream29/kodex/cli/rpc/RpcSessionDraft.kt#L82
[draft-source]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/rpc/src/commonMain/kotlin/io/github/stream29/kodex/cli/rpc/RpcSessionSettings.kt#L156
[draft-test]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/rpc/src/commonTest/kotlin/io/github/stream29/kodex/cli/rpc/DraftAndSettingsBoundaryTest.kt#L162
[backend-create]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/server/src/commonMain/kotlin/io/github/stream29/kodex/rpc/server/BackendSessionManagement.kt#L49
[history]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/cli/history/AgentHistoryViewModel.kt#L104
[history-spec]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/history/contract/AgentHistoryViewModel.kt#L124
[history-caller]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/rpc/src/commonMain/kotlin/io/github/stream29/kodex/cli/rpc/RpcAgentViewModel.kt#L108
[history-test]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/impl/viewmodel/src/commonTest/kotlin/io/github/stream29/kodex/cli/history/HistoryViewportAndScrollEffectTest.kt#L110
[catalog-test]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-catalog/impl/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/sessioncatalog/SessionCatalogPopupTest.kt#L137
[frontend-fixture]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/test-support/rpc/src/commonMain/kotlin/io/github/stream29/kodex/app/test/RpcFrontendFixture.kt#L74
[application]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/application/src/commonMain/kotlin/io/github/stream29/kodex/cli/app/Application.kt#L162
[openai-build]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/impl/client/build.gradle.kts#L20
[process-spec]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/process-client/spec/src/commonMain/kotlin/io/github/stream29/kodex/utils/processclient/ProcessClient.kt#L37
[process-common]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/process-client/impl/src/commonMain/kotlin/io/github/stream29/kodex/utils/processclient/ProcessClient.kt#L69
[process-jvm]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/process-client/impl/src/jvmMain/kotlin/io/github/stream29/kodex/utils/processclient/JvmProcessClient.kt#L62
[process-node]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/process-client/impl/src/jsMain/kotlin/io/github/stream29/kodex/utils/processclient/NodeProcessClient.kt#L192
[pipe]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/shell-client/impl/src/processClientMain/kotlin/io/github/stream29/kodex/utils/shellclient/PipeProcessSession.kt#L113
[process-test]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/process-client/impl/src/commonTest/kotlin/io/github/stream29/kodex/utils/processclient/ProcessClientIoTest.kt#L30
[node-test]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/shell-client/impl/src/jsTest/kotlin/io/github/stream29/kodex/utils/shellclient/NodePipeCleanupBarrierTest.kt#L20

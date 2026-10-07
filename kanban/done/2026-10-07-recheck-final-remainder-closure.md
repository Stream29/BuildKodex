# REVIEW READY — eight-path source repairs supported; VIEW stress and residual gates not accepted

- 审查工作已归档；结论固定在第三快照，不覆盖后来两个测试修改。
  当前中央证据见 [主任务](2026-10-07-plan-spec-impl-closure-and-consistency.md#final-scoped-closure-evidence)。

# Task Tree

- `Independently review the eight final remainder deltas`()
- `Check real producer, terminal release and rollback consumers`()
- `Report scoped defects and unverified gates without inheriting test passes`()

# Details

- 用户授权修复后重新审查；独立只读，不参加实现或集中构建。
- [主任务](2026-10-07-plan-spec-impl-closure-and-consistency.md)，历史基线
  `6b7129fa`；当前只读取第三快照 `/tmp/kodex-closure-remainder-review-20261007`，
  1292 文件，manifest SHA-256
  `fff68f99f942a9bf201e26cf3ce852718a4df29f26e33757d3352cbdd7ec0c1e`。
  `DELTA_FROM_SECOND_REVIEW.json` 准确列出 8 个增量路径。
- 独占第三快照中的 History completion 生产者/消费者/规范与门闩、
  Curl 抑制图/terminal outcome、MCP constructor rollback 原 raw-session
  等待，以及两个实际测试作用域/泛型语法修正。源码不改、所有旧快照不改。
- 中央实际门槛：原 History VM 字节执行四个新门闩四项失败；恢复修复后
  History VM 50 / RPC 85 项通过。旧 1000-item View 压力用例再次超时，
  正在单列定位，不把这次追加命令写全绿。
- Curl Linux x64 22、MCP stdio JVM 8 / Node 8 及 composition JVM 27 通过；
  reviewer 不执行运行/资源操作，不把转述中央结果当自己的测试。
- 独立报告只写本文件。列确认缺陷、具体疑点与复现条件、支持的最小修复、
  不确定平台；源码引用使用原仓库 URI/行号并明确对应第三快照字节。
  只审 8 个增量，不宣布所有 203 项目行为全面通过。

## Independent final delta report

### Verdict, provenance and execution boundary

- Independent READONLY reviewer, not an implementer. **R:** the actual History
  completion producer/consumer, Curl cleanup aggregation and MCP constructor
  rollback have supported local repairs. **No newly confirmed B1 within their
  scoped fault/authoritative-source assumptions.** This is not whole-repository,
  all-platform, every Throwable graph or History VIEW-stress acceptance.
- Classification: **R** = independently checked source repair and test intent;
  **B1** = confirmed conditional control-flow defect, not necessarily an observed
  OS failure; **B2** = specific remaining reproduction/evidence gate;
  **D** = retained behavior/debt/limit; **U** = unverified execution.
  An R entry does not mean this reviewer executed its tests.
- Current source exclusively:
  [third immutable snapshot](file:///tmp/kodex-closure-remainder-review-20261007/).
  Independently hashed **1292/1292 manifest entries**, zero missing/mismatches;
  manifest SHA-256
  `fff68f99f942a9bf201e26cf3ce852718a4df29f26e33757d3352cbdd7ec0c1e`.
  Unlisted files are exactly its manifest and delta artifact.
- Before exclusively:
  [second immutable snapshot](file:///tmp/kodex-closure-delta-review-20261007/).
  Independently hashed **1292/1292 entries**, zero missing/mismatches;
  manifest SHA-256
  `13a0843b6d57557bf1c98ad2ad940b77015a91c5469b8a130a38324ca513894f`.
  Unlisted files are exactly its manifest and `DELTA_FROM_FIRST_REVIEW.json`.
- Recomputed the manifest-map difference: **exactly eight paths**, all existing
  files, with every `previous` and `current` hash matching
  [DELTA_FROM_SECOND_REVIEW.json](file:///tmp/kodex-closure-remainder-review-20261007/DELTA_FROM_SECOND_REVIEW.json).
  Other manifest paths were integrity-checked, not newly substantively accepted.

| Exact delta responsibility/path | Current reference | Disposition |
| --- | --- | --- |
| History impl VM `AgentHistoryViewModel.kt` | [producer L151–170; consumer L760–833][history] | R completion identity/cursor |
| History `HistoryViewportAndScrollEffectTest.kt` | [L212–425][history-tests] | R five added gates; execution attributed below |
| History spec `AgentHistoryViewModel.kt` | [L46–62,125–159,219–227][history-spec] | R actual completion and demand semantics |
| MCP stdio `McpStdioTransport.kt` | [L23–72][mcp] | R constructor-only rollback |
| MCP stdio `McpStdioContractTest.kt` | [L85–227][mcp-tests] | R gated fault/cancellation tests and test-only launch scope correction |
| Curl `KodexCurlMultiApiHandler.kt` | [L28–94,142–178,457–551][curl] | R local graph/terminal handling |
| Curl `KodexCurlTerminalCleanupTest.kt` | [L131–241,350–429][curl-tests] | R actual-handler gates and graph assertions |
| JVM Pipe `PipeProcessLifetimeTest.kt` | [L216–230][pipe-test] | R test-only explicit Result type |

- All source links below use original `file:///.../BuildKodex/Kodex/` URIs for
  navigation; **line numbers refer to third-snapshot bytes, not mutable code**.
  Replace that prefix with the third snapshot URI to read reviewed bytes.
  Before links explicitly identify the second snapshot.
- Loaded AGENTS guidance, associated change/checklist/kanban/planning,
  ask-user/document/workspace/toolchain/IDE skills, Draft, main executable plan,
  this task, spec/impl, frontend, lifecycle, native-IO, Curl, MCP and relevant
  History/State guidance. Loaded both previous independent delta reports:
  [storage/frontend READY][previous-history] and
  [Native terminal READY with closure BLOCKED][previous-native].
  Those reports remain historical and **were not rewritten**.
- Original repository was used only for fixed Git history/dependencies:
  `6b7129fa0e719dde8f88a0034b9d572679984a03` pins MCP SDK
  `53f717661712d618cd4822d1760392e903989ead`. Read that SDK Git object and
  cached Ktor/Coroutines source archives; no mutable Kodex source substituted.
- Only this report was written, with `apply_patch`. No code fix, other document/
  report write, test/build/Gradle, IDE, native OS/process/network fixture,
  resource control or Git commit/push/switch. Read-only hash/diff/source commands
  created no temporary review files or retained processes/resources.
  Coordinator-owned snapshots were neither modified nor removed.

### Central results are attributed, not reviewer test passes

- Coordinator reports original second-snapshot History VM was **byte-identical
  by cmp** when executing four new gates: **four failed**, then the final VM
  hash was restored to
  `5e6cd0f760c400a6c033e036cfbb8a77b3f6e29fea79d8d90159460a0b323367`;
  final History VM **50** and RPC **85** passed. The earlier green command
  accidentally already used repaired VM bytes; it is not the red experiment.
  This report independently checked the before/after bytes and test design,
  **not runtime logs or XML archives**. [Central handoff][central]
- Coordinator reports the appended History command **failed overall**:
  **71 VIEW tests, one old 1000-item pressure timeout**, Ready window
  **509..502** versus visible **508..503**. VM/RPC green does not erase it.
  **D/B2-VIEW:** root diagnosis remains in the separate History VIEW lane.
  Neither completion identity repair nor demand KDoc establishes its cause
  or fixes it. This report does **not** approve VIEW stress or all 203 projects.
- Coordinator reports actual Curl Linux x64 **22**, stdio JVM **8** / Node **8**,
  MCP composition JVM **27**, and Shell Node **37** passed; main plan also
  records JVM Shell **37** after the Pipe type correction. These are reported
  central evidence only, not independently executed/verified results.
  JVM recording-test green is not Node raw-guard fault execution.
- Any future History VIEW diagnostic source change lies outside these eight
  hashes. Main must freeze/compare and independently review that later delta;
  do not silently extend this verdict or wait for builds to publish this report.

### History: actual completion producer → original synchronization → consumer

- Real FQCN remains
  `io.github.stream29.kodex.app.history.contract.AgentHistoryViewModel`;
  `io.github.stream29.kodex.cli.history.createAgentHistoryViewModel`
  [L928–932][history-factory] returns that spec via the existing
  `AgentHistoryViewModelImpl`. Production
  `io.github.stream29.kodex.cli.rpc.RpcAgentViewModel`
  [L108–112][history-caller] passes the captured binding's real storage,
  latest/display-state and **index cache nonce**, under its parented History
  scope. [AgentHistorySource L7–24][history-source] already requires one fixed
  Agent and an authoritative destructive-replacement nonce when supplied.
- **Before conditional B1-H1, now R:** second VM L809–825 handled delayed
  `ExternalWriteFinished(start,end)` as another destructive start≥end
  invalidation. Once synchronization/new Latest had already published nonce 41,
  that completion could again release its child and withdraw its exact effect;
  a captured end could also regress a later same-nonce append.
  [Before consumer][before-history]
- Current producer [L151–170][history] captures `generation` together with end
  **before `commands.send` can suspend**. The same original loop runs
  `synchronizeGeneration()` at L760–765 and L785. At L812–833:
  same-current nonce completion refreshes **actual `latestIndex.value`**;
  obsolete nonce completion is ignored; neither invokes a second destructive
  replacement solely because of captured start/end. No second operation
  authority, VM/controller, retry or journal was added.
- **R no-nonce responsibility retained:** fallback is the previous start/end
  and `lastInvalidation` algorithm, not deletion of completion handling.
  Same-cursor destructive rewrite still invalidates local generation and
  withdraws old children/effect. A preceding cursor-regression Refresh can
  deduplicate that pair. The authoritative spec [L125–159][history-spec]
  matches both modes; `requestOlder`/`requestNewer` KDoc precisely describes
  existing Ready/edge/full-buffer rejection, **not** a new paging implementation.
- **R test gate validity:** four combinations at [L212–347][history-tests]
  use the real VM, InMemory repository/runtime and actual
  `runtime.modify { revert(end); index[end] = replacement }`.
  Test-owned latest/state/nonce flows control delivery; **the VM's true State
  collector constructs completion**. No manually invoked History completion
  callback substitutes for runtime mutation or command production.
- `getExact(7)` blocks actual structural loading; **256 mixed navigation
  requests saturate bounded wakes**. Channel is `Channel.BUFFERED`, not a new
  production fixed-256-capacity queue. Completion's sender is run before the
  duration command's sender; the duration barrier is **after completion**, not
  merely after nonce Invalidate. Append variants advance real cursor after
  completion captures end, keeping nonce 41.
- Unconfined observation captures exact effect/window **at publication**
  [L292–308][history-tests]. Drain must retain those identities and child,
  correct final cursor/payload, no `hasNewer`, Ready/follow intent, rejected
  old-child timestamp access and stale/repeated exact ack semantics. A second
  equivalent-looking window cannot pass the identity gate.
  Positive no-nonce test [L349–425][history-tests] performs actual same-cursor
  rewrite and requires release, generation advance, withdrawn old effect,
  replacement payload and harmless stale ack. Existing tests were not removed
  or relaxed; gates/owners are released in finally.
- **B2-H2/U, bounded evidence:** controlled ports are not an end-to-end cached
  filesystem/RPC delivery experiment. Four new nonce cases use current nonce
  41; stale completion tagged 41 after another destructive nonce 42 is guarded
  statically, not directly exercised by these four tests. A further concrete
  gate would queue that completion, publish/navigation-capture nonce 42, then
  drain past completion and check exact window/child/effect and actual tail.
  Do not mislabel it as the existing VIEW-pressure root investigation.
- **Ablation:** removing captured generation/current-cursor reconciliation
  restores the identified late invalidation/regression. Removing no-nonce
  fallback loses same-cursor rewrite detection. Keep both in the original
  owner; no additional public model or renderer workaround is warranted.

### Curl: primary identity, one cleanup graph and terminal retirement

- Production chain remains
  `io.github.stream29.kodex.utils.ktorclientext.kodexcurl.KodexCurlClientEngine`
  → original `KodexCurlProcessor` → `KodexCurlMultiApiHandler`.
  [Processor L77–91,106–181][processor] creates the actual response Deferred,
  constructs handler **without hooks**, drives it on its original worker and
  closes that API in structured NonCancellable finalization. Private
  `RequestHolder`, actual easy handle/refs, response promise and body remain
  the resources; no fake owner or generic Throwable service was introduced.
- **Before conditional B1-C1, now R:** [second helper L426–435][before-curl]
  attached later C both to first cleanup B and admitted primary A, duplicated
  repeated B, and could attach later exact A beneath B after A already
  suppressed B. That A→B→A source counterexample is genuinely repaired locally.
- [Current L28–64][curl] traverses cause/suppressed by **reference identity**
  with visited nodes, so finite supplied cyclic/overlapping graphs cannot make
  the traversal recurse indefinitely. Already reachable exact cleanup/primary
  is not attached again. An incoming graph overlapping existing nodes gets a
  detached diagnostic IOException **without the unsafe cause link**; the
  supplied original graph is not rewritten.
- The diagnostic wrapper deliberately preserves text rather than unsafe
  Throwable identity. It is not another resource/owner or an alleged original
  native exception. **D:** this policy avoids new attachment cycles/duplicate
  paths for the scoped inputs; it does not normalize arbitrary preexisting
  cycles or duplicated edges wholly inside a disjoint incoming/root graph.
- [L471–500][curl] keeps the admitted primary as root, or first cleanup as root
  after an otherwise successful operation. Independent attempts cover native
  forbid-reuse/remove/easy cleanup, headers, every holder member and body close.
  `RequestHolder.dispose` [L78–94][curl] now sends **each** failure to one
  caller collector, instead of privately nesting a subtree and then attaching
  that subtree again. Actual native operations precede fault hooks.
- **R cross-holder close:** [L142–178][curl] uses one actual `closeCause` for
  pending holders, a shared cleanup root and original→safe-entry records.
  Later holder errors extend the same cleanup subtree; outer API aggregation
  recognizes already reachable identities rather than building A/B cycles.
  Each holder and final real multi cleanup is attempted despite prior failure.
  Multi error remains thrown/observable; close does not silently succeed.
- **R terminal order/guard:** body sees faults known before close, pending
  promise settles, then holder retires at [L490–521][curl]. Returned CurlFail,
  getInfo failure and admitted cancellation keep their root; published headers
  are not retracted. Successful-operation cleanup failure fails pending response
  and is thrown to the loop rather than manufacturing success.
  Completion-handler exceptions now get recorded; if Deferred is already
  settled it is **not retried**, and finally removes disposed refs from the map.
  This structurally closes the previous post-settlement retry/double-release
  exposure; it does not make a successfully closed body retroactively fail.
- **R tests, not full-client runtime certification:** new actual-handler tests
  [L131–241][curl-tests] cover getInfo/admitted CE roots, repeated shared faults,
  later exact primary, cleanup caused by primary, success-first-cleanup identity,
  and two admitted holders plus multi release. They execute real handler/native
  releases before synthetic faults; `publishHeaders=false` is explicitly a test
  distortion. Direct cancellation admission is not actual caller-Job routing.
  Existing post-header test uses returned real `CURLE_PARTIAL_FILE` body cause;
  getInfo hook throws **after real successful getInfo**, not a native getInfo
  error return. Full processor/client fault integration remains U.
- New assertions [L350–381][curl-tests] track the active traversal path to
  reject cycles and count **edges**, not only unique visited nodes, to reject
  duplicate identities. Existing reachability assertions were not weakened.
  `withActualHandler` [L384–429][curl-tests] constructs the real API, drives
  actual `perform`, and tears down its own loop/call/fixture.
- **Ablation:** basic graph-membership checks suffice for repeated safe
  identities. Removing the local `recorded` original→wrapper mapping, however,
  creates a fresh diagnostic for each repeat of an unsafe overlapping cleanup,
  especially across holders/multi release. Retain that once-wrapper duty.
  Recording safe pairs too could be simplified to unsafe pairs only; no
  generic exception framework is necessary or justified by this observation.

#### Remaining Curl gates and limits

- **B2-C2 retained:** inject `bodyClose` hook **after real successful body
  close**, and separately a real channel close/completion callback failure;
  inspect pending promise, already-published headers/body and loop outcome.
  Existing pre-close faults do not certify post-commit faults.
  Cached Ktor `ByteChannel.kt:140–165,214–218` commits close token before
  handlers; `ByteWriteChannelOperations.kt:126–131,239–246` sends successful
  close through a continuation discarding its Result. [Cached Ktor][ktor-cache]
- **B2-C3/U:** new Deferred catch/finally guard has no added throwing
  `invokeOnCompletion` test. Gate that callback on the actual response
  Deferred, require terminal outcome, one retirement and no second native
  release during API finally. Cached Coroutines `JobSupport.kt:1085–1093`
  permits a completion-handler exception to throw. [Cached Coroutines][coroutines-cache]
- **B2-C4/U:** `responseDataRef.get()` is still outside guarded release
  [L490,530][curl]. Repeated failure there can prevent settlement and
  final close still clears attempted holders. No valid normal-owner path to
  such StableRef failure was demonstrated; **do not dispose an invalid native
  pointer to manufacture evidence**. Safe reference-access/loop/getInfo/native
  return/callback matrices remain unverified beyond stated test seams.
- **B2-W1 retained:** original stale-token test simulates address collision;
  it does not force allocator reuse and deliver the old captured callback.
  Required gate remains actual freed-address reuse under a safe test seam,
  followed by that original closure, with no foreign unpause.
  Locked multi wakeup/token mechanisms are unchanged by this delta.
- **D/U:** untouched processor/startup rollback aggregation is not promoted
  to a universal cycle-free guarantee by file-private Curl helpers. All-body,
  native C callback, queued-task, WebSocket, TLS/proxy/reset/chunked/non-2xx and
  multi-failure integration matrices remain bounded by the prior report.
  No new claim of macOS/Windows or all Native platform execution.

### MCP: original suspend construction rollback and real raw-session barrier

- Real public operation remains
  `io.github.stream29.kodex.mcp.stdio.openMcpStdioTransport`;
  [McpServiceImpl L207–215][mcp-caller] calls it directly for stdio.
  It acquires one original `ProcessSession`, adapts raw streams to SDK
  coroutine stdio, and returns the same private `ProcessOwnedTransport`.
  No public signature/dependency/new factory or owner changed.
- **Before conditional B1-P1, now R:** [second catch L50–52][before-mcp]
  used `process.close(); throw failure`; synchronous cleanup throw replaced
  construction primary, and asynchronous Node close did not await raw cleanup.
  Current [L56–71][mcp] awaits **that acquired session** under NonCancellable
  and a finite 10-second cooperative cleanup timeout.
- Raw failure is captured as a value inside the masked boundary, preventing
  an extra exception rethrow/recovery copy at that boundary. Original
  construction failure is rethrown; a distinct directly unsuppressed cleanup
  is attached once, exact-primary cleanup is not self-suppressed.
  Timeout is also secondary. It never closes or joins the borrowed client.
- Actual Node `NodeProcessSession.closeAndJoin`
  [L192–199][node-raw] closes the session, joins its **existing cancellation
  guard**, then awaits saved release outcome within its seven-second wait.
  [L157–167,208–258][node-raw] still owns actual destroy/termination/close-event
  work. This is an exact raw guard barrier, not waiting for numeric exit or an
  active client. These actual files are hash-unchanged, not newly repaired here.
- **D:** finite cleanup is cooperative. Nested raw NonCancellable cleanup and
  synchronous host calls cannot be forcibly interrupted by the outer ten-second
  timeout. Saved raw exceptions can already have library recovery wrappers;
  this catch preserves the Throwable delivered by the raw operation, not a
  universal promise of OS-level reference identity.
- **R test distinction:** [L118–190][mcp-tests] creates explicit
  `caller = Job(coroutineContext[Job])`, then launches from
  `CoroutineScope(coroutineContext + caller)` without deprecated `launch(Job)`.
  This final correction is confined to test scope; the child is explicitly
  parented. `stdout` getter cancels the **real caller Job** before throwing
  primary in cancellation cases; it is not merely an injected CE on an active
  caller. Borrowed client owner is separate and asserted active.
- RecordingRawSession [L193–227][mcp-tests] has separate close/await counters,
  entered/release/finished gates and a suspend cleanup. Old close-only code
  cannot satisfy the entered or delayed-finish assertions. Tests observe primary
  as a value inside the original caller, avoiding Deferred/debug-stack copies
  in identity assertions. They require still-pending construction until release,
  exact primary and IO/CE cleanup identities, one already-suppressed secondary,
  and no exact-primary self edge. Finally releases gates and joins test Jobs.
- **SDK observable check:** read pinned
  `io.modelcontextprotocol.kotlin.sdk.client.StdioClientTransport`
  [L86–136,138–164,271–291][sdk]. Its constructor creates channel/scope;
  actual read/write launches start in `initialize`, not construction.
  New tests reject **Kodex's raw stdout argument getter before SDK constructor
  executes**, not real SDK IO/initialize or Node OS construction failure.
  KDoc's construction-rollback guarantee matches this try/catch boundary;
  acquisition failure remains outside it and owned by ProcessClient startup.
- **D ordinary close unchanged:** `ProcessOwnedTransport.close/onClose`
  [L117–143][mcp] still calls idempotent synchronous `closeProcess` and delegates
  SDK shutdown. This change does not repair all transport callback precedence,
  SDK scope lifecycle, partial SDK construction or post-start failure rollback.
  Existing ordinary framing/close contract tests remain.
- **B2-M1/U, not an invented graph guarantee:** MCP L68 checks self identity
  and direct suppression only. If a cleanup's cause/suppressed graph already
  reaches construction primary, attaching it would make a cycle; nested prior
  suppression can also give duplicate reachability. Actual raw spec
  [L46–57][raw-spec] permits platform cleanup/secondary failures but does not
  promise arbitrary exception-graph normalization. Standard actual stream
  getters and current tests do not demonstrate this shared-primary topology.
  Gate it at this real factory with a contract-valid acquired raw session and
  independently failing rollback, then establish any reachable actual fault
  before calling it a production regression. No broader public cycle-free MCP
  promise or generic Throwable framework is justified here.
- **Ablation:** remove masked exact-session await and old barrier/primary
  defects return. Remove capture-as-value and direct identity/dedup guards and
  tested cleanup identity/self/repeat cases lose their protection. Keep these
  local duties; do not rewrite normal transport close as part of this delta.

### Pipe test correction and final handoff

- [PipeProcessLifetimeTest L219][pipe-test] only changes `runCatching` to
  `runCatching<Unit>` for its recording raw close Result. The release latch,
  stream/exit observation and `Result<Unit>` publication remain identical.
  Production Pipe/Process files have no delta; this is compile-type correction,
  not a resource-lifetime fix or additional platform evidence.
- Scoped closure supported: previous History B2-H1 now has the actual
  completion repair plus attributed old-byte red/final-byte green; previous
  conditional Curl B1-C1/MCP B1-P1 have meaningful source repairs and relevant
  gates. Prior BLOCKED snapshot remains valid historical evidence.
- Main can consume this report asynchronously now. Keep B2/D/U and the separate
  VIEW failure explicit, correlate actual validation with this manifest, and
  obtain a new delta review for later source changes. No full-batch approval,
  commit authorization or build wait is implied.

### URI references

[history]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/cli/history/AgentHistoryViewModel.kt#L151
[history-factory]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/cli/history/AgentHistoryViewModel.kt#L928
[history-source]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/spec/src/commonMain/kotlin/io/github/stream29/kodex/cli/history/AgentHistorySource.kt#L7
[history-spec]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/history/contract/AgentHistoryViewModel.kt#L125
[history-tests]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/impl/viewmodel/src/commonTest/kotlin/io/github/stream29/kodex/cli/history/HistoryViewportAndScrollEffectTest.kt#L212
[history-caller]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/rpc/src/commonMain/kotlin/io/github/stream29/kodex/cli/rpc/RpcAgentViewModel.kt#L108
[before-history]: file:///tmp/kodex-closure-delta-review-20261007/app/component/history/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/cli/history/AgentHistoryViewModel.kt#L809
[curl]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/ktor-client-ext/impl/src/linuxMain/kotlin/io/github/stream29/kodex/utils/ktorclientext/kodexcurl/KodexCurlMultiApiHandler.kt#L28
[curl-tests]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/ktor-client-ext/impl/src/linuxX64Test/kotlin/io/github/stream29/kodex/utils/ktorclientext/kodexcurl/KodexCurlTerminalCleanupTest.kt#L131
[before-curl]: file:///tmp/kodex-closure-delta-review-20261007/utils/ktor-client-ext/impl/src/linuxMain/kotlin/io/github/stream29/kodex/utils/ktorclientext/kodexcurl/KodexCurlMultiApiHandler.kt#L411
[processor]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/ktor-client-ext/impl/src/linuxMain/kotlin/io/github/stream29/kodex/utils/ktorclientext/kodexcurl/KodexCurlProcessor.kt#L77
[mcp]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/spec/stdio/src/commonMain/kotlin/io/github/stream29/kodex/mcp/stdio/McpStdioTransport.kt#L23
[mcp-tests]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/spec/stdio/src/commonTest/kotlin/io/github/stream29/kodex/mcp/stdio/McpStdioContractTest.kt#L85
[mcp-caller]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/impl/composition/src/commonMain/kotlin/io/github/stream29/kodex/mcp/impl/McpServiceImpl.kt#L207
[before-mcp]: file:///tmp/kodex-closure-delta-review-20261007/mcp/spec/stdio/src/commonMain/kotlin/io/github/stream29/kodex/mcp/stdio/McpStdioTransport.kt#L30
[node-raw]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/process-client/impl/src/jsMain/kotlin/io/github/stream29/kodex/utils/processclient/NodeProcessClient.kt#L157
[raw-spec]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/process-client/spec/src/commonMain/kotlin/io/github/stream29/kodex/utils/processclient/ProcessClient.kt#L46
[sdk]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/KotlinMcpSdk/kotlin-sdk-client/src/commonMain/kotlin/io/modelcontextprotocol/kotlin/sdk/client/StdioClientTransport.kt#L86
[pipe-test]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/shell-client/impl/src/jvmTest/kotlin/io/github/stream29/kodex/utils/shellclient/PipeProcessLifetimeTest.kt#L216
[previous-history]: file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-07-recheck-storage-frontend-deltas.md#L1
[previous-native]: file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-07-recheck-native-terminal-deltas.md#L1
[central]: file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-07-plan-spec-impl-closure-and-consistency.md#L192
[ktor-cache]: file:///home/stream/.gradle/caches/modules-2/files-2.1/io.ktor/ktor-io/3.5.0/e8ff5ef5ab2302bde98f6a59d35d78c8c16e1c25/ktor-io-3.5.0-sources.jar
[coroutines-cache]: file:///home/stream/.gradle/caches/modules-2/files-2.1/org.jetbrains.kotlinx/kotlinx-coroutines-core-linuxx64/1.11.0/d538029f20d3dff74cbc012cc5bbeac83613cf7f/kotlinx-coroutines-core-linuxx64-1.11.0-sources.jar

REVIEW READY — SCOPED STATIC FIXES CONFIRMED; NATIVE FAULT-PATH CLOSURE BLOCKED

- 审查脚本已完成并归档；本结论对应第一固定快照，后续 Native 修复不回写
  此历史结论；当前证据见 [主任务](2026-10-07-plan-spec-impl-closure-and-consistency.md)。

# Task Tree

- `Independently review fixed Process, Shell and native Curl ownership`()
- `Trace actual platform startup, cancellation and cleanup()`
- `Report platform-specific failures and unverified guarantees()`

# Details

- 用户授权修复后重新审查；独立只读，不授权代码修复或提交。
- 固定快照与验证证据见
  [修复主任务](2026-10-07-plan-spec-impl-closure-and-consistency.md)。
- 独占审查：Process/Shell acquisition handoff、pipe close/exit、Node 的原 child
  与 raw IO、Native Curl loop/API/dispatcher/active body 和 post-header failure。
- 重点核对 parent join、取消-before-dispatch、未交付资源、cleanup identity、
  普通状态与 IO primary，以及 Native 截断/timeout 不被变成成功 EOF。
- 实际 JVM、Node、Linux x64 与其他未运行 actual 分别记录；真实隔离子进程/
  loopback 门闩不覆盖所有 forkpty、ConPTY、fd/startup/descendant 故障组合。
- 只改本文件报告，不构建、跑进程或改实现/历史报告；列 source/消费者、
  测试有效性、URI/行号及去层实验，不凭 `Job`/结果字段名字认定新架构层。

## Independent report

### Verdict and evidence boundary

- Independently reviewed the fixed snapshot, not the mutable implementation tree.
  Manifest SHA-256:
  `3d661ec7abdf3e33a8de2b103ad7a19b8504cfd2b8a1168f00980fbd7765335e`.
  All **1290** listed file hashes match; no missing or extra project files.
- Baseline comparison used `git show 6b7129fa0e719dde8f88a0034b9d572679984a03:<path>`.
  History identifies `6606efe9` organization, `eff7b10e` real contract cutover,
  and `e89c5094` earlier lifecycle repairs. No worktree source was substituted
  for snapshot content. Original URIs below are navigation links; their line
  numbers refer to the frozen bytes under
  `file:///tmp/kodex-closure-review-6b7129fa-20261007/`.
- **R** means a scoped static repair/contract trace is confirmed, not runtime
  certification. **B1** is a confirmed control-flow defect; its OS fault injection
  remains unrun. **B2** requires controlled reproduction before claiming an actual
  failure. **D** is retained, explicitly delimited behavior/debt. **U** is unverified.
- P1 nonblocking pipe close, P2 direct Node owner cleanup, C1 original Curl
  engine-child cleanup, and post-header transport-failure propagation are real
  repairs. There is **no unconditional platform closure**: B1-C1 remains, with
  B2-C2/B2-N1 and the OS fault matrix below still open.
- This reviewer ran no builds, Gradle, CLI/IDE, subprocess/network fixtures, PID
  controls, or device operations; made no implementation/other-report edits.
  Only this report is written. The shared snapshot is retained for other reviewers.
- User/coordinator evidence says post-header truncation was centrally reproduced
  before repair and Node/Linux core gates passed. This is **reported centralized
  evidence**, not an independently executed test or reviewed test-result archive.
  Exact final JVM/JS/Linux runs belong to the coordinator. The implementation
  handoff's older “C2/B2 unconfirmed; algorithm unchanged” is superseded by the
  frozen diff; it must not be used as the current algorithm description.

### Contract → actual → production trace

| Boundary | Actual ownership and real consumers | Static result |
| --- | --- | --- |
| `io.github.stream29.kodex.utils.processclient.ProcessClient` / `ProcessSession` | [Authoritative spec:19–72][process-spec]; [scope factory:34–35 and acquisition:56–117][process-common]. JVM `PlatformProcessClient.start` [27–40][process-jvm], POSIX [60–64][process-posix], MinGW [76–80][process-windows] use that acquisition function. JS uses the same real factory and `startNodeProcess` [66–137][process-node]. `io.github.stream29.kodex.mcp.impl.McpServiceImpl` injects the spec type [47–48, 214, 314][mcp-service]; `ProcessClient.openMcpStdioTransport` starts the original session [30–53][mcp-stdio] and forwards its original streams. | **R** contract/factory identity. No replacement process client, Lease or manager. |
| `io.github.stream29.kodex.utils.shellclient.ShellClient` / `ProcessSession` | [Spec ownership/start:5–28][shell-spec]; [exit/close:92–114][shell-session]; [factory and acquire:34–85][shell-common]. JVM/POSIX/MinGW actual startup wraps the original resource acquisition; pipe sessions call `ProcessClient.start` then `PipeProcessSession` [28–49][pipe]. JS pipe startup [86–110][shell-node] uses that same pipe path; JS PTY still uses `NodePtyTransport`/`NodeProcessSession`. `io.github.stream29.kodex.tool.unifiedexec.UnifiedExecToolClient` holds the original spec [35–57][unified], constructs it [190–199][unified], observes exit/output [228–233][unified] and delegates session close [218–219][unified]. | **R** original production path and P1 repair; **B2-N1** session-only barrier; PTY limits below. |
| Linux Ktor/SSE | Public extension contract remains `HttpClient.postSseEvents*` [17–80][sse-spec], using Ktor's real SSE Flow/session. `io.github.stream29.kodex.openai.client.OpenAiClient` creates default `HttpClient` [90–94][openai-client], calls the extension [370–387, 518–525][openai-client]. MCP also creates default Ktor [28–38][mcp-http]. Eager Linux registration and `KodexCurl.create` [21–41][curl-factory] instantiate `KodexCurlClientEngine`; its `execute` [29–74][curl-engine] creates the native request and returns the real response/body. | **R** meaningful spec and actual Ktor factory, not an invented Curl spec facade. JVM CIO, Darwin and WinHttp are different engines. |

### Confirmed scoped repairs

- **R-S1 — shared pipe close, JVM/JS/POSIX/MinGW.**
  `PipeProcessSession.close` [113–125][pipe] changed from UNDISTPATCHED launch to
  scheduled `ShellPipeIoDispatcher` launch. Blocking raw JVM termination no longer
  runs inline in the ordinary production caller. The original session guard
  [63–77, 226–255][pipe] owns NonCancellable IO cleanup; its Job remains under
  `sessionJob`, so parent cancellation cannot merely skip that finally.
  The fake blocking gate [42–71][pipe-test] actually invokes close inline and
  requires caller progress before opening the close gate.
- **R-S2 — primary IO and expected termination.**
  Spec now accurately permits an IO/observer failure to win over known numeric
  status [92–106][shell-session]. Input failure publishes the same primary before
  cancellation; guard cleanup attaches its failure rather than replacing it.
  [74–94][pipe-test] retains raw exit 0 while requiring the injected IO primary
  in both send/exit cause chains and the exact cleanup object as suppressed.
  New generic-output catch suppression [221–223][pipe] prevents Node's deliberate
  closed-source `IllegalStateException` after an explicit Shell termination
  request from spuriously replacing numeric termination.
- This does **not** mean arbitrary output failure is always propagated:
  existing IOException suppression [217–220][pipe] still depends on raw exit
  completion, and the new generic suppression covers every Throwable after the
  termination flag, not a typed closed-stream marker. These are **D-S1/B2**
  boundaries: the added test is input-only, not a distinct late-output-fault
  experiment. Do not broaden “can win” into “all output faults always win.”
- **R-P1 — JVM/POSIX/MinGW acquisition/dispatch cancellation.**
  A short acquisition child is registered under the real client before OS
  acquisition [65–98][process-common]; capture occurs inside the dispatcher before
  cancellable return. Failed handoff closes the captured session, waits its exit
  with a six-second bound, joins rollback and retains primary/suppressed errors.
  Session cancellation guard and blocking exit observer use ATOMIC startup
  [43–51][process-common], [63–75][process-jvm], [194–204][process-posix],
  [254–267][process-windows]. Removing the old LAZY startup prevents cancellation
  before dispatch from simply skipping close/reaping. Nullable guard assignment
  accommodates cleanup before the constructor assigns the Job.
- `CompletableDeferred<Result<Unit>>` is an **unparented observation cell**, not
  another resource owner: it launches nothing, owns no process, and only carries
  rollback outcome to the caller. The acquisition child is the owner barrier.
  [32–52, 55–130][acquire-test] gates owner cancellation during acquisition and
  dispatcher-return cancellation. These injected sessions do not prove OS spawn
  failure/descriptor/constructor fault paths.
- **R-P2 — direct Node JS.**
  Parentless `cleanupScope`/`SupervisorJob()` is deleted. Before asynchronous
  spawn delivery, `NodeProcessSession` is created and bound to the original client
  [102–126, 157–168][process-node]. Cancelled delivery uses the continuation
  cleanup hook; failed startup executes `closeAndJoin` in NonCancellable
  [128–136, 192–195][process-node]. Owner-before-spawn-delivery checks are explicit.
- Original Node guard cleanup [204–254][process-node] attempts stdin/stdout/stderr
  destroy independently; terminates only its captured child/group, awaits actual
  close, forces and awaits again on timeout (5s + 1s). Windows taskkill helper
  has its own bounded wait/destroy/close attempts [272–327][process-node].
  Explicit session close invalidates raw streams immediately and cancels only
  that guard; it does not cancel the client/application. Sources reject even
  retained chunks when owner is cancelled [357–398][process-node]. Natural exit
  preserves drain until explicit close; the stream owner intentionally remains.
  No arbitrary PID discovery/deletion or second cleanup service is introduced.
- **R-C1 — Linux Curl original engine owner.**
  `KodexCurlClientEngine` supplies its actual Ktor engine context
  [26, 78–80][curl-engine]. `KodexCurlProcessor.runEventLoop`
  [106–181][curl-processor] is an ATOMIC engine-Job IO child; normal native
  init/perform/callback handling stays on the single Curl dispatcher. Finally
  enters NonCancellable on IO, stops admission, cancels the current dequeued task,
  drains queued tasks, closes native API and then closes dispatcher back on IO.
  Undelivered-task cleanup [31–34, 248–260][curl-processor] also settles outcomes.
- There is no GlobalScope cleanup launch, engine-parent self-join, or Curl-worker
  self-close. Initialization failure joins the actual event-loop rollback
  [57–73][curl-processor]. `cleanupCompleted` is another result observation,
  not a second owner. The original call-context writer in `KodexCurlRaw`
  [111–124][curl-raw] still spells `GlobalScope.writer(coroutineContext)`; its
  supplied operation Job is meaningful. It is not the removed detached cleanup.
- **R-C3 — Linux post-header failure.**
  `collectFailedResponse` now checks `CURLE_OK`, not `httpStatusCode != 0`
  [447–484][curl-multi]. `processCompletedEasyHandle` preserves transfer cause and
  closes body with it [407–443][curl-multi], even when the response promise has
  already published status 200 [192–195, 367–371][curl-multi]. Body closure
  [82–85][curl-body] therefore does not manufacture successful EOF for truncation.
  HTTP 4xx/5xx with `CURLE_OK` still produce their real status/body rather than
  transport failure; no status-only rejection was introduced.
- Connect timeout remains distinct from post-header socket timeout
  [458–466][curl-multi]; ordinary idle timeout has its own SocketTimeoutException
  path [507–520][curl-multi]. Only the latter and truncation have new loopback
  tests. Non-2xx body, actual connect timeout, pre-header idle/connect competition,
  proxy/TLS/reset and chunked EOF matrices remain unverified.

### Remaining blocking defect and reproduction gates

#### B1-C1 — Linux Curl completion/cancellation cleanup can mask primary

- **Confirmed from control flow**, not a claim that libcurl emitted an injected
  error during this review. `processCompletedEasyHandle` saves `terminalCause`
  but its [436–443][curl-multi] nested finally directly calls
  `closeResponse`/`cleanupEasyHandle`. A cleanup `verify()` exception replaces a
  thrown primary (for example failed `getInfo`) or replaces the return carrying
  `KodexCurlFail`. The original transport cause is not attached to that new
  cleanup exception. `RequestHolder.dispose` in [373–375][curl-multi] can replace
  either again; it aggregates only its own cleanup failures.
- Cancellation has the same hole: [380–404][curl-multi] removes the holder,
  calls `processCancelledEasyHandle`, then directly calls holder disposal in
  finally. Closing body, FORBID_REUSE and remove-handle verification do not
  suppress onto the admitted cancellation cause. This is inherited baseline
  debt that remains inside the requested primary-preservation review.
- **Related completion fault:** if an exception leaves `processCompletedEasyHandle`
  before [367–371][curl-multi], its finally removes/disposes the holder without
  settling a still-pending response promise. The processor is in `api.perform`,
  not processing `activeTask`; finally cannot recover that removed holder.
  The awaiting operation has no direct outcome from that promise unless another
  cancellation path ends its call. This is not fixed by the new queue drain.
- Current native startup test [182–225][curl-test] fails before native admission
  and injects `beforeRelease` failure. It covers processor-level suppression,
  **not** a completed/cancelled admitted holder with remove/dispose failure.
- Minimal closure: retain the real holder/primary, independently attempt each
  release, attach cleanup to the primary, and settle its actual promise on every
  terminal path before dropping holder reachability. No ResourceManager/Lease.
  Needed centralized fault cases: getInfo primary + remove failure; returned
  transport failure + remove failure; admitted cancellation + close/option/
  disposal failure; pre-header completion failure must finish its await.

#### B2-C2 — Linux late unpause bypasses multi release lock

- Real callbacks are response backpressure finally [57–69][curl-body] and
  upload-content finally [78–85][curl-callbacks], not an invented external caller.
  Both reach `KodexCurlMultiApiHandler.unpauseEasyHandle` [561–566][curl-multi].
- The new `wakeup()` [297–300][curl-multi] correctly synchronizes native wakeup
  with final multi cleanup [117][curl-multi]. But unpause still invokes
  `curl_multi_wakeup(multiHandle)` **directly, outside that lifecycle lock**.
  Legal interleaving: callback reads `closed == false` at L566; Curl worker
  sets closed and frees multi; callback invokes wakeup on the released pointer.
  The queue lock and repeated atomic checks do not close that check/use window.
- A callback arriving wholly after close returns harmlessly. The open gate is
  the callback **straddling** native release. Actual crash/UAF reproduction is
  **not claimed**. There is also a retained bare-easy-pointer unpause queue:
  membership check [277–282][curl-multi] lacks the identity token used by
  `cancelRequest` [258–262][curl-multi]; late upload wakeup + easy-pointer reuse
  needs its own experiment before asserting it wakes a foreign request.
- Paused-body test [98–130][curl-test] genuinely reaches backpressure but does
  not gate callback between final closed-check and native wakeup or force easy
  reuse. Minimal ablation/fix candidate: route this existing wakeup through the
  existing lock-protected operation; test straddling close and stale callback
  identity without adding another owner or worker.

#### B2-N1 — JS Shell session-only join versus raw Node cleanup

- Raw Node guard belongs to the **ProcessClient** [157–168][process-node].
  Pipe `sessionJob` belongs separately to the Shell parent [48–49][pipe];
  `startPipeProcess` hands the already-client-owned raw session into it
  [28–41][pipe]. Parent Shell/client cancellation and join therefore include both
  paths, but raw guard is not a child of this Pipe `sessionJob`.
- On Pipe session cancellation, its guard calls asynchronous raw `process.close`
  [245–254][pipe], which only invalidates/cancels the Node guard
  [184–195][process-node]. It does **not await** `closeAndJoin`/raw release.
  Pipe readers/writer can cancel and session Job can finish while Node's guard
  is still awaiting the actual child close. That is a structural gap against
  Shell spec's [108–114][shell-session] session-Job cleanup barrier.
- No assertion of an observed orphan. Needed reproduction: real isolated Node
  Shell pipe, cancel only session.scope while keeping Shell/client parent alive,
  hold/delay raw close settlement, require session Job join not to finish before
  that child's close and all three raw sources are destroyed. Also test unclaimed
  Shell startup rollback, which currently waits the same session Job.
- Existing JVM `GatedRawProcess.close` is synchronous [196–225][pipe-test], so its
  session-join result cannot prove the JS asynchronous barrier. Direct Node tests
  [34–74][node-test] correctly join the **raw guard/client parent**, not this Shell
  session-only case. Close/exit observer waits cannot substitute for full release.
- Minimal candidate is an awaitable cleanup outcome on the original raw session,
  used by the original Pipe guard; do not join the whole still-active ProcessClient
  from the Shell child or introduce another cleanup owner.

### Platform/fault matrix and retained boundaries

| Platform/path | Reviewed status | Explicitly unverified or retained |
| --- | --- | --- |
| JVM direct pipes / shared Shell pipes | **R** P1 and acquisition ownership; **D-S1** selective output suppression | Real force fixture source exists; OS-specific descendant survival, blocking stdin/output close failure, constructor partial acquisition and cancellation-before-dispatch with real spawn remain **U**. Guard/observer close uses baseline best-effort runCatching [78–96][process-jvm], not a newly validated all-cleanup-error contract. |
| Node JS direct pipes | **R** P2 under original client | Acquired-return cancellation test [102–130][node-test] checks no guard remains, not that unclaimed child's PID is dead after a cleanup timeout. Owner-cancel-before-spawn-delivery, destroy/terminate/force/close failure, helper faults and descendant survival remain **B2/U**. No exact signal-to-numeric normalization is claimed (`null` close code still maps to 1). |
| Node JS Shell pipe | **R** shared non-inline raw close; **B2-N1** | Expected closed raw source is suppressed after explicit termination; independent simultaneous output-fault preservation and session-only release barrier need gated tests. |
| Linux x64 POSIX direct pipe | **R** original acquire guard + ATOMIC waitpid observer | Nested pipe transfer [74–121, 146–162][process-posix] retains child stdinRead/outputWrite/errorWrite only until transfer returns, then closes parent-unused ends; failed fcntl closes both ends. C fork child dup2/closes extras/exec rejection→127 [225–259][process-spawn]. Actual fd-flag/fork/dup2/close/constructor/kill failure and concurrent reaping remain **U**. `ECHILD→0` [224–235][process-posix] is **D**, not validated actual success status. |
| Linux x64 POSIX PTY | **R** new post-forkpty setup-failure rollback; **D/U** remaining algorithms | [124–144][shell-posix] now closes master, kills only captured child/group and retries EINTR reaping if session setup fails. forkpty [85–107][shell-spawn], descriptor setup/constructor cancellation, failed kill, observer-vs-cleanup reaping, EIO-as-terminal EOF, ECHILD polling and descendant failure are not reproduced. Cleanup still swallows errors [323–346][shell-posix]; reap polling [453–473][shell-posix] is not a bounded all-fault guarantee. |
| JVM pty4j / Node node-pty | **D/U**, not P1 pipe certification | Existing guards/releases remain separate PTY algorithms: JVM [219–241][shell-jvm], JS [182–205][node-pty-session] swallow cleanup failures. JS subscription acquisition [139–163][shell-node] lacks explicit rollback for partial onData/onExit registration and does not await an actual exit event during cancellation cleanup. No PTY fault-injection evidence. |
| MinGW direct pipes / ConPTY | **R** shared acquisition/ATOMIC source adaptation; runtime **U** | Suspended root + Job assignment before ResumeThread [139–189][process-windows]; thread closes in finally, transferred parent pipe handles retained. Job-bind/ResumeThread/TerminateJobObject/Wait/GetExitCode/CloseHandle/ConPTY creation and rollback fault matrix unrun. [270–278][process-windows] ignores termination result. ConPTY handle rollback [125–165][shell-windows] and retained fallback semantics are not runtime-certified. |
| Linux native Curl | **R** C1 and post-header cause; **B1-C1**, **B2-C2** | Multi setopt init rollback [72–85][curl-multi], easy/body init rollback [126–153][curl-multi], StableRef/header partial-transfer cleanup [154–254][curl-multi] are present. slist append failure frees accumulated list [67–84][curl-adapters]. Actual malloc/StableRef/easy/multi-init/options/add/remove/dispose failures, completion faults, late callback reuse and WebSocket matrix remain **U**. Kotlin allocations before the guarded block are not blanket OOM-safe evidence. |
| macOS arm64 / Linux arm64 / Windows hosts | **U runtime** | Reviewed shared/native source is not actual execution. No macOS forkpty/Node/JVM, Windows Node taskkill/ConPTY/JVM or arm64 runtime results were supplied to this reviewer. Compilation on another target does not discharge them. Custom Curl is Linux-only; Darwin/WinHttp results cannot close its gates. |
| Browser/Wasm/iOS | **U / no host actual certification** | Available spec/model declarations do not establish executable local Process/Shell implementations. No inferred support from another platform. |

### Test validity and primary implementation-library checks

- New tests retain real ownership assertions rather than merely checking fields
  named Job/result. Acquisition tests gate actual dispatcher return; Node tests
  spawn the test Node executable and only query PID published by that child.
  [148–175][node-test] makes parent finally await cleanup. No foreign process
  discovery or broad cleanup is present in those tests.
- Native fixture [15–77][loopback-test] owns loopback ephemeral listener/accepted
  peer, uses native IO timeouts and finally release under test IO children.
  It always advertises **200**, so it provides no real non-2xx-body experiment.
  JVM fixture installs its slow hook in the separately launched fixture class,
  not the runner [147–187, 229–237][pipe-test].
- Native pause test sends 1,310,720 bytes, exceeding real Ktor **1 MiB** channel,
  and waits `isPausedForBackpressure` [98–116][curl-test]; this is not a presumed
  4 KiB buffer test. Truncation gate saves one `bodyAsChannel` at
  [29–35][truncation-test], receives headers/partial content before closing peer,
  then requires IOException. It cannot accidentally pass through double receive
  or successful EOF. Idle-timeout test [42–53][truncation-test] requires actual
  SocketTimeoutException in the cause chain, not any timeout.
- Graceful client-close test [134–156][curl-test] closes inside the response call,
  exits that call, **then** joins client/engine and awaits cleanup. It does not
  assert engine join inside its own still-active call.
- JVM Deferred/Job debug recovery can copy exceptions: cause-chain identity plus
  exact suppressed-cleanup identity [82–87, 127–130][pipe-test] is meaningful,
  not weakened error semantics. Native direct failure tests retain `assertSame`
  [53–95, 211–217][curl-test]. Ktor channel close itself wraps/copies causes;
  requiring original cause-chain membership is correct.
- Primary cached sources read directly, without extraction/download/build:
  [Ktor core 3.5.0 sources][ktor-core-cache], entries
  `commonMain/io/ktor/client/HttpClient.kt:1289–1314,1476–1490` and
  `engine/HttpClientEngineBase.kt:48–58`, confirm child call/client ownership and
  graceful `complete()`. `call/HttpClientCall.kt:89–94` confirms DoubleReceive.
  [Ktor IO 3.5.0 sources][ktor-io-cache], `ByteChannel.kt:22,140–165` and
  `CloseToken.kt:17–25`, confirm capacity/close-cause wrapping.
  [Coroutines Linux 1.11.0 sources][coroutines-cache],
  `nativeMain/MultithreadedDispatchers.kt:73–75,154–171`, confirm dispatcher close
  blocks for native worker termination, supporting the IO-coordinator repair.
- Test source validity is **not a runtime pass**. Outstanding suites include
  existing factory/raw bytes/EOF/environment, Shell stdin ack/merged bounded
  buffer/Unicode/PTY and JVM SSE regressions. Final centralized results must
  identify exact target/suite and match the frozen manifest or receive delta review.

### Minimal meaningful ablation

- Remove the acquisition child: owner join can finish during unclaimed spawn;
  lose rollback waiting. Keep it. Remove the ATOMIC guard/observer startup:
  cancelled-before-dispatch can skip finally/reaping. Keep them.
- Remove result observation deferreds: no extra owner disappears; only explicit
  cleanup outcome/identity evidence disappears. Keep where needed; do not add
  owner-shaped wrappers around them.
- Remove the original Node guard: parent loses the actual terminate/force/close
  barrier. Keep guard-owned cleanup; do not restore parentless cleanupScope.
- Remove Curl IO coordinator or move dispatcher close back to Curl worker:
  recreate native self-close risk. Keep one engine-owned loop; do not join engine
  parent from that child.
- Delete HTTP-status shortcut and pass actual transfer cause into existing body
  close: this is the observed minimal post-header repair. Do not replace response
  promise/body or reject all non-2xx statuses.
- B1-C1 needs local primary-preserving terminal release, B2-C2 the existing native
  lock/identity gate, B2-N1 the original raw resource's awaited cleanup boundary.
  None warrants another client, ProcessManager, ResourceManager or Lease.

[process-spec]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/process-client/spec/src/commonMain/kotlin/io/github/stream29/kodex/utils/processclient/ProcessClient.kt#L19
[process-common]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/process-client/impl/src/commonMain/kotlin/io/github/stream29/kodex/utils/processclient/ProcessClient.kt#L34
[process-jvm]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/process-client/impl/src/jvmMain/kotlin/io/github/stream29/kodex/utils/processclient/JvmProcessClient.kt#L27
[process-posix]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/process-client/impl/src/posixMain/kotlin/io/github/stream29/kodex/utils/processclient/PosixProcessClient.kt#L60
[process-windows]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/process-client/impl/src/mingwX64Main/kotlin/io/github/stream29/kodex/utils/processclient/WindowsProcessClient.kt#L76
[process-node]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/process-client/impl/src/jsMain/kotlin/io/github/stream29/kodex/utils/processclient/NodeProcessClient.kt#L79
[process-spawn]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/process-client/impl/src/nativeInterop/cinterop/process_client_spawn.def#L225
[shell-spec]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/shell-client/spec/src/commonMain/kotlin/io/github/stream29/kodex/utils/shellclient/ShellClient.kt#L5
[shell-session]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/shell-client/spec/src/commonMain/kotlin/io/github/stream29/kodex/utils/shellclient/ProcessSession.kt#L92
[shell-common]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/shell-client/impl/src/commonMain/kotlin/io/github/stream29/kodex/utils/shellclient/ShellClient.kt#L34
[pipe]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/shell-client/impl/src/processClientMain/kotlin/io/github/stream29/kodex/utils/shellclient/PipeProcessSession.kt#L113
[shell-jvm]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/shell-client/impl/src/jvmMain/kotlin/io/github/stream29/kodex/utils/shellclient/JvmShellClient.kt#L33
[shell-posix]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/shell-client/impl/src/posixMain/kotlin/io/github/stream29/kodex/utils/shellclient/PosixShellClient.kt#L124
[shell-windows]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/shell-client/impl/src/mingwX64Main/kotlin/io/github/stream29/kodex/utils/shellclient/MingwShellClient.kt#L125
[shell-node]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/shell-client/impl/src/jsMain/kotlin/io/github/stream29/kodex/utils/shellclient/NodeShellClient.kt#L86
[node-pty-session]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/shell-client/impl/src/jsMain/kotlin/io/github/stream29/kodex/utils/shellclient/NodeProcessSession.kt#L182
[shell-spawn]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/shell-client/impl/src/nativeInterop/cinterop/shell_client_spawn.def#L85
[unified]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/unified-exec/src/commonMain/kotlin/io/github/stream29/kodex/tool/unifiedexec/UnifiedExecToolClient.kt#L35
[mcp-service]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/impl/composition/src/commonMain/kotlin/io/github/stream29/kodex/mcp/impl/McpServiceImpl.kt#L47
[mcp-stdio]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/spec/stdio/src/commonMain/kotlin/io/github/stream29/kodex/mcp/stdio/McpStdioTransport.kt#L30
[mcp-http]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/spec/streamable-http/src/commonMain/kotlin/io/github/stream29/kodex/mcp/streamablehttp/McpStreamableHttp.kt#L28
[openai-client]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/impl/client/src/commonMain/kotlin/io/github/stream29/kodex/openai/client/OpenAiClient.kt#L90
[sse-spec]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/ktor-client-ext/spec/src/commonMain/kotlin/io/github/stream29/kodex/utils/ktorclientext/SseRequests.kt#L17
[curl-factory]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/ktor-client-ext/impl/src/linuxMain/kotlin/io/github/stream29/kodex/utils/ktorclientext/kodexcurl/KodexCurl.kt#L21
[curl-engine]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/ktor-client-ext/impl/src/linuxMain/kotlin/io/github/stream29/kodex/utils/ktorclientext/kodexcurl/KodexCurlClientEngine.kt#L26
[curl-processor]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/ktor-client-ext/impl/src/linuxMain/kotlin/io/github/stream29/kodex/utils/ktorclientext/kodexcurl/KodexCurlProcessor.kt#L106
[curl-multi]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/ktor-client-ext/impl/src/linuxMain/kotlin/io/github/stream29/kodex/utils/ktorclientext/kodexcurl/KodexCurlMultiApiHandler.kt#L407
[curl-body]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/ktor-client-ext/impl/src/linuxMain/kotlin/io/github/stream29/kodex/utils/ktorclientext/kodexcurl/KodexCurlHttpResponseBody.kt#L57
[curl-callbacks]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/ktor-client-ext/impl/src/linuxMain/kotlin/io/github/stream29/kodex/utils/ktorclientext/kodexcurl/KodexCurlCallbacks.kt#L78
[curl-raw]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/ktor-client-ext/impl/src/linuxMain/kotlin/io/github/stream29/kodex/utils/ktorclientext/kodexcurl/KodexCurlRaw.kt#L111
[curl-adapters]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/ktor-client-ext/impl/src/linuxMain/kotlin/io/github/stream29/kodex/utils/ktorclientext/kodexcurl/KodexCurlAdapters.kt#L67
[pipe-test]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/shell-client/impl/src/jvmTest/kotlin/io/github/stream29/kodex/utils/shellclient/PipeProcessLifetimeTest.kt#L41
[acquire-test]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/process-client/impl/src/commonTest/kotlin/io/github/stream29/kodex/utils/processclient/ProcessAcquisitionLifetimeTest.kt#L31
[node-test]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/process-client/impl/src/jsTest/kotlin/io/github/stream29/kodex/utils/processclient/NodeProcessLifetimeTest.kt#L33
[curl-test]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/ktor-client-ext/impl/src/linuxX64Test/kotlin/io/github/stream29/kodex/utils/ktorclientext/kodexcurl/KodexCurlLifetimeTest.kt#L21
[truncation-test]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/ktor-client-ext/impl/src/linuxX64Test/kotlin/io/github/stream29/kodex/utils/ktorclientext/kodexcurl/KodexCurlTruncatedBodyTest.kt#L24
[loopback-test]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/ktor-client-ext/impl/src/linuxX64Test/kotlin/io/github/stream29/kodex/utils/ktorclientext/kodexcurl/NativeLoopbackHttpFixture.kt#L15
[ktor-core-cache]: file:///home/stream/.gradle/caches/modules-2/files-2.1/io.ktor/ktor-client-core/3.5.0/2f1796882e83530e2eb4b551d9bb600fc1d82e0e/ktor-client-core-3.5.0-sources.jar
[ktor-io-cache]: file:///home/stream/.gradle/caches/modules-2/files-2.1/io.ktor/ktor-io/3.5.0/e8ff5ef5ab2302bde98f6a59d35d78c8c16e1c25/ktor-io-3.5.0-sources.jar
[coroutines-cache]: file:///home/stream/.gradle/caches/modules-2/files-2.1/org.jetbrains.kotlinx/kotlinx-coroutines-core-linuxx64/1.11.0/d538029f20d3dff74cbc012cc5bbeac83613cf7f/kotlinx-coroutines-core-linuxx64-1.11.0-sources.jar

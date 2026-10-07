REVIEW READY — SCOPED REPAIRS CONFIRMED; ERROR/CONSUMER CLOSURE BLOCKED

- 本轮审查已完成，结论对应第二固定快照；Curl/MCP 末项修复另审，不回写
  本历史结论；当前证据见 [主任务](2026-10-07-plan-spec-impl-closure-and-consistency.md)。

# Task Tree

- `Independently review native terminal cleanup and exact raw-session barriers`()
- `Verify release, promise completion and late callback identity`()
- `Report scoped closure and unverified platform fault paths`()

# Details

- 用户授权修复后重新审查；独立只读，不能改实现或沿用作者 READY 为通过。
- 第二固定快照：`file:///tmp/kodex-closure-delta-review-20261007/`。
  1292 文件；manifest SHA-256
  `13a0843b6d57557bf1c98ad2ad940b77015a91c5469b8a130a38324ca513894f`。
  第一固定快照与 `DELTA_FROM_FIRST_REVIEW.json` 提供前后对照。
- 独占：ProcessSession 唯一 closeAndJoin 契约与所有 actual、Pipe exact
  raw session join、Curl holder terminal release、promise/body 错误、unpause
  生命周期锁与 captured token；核对真实 Node/loopback fault tests。
- 不构建、改源码、跑进程/网络、操作 IDE/设备或提交；只写本报告。
  不把 Linux/Node 静态支持扩展为 Windows/macOS 实测通过。

## Independent report

### Evidence boundary and verdict

- Reviewed **only** current fixed source at
  `file:///tmp/kodex-closure-delta-review-20261007/`, against the first fixed
  snapshot at `file:///tmp/kodex-closure-review-6b7129fa-20261007/`.
  Original workspace source links below are navigation links: **all their line
  numbers map to the second snapshot**, not the dirty worktree.
- Independently checked both manifest SHA-256 values:
  - Current: `13a0843b6d57557bf1c98ad2ad940b77015a91c5469b8a130a38324ca513894f`;
    all **1292** listed files match.
  - Before: `3d661ec7abdf3e33a8de2b103ad7a19b8504cfd2b8a1168f00980fbd7765335e`;
    all **1290** listed files match.
  - No missing/mismatched files. Current extras are exactly
    `REVIEW_SOURCE_MANIFEST.json` and `DELTA_FROM_FIRST_REVIEW.json`;
    before has only its manifest artifact.
  - Recomputed the union-of-paths delta: **37** entries, including additions
    and deletions, exactly equal to every `previous`/`current` hash in the delta
    artifact. Other lanes' paths were hash-verified, not substantively accepted.
- Loaded project guidance, associated workflow skills, lifecycle/native-IO,
  spec/impl, Shell and Curl checklists, the executable main plan, this task,
  and the first platform report. Read authoritative Process/Shell code specs.
  Read only fixed Git history where used (`6b7129fa0e719dde8f88a0034b9d572679984a03`)
  and cached primary dependency sources; no worktree implementation substituted.
- **R** = confirmed scoped source repair; **B1** = confirmed conditional
  control-flow defect, not an independently observed OS failure; **B2** = specific
  reproduction/evidence gate; **D** = retained behavior or delimited debt;
  **U** = unverified. Test-source validity is not runtime certification.
- **R:** the first review's asynchronous Pipe/raw barrier gap and unchecked
  unpause check/use window have meaningful local repairs. Curl's known
  transfer/getInfo/cancellation paths now preserve primary and settle the
  original promise. **Not unconditional closure:** suppression aggregation still
  permits duplicate/cyclic exception graphs (B1-C1); an existing MCP rollback
  consumer can mask its primary when the newly throwing synchronous close fails
  (B1-P1). The gates below remain separate from these confirmed conditional defects.
- Coordinator-reported **17 Linux Curl, 37 Node Shell and 14 raw Node** results
  are reported centralized evidence only. No independent runtime or result-archive
  verification here; neither these counts nor “203 green” certify every actual,
  descendants, PTY, callback failure or the new gates.
- Per authorization: no implementation edits, builds/Gradle, subprocess or
  network fixtures, IDE/device/resource control, Git commit/push/switch, or
  other document writes. Only this report was edited with `apply_patch`.
  No fixture, persistent process or temporary review file was created; shared
  snapshots/cache remain available to the coordinator. Actual source after
  author handoff must be compared by the main reviewer to **this manifest**;
  later changes are outside this acceptance bound.

### Original raw session: contract, actuals and consumers

| Original responsibility | Before → current fixed evidence | Classification |
| --- | --- | --- |
| `io.github.stream29.kodex.utils.processclient.ProcessSession` | First spec L28–45 had only `close`; current [L28–57][raw-spec] declares mandatory `suspend closeAndJoin()` with **no default body**. It names exact-session cleanup, active-client independence, saved outcomes and synchronous-host limitations. Factory [L34–35][raw-common] still returns the original `ProcessClient`; no second raw interface/facade/model. | **R** meaningful spec/API addition on the same FQCN. |
| `io.github.stream29.kodex.utils.processclient.JvmProcessSession` | Current [L64–130][raw-jvm] retains original guard/exit observer as client children, saves independently attempted synchronous cleanup in `releaseResult`, then NonCancellable IO awaits that cell and joins those two Jobs inside seven seconds. Before L78–96 silently discarded most close failures and had no public exact-session await. | **R** saved release and exact owner observation; **D/U** host waits/errors below. |
| `io.github.stream29.kodex.utils.processclient.PosixProcessSession` | Current [L193–253][raw-posix] saves kill/stream cleanup errors, then joins the original guard and waitpid observer. Before L207–223 had no await and sequential stream closes. No client-parent join, extra reaper or PID registry. | **R** original-resource barrier; POSIX return-code/error matrix **D/U**. |
| `io.github.stream29.kodex.utils.processclient.WindowsProcessSession` | Current [L255–324][raw-windows] also retains `observerReleaseResult`: observer finally independently attempts captured process/Job-handle closes, and exact-session await joins it before checking that result. Before L260–263 sequential closes could skip the second. | **R** source adaptation; actual MinGW execution **U**, unchecked native returns retained. |
| `io.github.stream29.kodex.utils.processclient.NodeProcessSession` | Current [L153–199,208–258][raw-node] uses the **existing** client-child guard, `processClosed`, and release deferred. Public override adds NonCancellable seven-second wait; before L192–196 was a private implementation-only await. Real destroy/terminate/close wait still happens in that guard, not in a new cleanup scope. | **R** exact raw guard barrier. Deferred is an outcome observation, not a second owner. |
| Real acquisition rollback | Current [L65–115][raw-common] changes captured resource release from `close()` to `closeAndJoin()` at L78, then retains six-second exit observation, guard join and primary suppression. Node startup [L102–136][raw-node] already uses exact captured raw await in failed/cancelled handoff. | **R** original acquisition child still owns undelivered rollback. |
| `io.github.stream29.kodex.utils.shellclient.PipeProcessSession` | First L119/L246 used asynchronous raw `close`. Current [L113–125,226–254][pipe] awaits **that same `process`** in both scheduled termination and NonCancellable session-guard finally. Session Job cannot normally finish while the awaited raw guard is still releasing. It never joins the active `ProcessClient`. | **R** closes the first review's B2-N1 structural hole for ordinary pipes, not PTY. |

- Production chain remains real: Node `PlatformShellClient` owns its original
  raw client and routes `tty=false` through [L84–105][shell-node];
  shared [L28–49][pipe] passes the acquired raw session into the existing Pipe.
  `UnifiedExecToolClient` creates the dedicated Shell client, retains that
  session, delegates close and reads its exit/output
  [L190–219,228–233][unified]. MCP uses the same raw spec session/streams
  [L30–53,84–103][mcp-stdio], not a fake production raw owner.
- Shell acquisition [L52–83][shell-common] still cancels/joins the actual
  undelivered Shell session Job. Its Pipe guard now reaches the exact raw await;
  there is no whole-client self-join hidden in that rollback.
- Known IO, cancellation and status boundaries remain: input failure publishes
  primary before session cancellation; guard attaches cleanup to that primary
  [L164–195,245–253][pipe]. An already-known numeric exit is not a release
  barrier. Output IOException is still suppressed after requested termination
  **or completed raw exit**, and generic output failure after requested
  termination [L215–223][pipe] is still ignored (**D**, not “all IO wins”).
  Borrowed client/parent remains alive when only a session is closed.
- JVM/POSIX/MinGW save caught synchronous-close failures before throwing.
  Subsequent `runCatching { close() }` does not discard those saved errors.
  The guard and observer are actual structured children; result latches only
  observe their work. No `fakeRawProcessSession` production layer, forwarding
  facade, mutable public model or independent cleanup owner was introduced.
- **D — finite means cooperative waits, not forced OS interruption.**
  JVM `process.waitFor()` inside synchronous close [L108][raw-jvm],
  POSIX blocking waitpid [L256–264][raw-posix], and Windows INFINITE wait
  [L327–335][raw-windows] remain host calls. Timeout around join cannot interrupt
  those calls or guarantee all parent children finish in seven wall-clock
  seconds. A timed-out exact wait can fail while the real observer/guard remains
  owned; it is not a successful release. Waiting timeout outcomes themselves
  are not latched as a single immutable whole-barrier result.
- **D — the broad `@throws` is not proof of every native failure being detected.**
  POSIX descriptor close [L279–282][raw-posix] ignores `close`'s result;
  Windows termination [L291–293][raw-windows] and `CloseHandle`
  [L528–530][raw-windows] still ignore native failure returns. Observer
  observation errors remain on `exitCode`, distinct from release-result cells.
  Native false-return/errno and repeated wait-timeout experiments are **U**;
  this review does not certify the strongest all-error reading of the new KDoc.
- **D/U — PTY outside the raw barrier repair.** JVM pty4j, Node node-pty,
  POSIX forkpty/reaping and MinGW ConPTY use separate Shell actual algorithms.
  No PTY file changed in this delta; the first review's PTY limitations remain.

#### Actual Node tests, not substitute owners

- [NodePipeCleanupBarrierTest L22–115][node-barrier-test] has three real
  Node cases: session-only cancellation, ordinary close, and unclaimed Shell
  handoff. First two use actual `ShellClient.start(tty=false)`; the third drives
  the original acquisition helper/`PipeProcessSession` with a real raw child.
  Its return-dispatch gate is not the entire platform Shell start method.
- Test-only prototype shim [L181–219][node-barrier-test] identifies a unique
  test-owned spawn argument and delays **real ChildProcess `close` event delivery**.
  It does not replace spawn, stdio, raw session or raw guard. Tests require all
  three real streams destroyed, join/caller still pending while delivery is
  gated, and original owner/client still active; finally delivers the held
  event and restores the shim before owner teardown.
- The fourth case [L117–141,229–263][node-barrier-test] is a gated fake for
  secondary-error identity only; it is not real-Node OS-failure evidence.
  Common real-child/sibling test [L30–53][raw-io-test] exercises repeated exact
  close without killing the active sibling. Common acquisition and JVM Pipe
  fakes retain result observation, not another production resource owner.
- Node release errors are on the original deferred; platform/coroutine debug
  recovery and Ktor channel close may wrap/copy exceptions. Original cause-chain
  membership plus exact suppressed cleanup identity is meaningful; do not
  demand reference identity of every library-recovered outer exception.
  Actual destroy/kill/helper/listener faults and descendant death remain **U**.

### Curl: original holder and terminal release

- Same `io.github.stream29.kodex.utils.ktorclientext.kodexcurl.KodexCurlMultiApiHandler`,
  actual `RequestHolder`, response promise and body. Before
  [L359–443 in the first snapshot][before-curl], nested finally could replace
  transport/getInfo primary and remove/dispose before a pending promise settled.
- **R-C1 — admitted terminal paths repaired.** Current
  [L383–485][curl] obtains the real holder without dropping map reachability;
  thrown getInfo is converted to a `KodexCurlFail`, then `finishRequest` receives
  the original cause. Cancellation/timeout uses the same helper with its admitted
  cause. Independent attempts cover forbid-reuse, remove, easy cleanup, header
  buffer, holder disposal and body close. Holder disposal itself independently
  attempts cancellation subscription, real slist and each StableRef [L39–55][curl].
- **R-C2 — known cleanup faults no longer manufacture body success.**
  The real builder is captured before disposal; its Kotlin body remains usable
  after StableRef disposal. Native easy cleanup precedes freeing callback refs.
  Body close occurs after native/header/holder failures are known
  [L437–451][curl]. A post-header 200 response stays published, but body gets
  the actual transfer primary, with cleanup attached; the failed second
  `completeExceptionally` does not rewrite already-delivered headers.
- Pending pre-header promises now fail even for returned transport failure or
  caught getInfo failure. Promise completion precedes map removal [L448–451][curl].
  On an otherwise-successful transfer, cleanup failure is both published and
  thrown [L454][curl]; it can fail the existing engine loop rather than silently
  report success. It is still that loop's responsibility, not a new fault owner.
- **R-C3 — engine finalization remains structured.** Unchanged processor
  [L106–181][processor] is an ATOMIC engine child; NonCancellable IO coordinator
  stops admission, cancels active dequeued task, drains queued tasks, closes
  original API and closes the native dispatcher off its worker. API
  [L104–135][curl] independently visits remaining holders and attempts multi
  cleanup even after another release fails. Engine context
  [L26,78–80][engine] and all existing request/call metadata ownership remain
  unchanged. Parent/client join is not attempted from this child.

#### B1-C1 — `firstCleanup` creates duplicate and potentially cyclic error graphs

- **Confirmed conditional source defect in new code**, [L426–435][curl],
  irrespective of whether libcurl happened to emit these faults centrally.
  With primary **A**, first cleanup **B**, later cleanup **C**:
  the code adds C to B **and** A, while A already suppresses B. Thus the same
  secondary has two traversal paths. Reusing exact B for two release faults
  adds B twice directly to A: there is no identity-membership check at L434.
- More seriously, after first cleanup B, a later cleanup throws exact primary A.
  L429 checks `previousCleanup !== primary`, not `cleanup !== primary`:
  it adds A under B. A already suppresses B, producing **A → B → A**.
  A cleanup whose cause chain already contains A is also not excluded.
  This is a graph problem, not loss of the root reference or proof of a native leak.
- Existing fault tests use distinct new IOException objects and only test
  reachability [L225–239][curl-terminal-test]. Their recursive helper has no
  visited-identity set; it neither detects duplicates nor safely handles cycles.
  They cannot close the user's explicit no-cycle/no-duplicate gate.
- Smallest responsibility/ablation: keep original helper/holder/promise; remove
  redundant dual attachment and use one identity-safe primary/cleanup aggregation
  policy, including the API-close observable cleanup outcome. No generic manager.
  Coordinator can exercise the **existing actual handler** after real remove/
  disposal with shared cleanup identity and a later throw of the admitted
  primary; require exact root, each secondary once and acyclic cause/suppression
  traversal. No deliberate native leak or invalid pointer is needed.

#### B1-P1 — real MCP constructor rollback can still mask startup primary

- Actual consumer [McpStdioTransport L50–52][mcp-stdio] is unchanged:
  `catch (failure) { process.close(); throw failure }`.
  The delta now permits synchronous raw close to throw its saved cleanup error,
  e.g. JVM [L111–113][raw-jvm]. If both fail, cleanup replaces the constructor
  primary; Node's asynchronous `close()` also does not await exact raw release.
- This is a **conditional control-flow defect**, not a claim this review
  reproduced a real SDK-constructor or OS-close fault. The new raw method cannot
  retroactively give this consumer an awaited rollback. Existing
  `McpStdioContractTest` exercises rejected construction but its recording close
  succeeds; it does not discharge the dual-fault case.
- Minimal responsibility is the same acquired session in this suspend catch:
  exact await with original-primary suppression. Do not add another transport
  owner or claim all MCP transport/callback shutdown was repaired by this delta.
  The ordinary transport close/callback path still calls its original
  non-suspending `closeProcess`; broader MCP lifecycle acceptance remains
  separately scoped.

### Unpause lifecycle, captured identity and test limits

- **R-W1 — first B2-C2 check/use window structurally closed.** Before
  Curl L563–566 queued a bare easy pointer then directly woke multi outside
  its lifecycle lock. Current [L602–607][curl] enqueues a
  `KodexCurlRequestHandle`, releases queue lock, and calls [L322–335][curl].
  Closing check and native wakeup are inside the **same nativeLifecycleLock**
  used by real multi cleanup [L129–133][curl]. Close can set `closed` while a
  wake is gated, but cannot free multi until that entered wake returns.
- **R-W2 — original callback identity captured.** Handle/token is created at
  [L149][curl], before response/upload closures [L159–160,185–189][curl].
  It is not reconstructed by looking up whatever request currently occupies
  the pointer. Worker [L296–307][curl] checks original holder token before
  `curl_easy_pause`; stale callback cannot resume an unrelated same-address
  request under that comparison. No new routing controller.
- **R/D — lock-order review.** Producer does queue-lock work, unlocks, then
  native-lifecycle work. Worker holds queue lock across native easy-unpause,
  which can reenter callbacks; the cached Ktor POSIX `SynchronizedObject` is
  reentrant on the same thread. Callback's queue→native nesting has no inverse
  native→queue path in ordinary wakeup/multi-finalization code. HTTP body
  close/cancellation is outside the native-lifecycle lock. No source-supported
  new two-lock inversion found; this is not a blanket native callback-fault proof.
- **R test design, limited execution claim:** wake straddle
  [L131–176][curl-terminal-test] gates the exact locked check/use point, requires
  actual close to reach native release but not pass it, then releases the wake.
  Real multi cleanup still executes. Last `curl_multi_wakeup` itself is
  intercepted to detect a forbidden post-release call without intentionally
  causing UAF. It uses captured handler unpause manually, not full real
  backpressure callback delivery. This validates lock sequencing, not every
  libcurl/native callback interleaving.
- **B2-W1 — “forced reused easy address” is token-collision simulation, not
  forced allocator reuse.** [L194–215][curl-terminal-test] genuinely retires an
  old request and admits a new one, but manually constructs
  `KodexCurlRequestHandle(actual.easyHandle, old.token)`. It does not make the
  old closure's captured easy pointer equal the newly allocated pointer.
  Good stale-token predicate test; insufficient evidence for actual reuse plus
  delayed original callback. Gate: retain the old original closure/handle,
  deterministically reuse that freed native address for the next real request
  using a test-only allocation seam, then deliver that old closure and require
  no foreign unpause. Do not infer malloc reuse from manual field substitution.

### Specific remaining gates and retained regression boundaries

- **B2-C2 — body-close failure after success commit.** Existing faults use
  `beforeBodyClose`, so primary is known before actual close. The same seam also
  supports `bodyClose`, invoked **after** real `responseBody.close(primary)`
  [L422–447][curl], but no test injects there. With otherwise successful transfer,
  that hook can throw after successful channel close and fail promise/loop while
  the already-closed body retains success. Actual Ktor cancel/Job completion
  callbacks can throw after close state commits; success channel close also
  uses `fireAndForget`, whose terminal continuation discards its failure.
  An ordinary production occurrence is **U**, not claimed reproduced. Gate
  real existing body's close/cancel/Job failure and inspect pending and
  published-header outcomes, including whether it reaches the helper at all.
  Do not call the pre-close synthetic test an all-body-close-fault guarantee.
- **B2/U — reference access and callback faults are not blanket protected.**
  `responseDataRef.get()` is outside guarded release at [L437][curl], and
  outside completion try at [L464][curl]. A fault there leaves the holder mapped;
  `close()` retries it, but ultimately clears the map after attempted releases.
  That combination could drop an unresolved promise if access repeatedly fails.
  No valid normal-owner path to an invalid StableRef was demonstrated here;
  **do not manufacture/dispose an invalid pointer to “prove” a leak**.
  Holder remains safely locally reachable after ordinary disposal, so a
  post-disposal injected error is not itself this failure.
- Promise completion itself is outside `attempt` [L448–451][curl].
  A throwing completion handler can leave a settled holder mapped after refs
  were disposed, exposing retry/double-release risk during API finally.
  Coroutines permits completion-handler exceptions; actual private production
  response-deferred callback fault occurrence and safe handling remain **U**.
  No claim that known getInfo/remove faults still leave the promise pending.
- **R test scope:** new [L16–129][curl-terminal-test] drives actual handler,
  actual loopback transfers and real native operations. `completedGetInfo`
  injection is **after successful real getInfo**, not libcurl returning failed
  getInfo. remove, easy/slists/StableRef hooks execute after actual release, so
  test faults do not intentionally leak before injection. Tests cover pending
  getInfo failure, header-200/truncation plus cleanup, success cleanup failure
  before/after headers, admitted cancellation, and API close.
- Those tests construct/drain handler directly [L242–285][curl-terminal-test],
  not full `HttpClient`/processor task routing. Fixture [L15–77][loopback-test]
  owns listener/peer with native IO timeouts and always sends 200. Emergency
  teardown disables fault hooks and attempts actual API/fixture/call cleanup.
  Full-client fault integration, multiple concurrent failing holders and queued
  tasks plus API-finalizer failure remain **U**, notwithstanding source attempts.
- Hook class [L65–77][curl] is **internal**, per-handler, nullable function fields
  with default-null production construction [L113][processor]. It launches/
  owns nothing and is not a production framework or public routing API.
  `publishHeaders=false`, manual callbacks and substituted native wakeup are
  test distortions to state explicitly, not behavior of ordinary requests.
- **Retained:** real Ktor channel capacity is **1 MiB**; existing paused-body
  fixture exceeds it. Packet-idle socket-timeout algorithm and typed timeout
  remain; `CURLE_OK`, not nonzero HTTP status, still decides transport success
  [L495–525][curl]. Non-2xx with successful transport is not newly rejected.
  Existing truncation test receives body once via one `bodyAsChannel`, so its
  IOException is not DoubleReceive. These files are hash-unchanged in the delta.
  Real non-2xx, TLS/proxy/reset/chunked and connect/idle competition remain **U**.
- **U explicitly retained:** macOS/arm64, Windows taskkill/MinGW/ConPTY,
  JVM/native descendant survival, fork/forkpty/dup2/fd/constructor rollback,
  descriptor/handle-close and termination failures, simultaneous IO/exit/cancel,
  node-pty subscriptions, C callback error matrices and WebSockets.
  No API/wire/source-generation acceptance outside the approved mandatory raw
  operation; no platform runtime inferred from another host's green suite.

### Primary dependency checks and smallest ablation

- Read [cached Ktor IO 3.5.0 sources][ktor-io-cache] without extraction/download:
  `posixMain/io/ktor/utils/io/locks/Synchronized.kt:39–92,304–310` confirms
  same-thread reentrancy and finally unlock; `ByteChannel.kt:22,140–165,214–218`
  confirms capacity and close-token publication before callbacks;
  `ByteWriteChannelOperations.kt:126–131,239–246` distinguishes success close
  from error cancel and shows success `fireAndForget` discarding its outcome;
  `CloseToken.kt:17–25` explains cause wrapping/copying.
- Read [cached Coroutines 1.11.0 Linux sources][coroutines-cache],
  `commonMain/JobSupport.kt:1085–1093`, confirming completion-handler error
  handling can throw. This supports the specific callback gate, not an
  allegation every Job completion throws.
- Remove exact raw await from Pipe guard: real Node session-only join can again
  precede raw callback/release. Keep it. Remove actual raw guard/observer jobs:
  lose termination and owner barrier. Keep them.
- Remove result latches: no extra process owner disappears, only saved outcome
  observation/identity. Do not replace them with another client or manager.
- Remove locked wakeup or token: reopen freed-multi check/use or foreign-address
  callback routing. Keep those local responsibilities. Tighten the reuse test
  rather than adding a production routing layer.
- Keep local `finishRequest` and actual holder until promise settles. Its
  primary/suppression aggregation needs the smallest identity-safe correction,
  not another terminal controller. Preserve delayed body success publication,
  HTTP status/timeout behavior and original request/call ownership.
- **Handoff:** main reviewer can consume this report asynchronously now.
  Scoped R entries supersede corresponding first-snapshot structural gates only;
  B1/B2/D/U above prevent an unqualified Native/platform closure claim.

[raw-spec]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/process-client/spec/src/commonMain/kotlin/io/github/stream29/kodex/utils/processclient/ProcessClient.kt#L28
[raw-common]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/process-client/impl/src/commonMain/kotlin/io/github/stream29/kodex/utils/processclient/ProcessClient.kt#L56
[raw-jvm]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/process-client/impl/src/jvmMain/kotlin/io/github/stream29/kodex/utils/processclient/JvmProcessClient.kt#L64
[raw-posix]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/process-client/impl/src/posixMain/kotlin/io/github/stream29/kodex/utils/processclient/PosixProcessClient.kt#L193
[raw-windows]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/process-client/impl/src/mingwX64Main/kotlin/io/github/stream29/kodex/utils/processclient/WindowsProcessClient.kt#L255
[raw-node]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/process-client/impl/src/jsMain/kotlin/io/github/stream29/kodex/utils/processclient/NodeProcessClient.kt#L153
[pipe]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/shell-client/impl/src/processClientMain/kotlin/io/github/stream29/kodex/utils/shellclient/PipeProcessSession.kt#L113
[shell-node]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/shell-client/impl/src/jsMain/kotlin/io/github/stream29/kodex/utils/shellclient/NodeShellClient.kt#L84
[shell-common]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/shell-client/impl/src/commonMain/kotlin/io/github/stream29/kodex/utils/shellclient/ShellClient.kt#L52
[unified]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/unified-exec/src/commonMain/kotlin/io/github/stream29/kodex/tool/unifiedexec/UnifiedExecToolClient.kt#L190
[mcp-stdio]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/spec/stdio/src/commonMain/kotlin/io/github/stream29/kodex/mcp/stdio/McpStdioTransport.kt#L30
[node-barrier-test]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/shell-client/impl/src/jsTest/kotlin/io/github/stream29/kodex/utils/shellclient/NodePipeCleanupBarrierTest.kt#L22
[raw-io-test]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/process-client/impl/src/commonTest/kotlin/io/github/stream29/kodex/utils/processclient/ProcessClientIoTest.kt#L30
[curl]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/ktor-client-ext/impl/src/linuxMain/kotlin/io/github/stream29/kodex/utils/ktorclientext/kodexcurl/KodexCurlMultiApiHandler.kt#L411
[before-curl]: file:///tmp/kodex-closure-review-6b7129fa-20261007/utils/ktor-client-ext/impl/src/linuxMain/kotlin/io/github/stream29/kodex/utils/ktorclientext/kodexcurl/KodexCurlMultiApiHandler.kt#L359
[processor]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/ktor-client-ext/impl/src/linuxMain/kotlin/io/github/stream29/kodex/utils/ktorclientext/kodexcurl/KodexCurlProcessor.kt#L106
[engine]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/ktor-client-ext/impl/src/linuxMain/kotlin/io/github/stream29/kodex/utils/ktorclientext/kodexcurl/KodexCurlClientEngine.kt#L26
[curl-terminal-test]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/ktor-client-ext/impl/src/linuxX64Test/kotlin/io/github/stream29/kodex/utils/ktorclientext/kodexcurl/KodexCurlTerminalCleanupTest.kt#L16
[loopback-test]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/ktor-client-ext/impl/src/linuxX64Test/kotlin/io/github/stream29/kodex/utils/ktorclientext/kodexcurl/NativeLoopbackHttpFixture.kt#L15
[ktor-io-cache]: file:///home/stream/.gradle/caches/modules-2/files-2.1/io.ktor/ktor-io/3.5.0/e8ff5ef5ab2302bde98f6a59d35d78c8c16e1c25/ktor-io-3.5.0-sources.jar
[coroutines-cache]: file:///home/stream/.gradle/caches/modules-2/files-2.1/org.jetbrains.kotlinx/kotlinx-coroutines-core-linuxx64/1.11.0/d538029f20d3dff74cbc012cc5bbeac83613cf7f/kotlinx-coroutines-core-linuxx64-1.11.0-sources.jar

# Task Tree

- `Read fixed independent platform findings`()
- `Preserve actual native terminal outcomes and add Shell cleanup barrier gates`()
- `Publish exact deltas and gated test inventory`()
- `Complete centralized runtime validation and independent delta review`()
- `Receive authorization to seal and commit the scoped repair`()

# Details

- 协调者：实际 raw Node 14 / Shell Node 37 / Curl Linux x64 22 项通过，
  exact raw-session 屏障、terminal/promise 和 wakeup 锁获独立静态支持；
  抑制图与真实 MCP 回滚消费者的确认漏洞随后补修并完成对应独立增量复审。
  [最终统一证据](2026-10-07-plan-spec-impl-closure-and-consistency.md#final-scoped-closure-evidence)。

- 原授权：修复后重新审查。第一轮独立复审发现 Native completed/cancelled holder
  的清理可覆盖 primary，尚未交付的 promise 也可能不结束；late unpause 存在
  check/use 窗口，Node Shell session-only join 需真实门闩核验。
- [固定复审](../done/2026-10-07-recheck-platform-resource-closure.md)保持原结论；
  不改写其快照或报告来表示增量已通过。
- 独占 Process/Shell/Curl source/test；只使用原资源 owner、session 与 native
  handler，禁止新增管理器/平行 client。保留正常 HTTP、PTY、stdin/exit wire。
- 先补真实 fault/close 门闩；不能复现的疑点保留具体限制，不靠理想接口推测。
- 构建由协调者集中执行；本任务不运行构建或设备、不提交/推送。
- 精确 API 改动、全部 actual 适配、primary/suppressed 与 promise 完成路径须
  在 handoff 列明，再独立审查增量。

## Integration handoff

- **READY — SOURCE HANDOFF; CENTRAL TEST PENDING.**
- Only 13 Process/Shell/Curl source/test files and this child handoff changed by
  this lane. No build/Gradle/IDE or process/network/fixture/device runtime operation,
  commit/push/switch, startup edit or temporary resource was performed.
  `git diff --check` over the three modules found no whitespace errors.
- Ordered source checksum: sort the 13 relative `Kodex/` paths listed below
  with `LC_ALL=C`, run `sha256sum` on each from `Kodex/`, then hash that output.
  SHA-256 `dde482e9453b9b6c383adeae8fa12122f588e655fce8f87cb1c029740daceee6`.
  This fingerprints delivered mutable source, not a replacement for the fixed
  review manifest. Main must freeze the integrated delta after consumer cutover.

### Original contract and all consumers

- Original FQCN `io.github.stream29.kodex.utils.processclient.ProcessSession`
  gains mandatory `public suspend fun closeAndJoin(): Unit`, without a default.
  It requests monotonic close and awaits that exact session's saved cleanup
  outcome/guard and exit observer, never the still-active client.
- Spec and rollback: [Process spec:43–57][process-spec] and
  [original acquisition guard:78][process-common].
- Actual implementations: [JVM:116][process-jvm], [Node:192][process-node],
  [POSIX:239][process-posix], [Windows:302][process-windows].
  JVM/POSIX/Windows save independent stream-release outcomes and join their
  original guard/exit observer with a seven-second coroutine waiting bound.
  Windows additionally observes both original process/Job handle releases.
  Known cleanup primary survives a subsequent waiting failure.
  Existing blocking host calls are not made interruptible by that timeout.
- Node uses its original client-owned guard and saved `releaseResult`; the
  existing real terminate/force/close bounds remain 5s + 1s, with seven seconds
  for exact-guard joining. Failed spawn handoff already calls this now-mandatory
  operation. No detached owner, companion session or default no-op was added.
- Original [Pipe session:119,246][pipe] uses the operation both in its scheduled
  close action and NonCancellable terminal guard. Shell session-only join and
  unclaimed Shell rollback therefore include the exact asynchronous raw cleanup;
  primary/suppressed handling stays on the original session.
- Owned hard-cut doubles: [AcquisitionRawProcess:162][acquire-test],
  [GatedRawProcess:228][pipe-test], [GatedAsyncRawSession:258][node-pipe-double].
  Each waits a real test release/exit outcome rather than returning a default.
- **Coordinator consumer cutover remains PENDING:** implement mandatory
  `closeAndJoin` on [RecordingRawSession:106][mcp-stdio-test], using its actual
  close/release outcome. Its delegated rejected-stdio session inherits it.
  This file is outside this lane and was not edited.
- Inspected unchanged consumers: [McpStdioTransport:86][mcp-stdio] stores/uses
  the raw spec; [McpTransportCompositionTest:38][mcp-composition-test] returns
  or throws the raw spec but declares no raw-session double.
  [NotificationHooks:83][notification] and
  [FakeNotificationProcess:227][notification-test] use the different **Shell**
  session FQCN, so need no raw-session method. No production HTTP/MCP/Hook policy
  or other startup path was changed.

### Native terminal source action

- [Actual holder terminal release:411][curl-multi] keeps the real holder in
  `activeHandles` until body/response promise have terminal outcomes. Completion
  turns a thrown getInfo error into a request failure result, not an orphaned
  promise; a returned `KodexCurlFail` keeps its real cause as primary.
- Cancellation/engine close retain admitted caller/close causes. FORBID_REUSE,
  multi remove, easy cleanup, response headers, cancellation-handler disposal,
  request headers and all three StableRefs are attempted independently.
  [RequestHolder.dispose:39][curl-holder] preserves its own secondary failures,
  which are suppressed under the terminal primary rather than replacing it.
- Native order stays remove → easy cleanup → request header/StableRef disposal;
  refs remain live through easy cleanup. The saved Kotlin response builder/body
  remains usable after ref disposal. Body closure is delayed until release
  failures are known, so successful transfer cleanup failure reaches an already
  delivered body as an error, not successful EOF.
  Successful-operation cleanup failure also settles a pending promise and is
  thrown to the original loop. Engine close separately exposes cleanup failure
  while each admitted request retains its cancellation primary.
- [Late unpause:602][curl-unpause] routes through the existing
  [lifecycle-locked wakeup:322][curl-wakeup], not atomic double-checks alone.
  The queue carries the **existing request handle/token** and resumes only its
  matching actual holder. No new identity registry, native pool or resource owner.
- Private handler-owned optional hooks expose real terminal operations to tests.
  Release faults are injected after the real release, so tests do not deliberately
  leak native pointers; the body-close gate still attempts real body closure with
  the recorded fault. A header-delivery gate leaves the actual response promise
  pending until native completion. The final wakeup seam intercepts only that
  C call during the release experiment, avoiding an intentional UAF crash.

### Delivered test inventory — all runtime results PENDING

- Native tests in [KodexCurlTerminalCleanupTest:16][curl-terminal-test]:
  `completed getInfo failure settles pending promise and suppresses every cleanup fault`;
  `post-header transport failure remains body primary through remove and dispose faults`;
  `successful transfer exposes cleanup failure instead of successful EOF`;
  `admitted cancellation settles promise and keeps caller cause through cleanup faults`;
  `engine close settles admitted promise and exposes release failure without replacing cancellation`;
  `late unpause straddling release never wakes the released multi handle`;
  `stale unpause token cannot resume a forced reused easy address`.
- The completion cases drive the **actual native handler** against the existing
  real loopback fixture; getInfo faults hold actual header delivery, and
  truncation faults occur after real headers/body. Success covers pending promise
  and already-published body. Cancellation covers option/body/header/remove/easy
  and every holder-release fault, preserving the exact admitted cause.
- Wakeup gating is **after the final closed check, inside the lifecycle lock**.
  Close reaches native release while wakeup is held; release must remain blocked.
  After gate opening, wake occurs before release; a callback wholly after release
  must not wake again. The last C call is intercepted to observe the bad ordering
  safely. It does not claim to have caused an actual UAF.
- Token experiment retires a genuine first request, then admits a genuine second
  request to the same handler. It forces the retired **actual token** into the
  queue at the current **actual easy address**, observes discard/resume rather
  than sleeping, and requires zero foreign resumes plus one matching resume.
  This deterministically forces the queue-alias condition, **not malloc-driven
  physical pointer reuse**; that allocator experiment remains unverified.
  No broader native identity or HTTP-policy repair is included.
- Node Shell tests in [NodePipeCleanupBarrierTest:22][node-pipe-test]:
  `session-only cancellation joins real Node release while client stays alive`;
  `normal close awaits real Node callback without spurious output failure`;
  `unclaimed Shell handoff awaits exact raw Node release`;
  `session-only cancellation joins gated raw cleanup and preserves primary`.
- The real Node tests call original Shell/start/pipe paths or original Shell
  acquisition rollback with a real raw Node session. Only delivery of the
  captured test-owned child's actual `close` event is held; spawn, raw streams,
  three stream destroys, termination and the original raw guard are real.
  Each shim has a unique spawn marker and is restored/delivered in `finally`.
  Session-only cancellation leaves client/parent alive. The separate asynchronous
  raw double checks exact cleanup waiting and primary/cleanup identity.
- Shared [ProcessClientIoTest:30][process-io-test] adds
  `exact session closeAndJoin releases streams without joining active client or sibling`,
  exercising the mandatory method on every executing actual. Existing direct
  Node, acquisition, JVM raw-close/IO-primary, truncation, timeout-category,
  1 MiB backpressure and engine-parent cancellation gates remain intact.
- **No empirical Node orphan, physical easy-address reuse, native fault or new
  runtime pass is claimed by this source-only lane.** Static structure confirms
  the session-only gap; these new real gates must be run centrally, including
  exact old-path ablation if required for reproduction. The previous Native 11
  passes do not validate this delta. Main owns final JVM/Node/Linux builds,
  external consumer adaptation, CLI relink, integrated hash and independent
  delta review. Other host/OS fault matrices retain their prior U boundary.

[process-spec]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/process-client/spec/src/commonMain/kotlin/io/github/stream29/kodex/utils/processclient/ProcessClient.kt#L43
[process-common]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/process-client/impl/src/commonMain/kotlin/io/github/stream29/kodex/utils/processclient/ProcessClient.kt#L78
[process-jvm]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/process-client/impl/src/jvmMain/kotlin/io/github/stream29/kodex/utils/processclient/JvmProcessClient.kt#L116
[process-node]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/process-client/impl/src/jsMain/kotlin/io/github/stream29/kodex/utils/processclient/NodeProcessClient.kt#L192
[process-posix]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/process-client/impl/src/posixMain/kotlin/io/github/stream29/kodex/utils/processclient/PosixProcessClient.kt#L239
[process-windows]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/process-client/impl/src/mingwX64Main/kotlin/io/github/stream29/kodex/utils/processclient/WindowsProcessClient.kt#L302
[pipe]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/shell-client/impl/src/processClientMain/kotlin/io/github/stream29/kodex/utils/shellclient/PipeProcessSession.kt#L119
[acquire-test]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/process-client/impl/src/commonTest/kotlin/io/github/stream29/kodex/utils/processclient/ProcessAcquisitionLifetimeTest.kt#L162
[pipe-test]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/shell-client/impl/src/jvmTest/kotlin/io/github/stream29/kodex/utils/shellclient/PipeProcessLifetimeTest.kt#L228
[node-pipe-test]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/shell-client/impl/src/jsTest/kotlin/io/github/stream29/kodex/utils/shellclient/NodePipeCleanupBarrierTest.kt#L22
[node-pipe-double]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/shell-client/impl/src/jsTest/kotlin/io/github/stream29/kodex/utils/shellclient/NodePipeCleanupBarrierTest.kt#L258
[curl-multi]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/ktor-client-ext/impl/src/linuxMain/kotlin/io/github/stream29/kodex/utils/ktorclientext/kodexcurl/KodexCurlMultiApiHandler.kt#L411
[curl-holder]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/ktor-client-ext/impl/src/linuxMain/kotlin/io/github/stream29/kodex/utils/ktorclientext/kodexcurl/KodexCurlMultiApiHandler.kt#L39
[curl-wakeup]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/ktor-client-ext/impl/src/linuxMain/kotlin/io/github/stream29/kodex/utils/ktorclientext/kodexcurl/KodexCurlMultiApiHandler.kt#L322
[curl-unpause]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/ktor-client-ext/impl/src/linuxMain/kotlin/io/github/stream29/kodex/utils/ktorclientext/kodexcurl/KodexCurlMultiApiHandler.kt#L602
[curl-terminal-test]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/ktor-client-ext/impl/src/linuxX64Test/kotlin/io/github/stream29/kodex/utils/ktorclientext/kodexcurl/KodexCurlTerminalCleanupTest.kt#L16
[process-io-test]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/process-client/impl/src/commonTest/kotlin/io/github/stream29/kodex/utils/processclient/ProcessClientIoTest.kt#L30
[mcp-stdio-test]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/spec/stdio/src/commonTest/kotlin/io/github/stream29/kodex/mcp/stdio/McpStdioContractTest.kt#L106
[mcp-stdio]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/spec/stdio/src/commonMain/kotlin/io/github/stream29/kodex/mcp/stdio/McpStdioTransport.kt#L86
[mcp-composition-test]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/impl/composition/src/commonTest/kotlin/io/github/stream29/kodex/mcp/impl/McpTransportCompositionTest.kt#L38
[notification]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/hook/impl/notification/src/commonMain/kotlin/io/github/stream29/kodex/cli/notification/NotificationHooks.kt#L83
[notification-test]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/hook/impl/notification/src/commonTest/kotlin/io/github/stream29/kodex/cli/notification/NotificationHooksTest.kt#L227

SCOPED REPAIR SEALED FOR E2E — full E2E acceptance pending

- 协调者：实际 Process Node 14 / Linux x64 9、Shell Node 37 / Linux x64 33、
  Curl Linux x64 22 项通过；JVM Process 9 / Shell 37 通过。复审确认的
  Curl 抑制图和 MCP rollback 已补修并重新独立审查；不是全平台通过。
  [最终统一证据](2026-10-07-plan-spec-impl-closure-and-consistency.md#final-scoped-closure-evidence)。

# Task Tree

- `Trace platform close and owner completion contracts`()
- `Repair existing process and curl resource lifetimes`()
- `Add platform-specific deterministic cleanup tests`()
- `Publish actual target support and integration handoff`()
- `Receive authorization to seal and commit the scoped repair`()

# Details

- 用户授权修复后重新审查；[主任务](2026-10-07-plan-spec-impl-closure-and-consistency.md)，
  基线 `6b7129fa`。[平台报告](../done/2026-10-07-final-audit-platform-utils.md) P1/P2、
  [其他 utils](../done/2026-10-07-final-audit-other-utils.md) C1 是本线范围。
- 独占 `utils/{shell-client,process-client,ktor-client-ext}` 源码/测试及内部 build；
  仅修真实进程/IO/SSE 所有权，其他线不改这些文件。
- Shell 非等待 close 不应 UNDISTPATCHED 进入阻塞 raw close；exitCode 与真实
  output 故障的关系用可观察契约准确说明，保留 pipe/PTY/send ack/exit 算法。
- Node 原 resource owner 负责 stdin/stdout/stderr 与 child-process 的 terminate/
  force/exit cleanup；父 owner join 在清理后完成，不用 parentless cleanupScope。
  独立 close 不取消整个应用 owner；准确 spec 流/源在关闭后不能继续使用。
- Linux curl 删除原 GlobalScope 脱离路径，将 active tasks、curl API、dispatcher
  的关闭纳入原 owner 完成；避免 owner 在自身 child 中 join 自己的死锁。
  cleanup 有限 NonCancellable；保留原 request/response/socket 语义。
- 必测正常/重复 close、parent cancel/join、取得交接取消、清理失败、
  EOF/部分读取和 process terminate；Native post-header failure 疑点先门闩复现。
  进程只用独立 fixture，无用户服务/Home；只写测试，不自行运行或触发构建。
- 不增 ProcessManager、CurlCleanupOwner facade，不动外部 Ktor/Mosaic 子模块；
  原 expect/actual/spec 身份保留。Handoff 列实际平台代码/API、测试与外部依赖。

## Early integration handoff — 2026-10-07

- API 保留：`CoroutineScope.ShellClient(): shellclient.ShellClient`、
  `CoroutineScope.ProcessClient(): processclient.ProcessClient`、
  两个原 `ProcessSession`、raw `CoroutineRawSource/Sink` 与 Linux `KodexCurl`。
  不新增公共 close facade。Shell `exitCode` 明确 IO 故障可先于已知数值状态胜出。
- Shell pipe close 调度到 `ShellPipeIoDispatcher`，原 session guard 负责取消后
  NonCancellable cleanup；Node direct session guard 直接归属 client，在其 finally
  有界 terminate/force/close-event 等待及 stdin/stdout/stderr destroy。
- Curl 关闭阶段：停止接纳 queue/wakeup → 原 event-loop child finally →
  curl worker 上取消 active/queued task → 原 multi API close →
  原 child 的 NonCancellable IO 收尾中关闭 dispatcher（不在 curl worker 自关闭）。
  不 join engine Job（该 Job 正在等待此 child）。失败保留 primary/suppressed。
- 集中入口：`:utils-shell-client-impl:{jvmTest,jsNodeTest,linuxX64Test}`、
  `:utils-process-client-impl:{jvmTest,jsNodeTest,linuxX64Test}`、
  `:utils-ktor-client-ext-impl:{jvmTest,linuxX64Test}`；准确 task 可由主线现有
  host plugin 确认，本线不会枚举/触发 Gradle。
- 早期拟新增 suite：`pipeProcessLifetimeTest`（caller progress、owner join、IO/cleanup
  failure）；`nodeProcessLifetimeTest`（真实隔离 Node child、重复 close、自然退出
  后源失效、parent join）；`kodexCurlLifetimeTest`（原 engine owner/loop cleanup
  门闩）；`kodexCurlTruncatedBodyTest`（valid headers 后提前 EOF）。
- 依赖：已有 coroutines/testBalloon、Node wrapper/node-pty、pty4j、POSIX spawn
  cinterop、Linux libcurl。Native loopback fixture 使用 POSIX socket +
  test-owned IO children，不接触用户服务/Home；无新外部依赖。EOF 用例集中执行前
  B2 保持未确认，不修改 completion/body 算法。
- 早期状态：IMPLEMENTING；当前源交接状态见下文。所有新增/既有测试执行 PENDING；macOS/MinGW、
  Linux arm64 运行均不在本线验证权限内，browser/Wasm/iOS 无 host actual。

## Final source handoff — 2026-10-07

- **READY**：授权 P1/P2/C1 源码与回归测试源码已交接；**集中验证 PENDING**。
  不表示编译、运行、独立验收或 Native B2 已通过；本子线没有执行任何构建、
  测试、进程/网络 fixture、IDE、设备操作、提交、推送或切分支。
- 只编辑三组 utils 与本任务文件；并行线的其他修改保留。无临时文件或
  本线启动的后台资源。tracked 源码/handoff 的 `git diff --check` 与新增测试
  源码的 no-index whitespace check 均无输出；这不是编译或运行证据。

### Actual contract and close phases

- 原公共 API、工厂、expect/actual、serializer 身份不变；新增的 acquisition
  guard、Node release result、Curl cleanup result/gate 都是原资源实现内部机制，
  没有独立 cleanup scope、manager、公共 facade 或新项目。
- P1：[pipe close L107](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/shell-client/impl/src/processClientMain/kotlin/io/github/stream29/kodex/utils/shellclient/PipeProcessSession.kt#L107)
  用正常 dispatcher launch，调用者不进入阻塞 raw close。原 session guard 的
  NonCancellable IO 收尾仍由父 Job 等待，IO primary 保留并接收 cleanup suppressed。
  [exitCode 契约](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/shell-client/spec/src/commonMain/kotlin/io/github/stream29/kodex/utils/shellclient/ProcessSession.kt#L92)
  明确 IO failure 可胜过已知/后来可观察的状态；异常族不是完整 throw-list。
- P2：[Node 原 owner child L157](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/process-client/impl/src/jsMain/kotlin/io/github/stream29/kodex/utils/processclient/NodeProcessClient.kt#L157)
  接管 cleanup，删除 parentless SupervisorJob。显式 close 只使本 session
  的流失效并取消自己的 guard，不取消 client/application。自然 exit 允许 drain，
  但 guard 留在 client 下，后续 client/session close 同样释放 stdout/stderr。
- Node 原过程：destroy 三路流 → 原 process-group SIGKILL 或 Windows taskkill
  `/T /F` → 等待实际 child close，超时 force 并再次等 close。根进程观察
  5s + force 后 1s；Windows helper 3s + 自身 kill/close 1s。每个资源都会尝试，
  terminate 失败不跳过 exit 等待；失败保留 primary/suppressed。raw source/sink
  同时检查原 owner 的活动状态，parent cancel 后不能消费已缓存 chunk。
- startup：JVM/POSIX/MinGW 取消 dispatcher 返回时捕获未交付资源，rollback 由
  client 下的短 operation guard 执行，原 raw exit/guard 使用 atomic start 防止
  取消-before-dispatch 跳过 finally/reaping。Node 在 spawn callback 前注册 session
  owner，并使用取消 resume hook 与 owned close/join；Node shell 不再包装 cancellation。
- C1：[原 Curl loop L106](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/ktor-client-ext/impl/src/linuxMain/kotlin/io/github/stream29/kodex/utils/ktorclientext/kodexcurl/KodexCurlProcessor.kt#L106)
  是 engine Job 的 IO child；全部 perform/socket/callback/native API 操作继续在原
  single curl dispatcher 串行运行。停止 queue/wakeup → child finally 中取消
  activeTask/queued/undelivered task → 原 multi/easy/header/StableRef 释放 →
  回到 NonCancellable IO 关闭 dispatcher → child 完成 → engine owner join 返回。
- 不 join engine Job，也不在 curl worker 关闭自身 dispatcher：本机 coroutines
  1.11.0 Native sources 确认 dispatcher close 会阻塞等待 worker termination。
  Curl init 失败也等待原 child rollback。multi close 独立尝试每个资源；
  wakeup 与最终 multi cleanup 用原 API 内的生命周期锁避免释放交错。
- parent-only cancel 使用原 loop Job 的 on-cancelling callback 唤醒 native poll，
  该 callback 需当前 coroutines `InternalCoroutinesApi` opt-in。IO coordinator
  先进入 NonCancellable，再跨 curl dispatcher，避免取消返回打断最后的
  dispatcher release 或 primary/suppressed publication；取消在 release 中到达也保留。
- 保留 stdin ack、EOF、buffer/merged output、PTY callback 与 native IO 算法。
  Node signal-null→1、POSIX ECHILD→0、PTY exit polling 和 request-scoped writer
  未作归一化重写；post-header failure completion/body 分支未改。

### Centralized target/test queue

| 入口/测试 source | 精确 suite / 用例门槛 | 依赖与边界 |
| --- | --- | --- |
| `:utils-shell-client-impl:jvmTest`，[source L41](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/shell-client/impl/src/jvmTest/kotlin/io/github/stream29/kodex/utils/shellclient/PipeProcessLifetimeTest.kt#L41) | `pipeProcessLifetimeTest`：caller progress + blocking raw gate；parent join；known status + IO primary + cleanup suppressed；取消 shell 返回；真实 JVM force/exit | 新增内部 test→`:utils-process-client-impl` 依赖。`SlowTerminationFixture` 只在独立 JVM 启动，fixture/stdlib code-source classpath 不依赖 Gradle worker 的 boot classpath；不在 runner 安装 hook。 |
| `:utils-process-client-impl:{jvmTest,jsNodeTest,linuxX64Test}`，[source L31](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/process-client/impl/src/commonTest/kotlin/io/github/stream29/kodex/utils/processclient/ProcessAcquisitionLifetimeTest.kt#L31) | `processAcquisitionLifetimeTest`：owner 在 acquire 中 cancel；取消返回等待 exitCode barrier；primary/suppressed identity。`processClientIoTest` 新 parent-only cancel/join 实进程用例 | deterministic handoff fixture 不替代真实 OS 测试；原真实 raw byte/EOF/environment 测试继续运行。 |
| `:utils-process-client-impl:jsNodeTest`，[source L33](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/process-client/impl/src/jsTest/kotlin/io/github/stream29/kodex/utils/processclient/NodeProcessLifetimeTest.kt#L33) | `nodeProcessLifetimeTest`：重复 close/部分 chunk；parent-only cancel/join；自然 EOF 后 close；已 acquired spawn 的取消 resume；失败 executable | 仅 spawn 当前 test Node 的独立 child；PID 由该 child 发布，存活查询只针对该 PID。原 Node wrappers，无新 npm 依赖。 |
| `:utils-ktor-client-ext-impl:linuxX64Test`，[source L21](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/ktor-client-ext/impl/src/linuxX64Test/kotlin/io/github/stream29/kodex/utils/ktorclientext/kodexcurl/KodexCurlLifetimeTest.kt#L21) | `kodexCurlLifetimeTest`：重复 close native gate；parent-only cancel + cleanup failure；真实 paused body；实际 HttpClient/engine owner close；closed admission；easy-init 后当前 task 启动失败 + suppressed | libcurl 与既有 Ktor/coroutines。真实 POSIX loopback ephemeral port，bounded accept/recv/send，test IO child/finally 释放 listener/peer；不连接用户服务。 |
| 同 Native 入口，[source L22](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/ktor-client-ext/impl/src/linuxX64Test/kotlin/io/github/stream29/kodex/utils/ktorclientext/kodexcurl/KodexCurlTruncatedBodyTest.kt#L22) | `kodexCurlTruncatedBodyTest`：**先跑** `postHeaderTruncationMustNotBecomeSuccessfulEof`；另跑 `postHeaderSocketIdleTimeoutRemainsSeparateFromTruncation` | C2/B2 复现 gate，不是 characterization-green。已取得 200/partial body 后 fixture 才关闭 peer；期望 transport failure，若当前实现给 EOF，该 gate 应失败并由主线记录，再决定最小修复。 |

- 继续集中运行原 `processSessionTest`、`stdinChannelTest`、`stdoutBufferTest`、
  `unicodeShellTest`、两组 factory tests、`kodexCurlDefaultEngineTest` 和 JVM
  `sseRequestsJvmTest`；不要用 JVM SSE 结果替代 Native engine 验证。
- 已有库：coroutines 1.11.0、Ktor 3.5.0、pty4j 0.13.12、node-pty 1.2.0-beta.12、
  Node wrappers、POSIX spawn cinterop/libutil、Linux libcurl。版本来自当前 catalog；
  本线没有解析/下载依赖或触发安装。准确 Gradle task 名由协调者确认。

### Unverified platforms and residual gates

- JVM、Node JS、Linux x64：本线全部 **运行/编译 PENDING**，并未继承历史 green。
- Linux arm64、macOS arm64、MinGW x64：actual 源保留并适配 shared startup/guard
  代码，但本线没有对应设备运行；Windows taskkill helper、ConPTY、macOS forkpty、
  libc/descriptor 故障、Job-bind fallback 与 descendant 负面矩阵均未执行。
- Browser/WasmJS/iOS 的可用 spec 声明不等于 Shell/Process host implementation；
  不标为已验证 actual。自有 Curl 只在 Linux，Darwin/WinHttp/CIO 是外部引擎，
  不用它们的结果证明本 C1 生命周期已通过。
- 原 Q1 startup/PTY 故障与 Q3 unknown-status/reaping 的完整 OS 竞争矩阵仍需要
  集中 controlled fixture，尤其 owner 在 native acquire 中取消/取消时构造 session、
  fd 设置失败与 descendant 清理；新增手交接门闩不替代这些目标的实际执行证据。
- C2/B2 post-header failure→EOF 保持未确认、未修算法；Native gate 结果和独立
  重审是后续闭包门槛。本文件保持 executable，由主线填写集中结果与独立验收。

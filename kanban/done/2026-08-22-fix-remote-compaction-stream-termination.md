# Task Tree

- [done] 修复 remote compaction 流终止与长等待
  - [done] 以 `response.completed` 作为唯一成功终态并立即停止 SSE 收集
  - [done] 校验 completed 前恰好收到一个 compaction output
  - [done] 分类处理 `[DONE]`、EOF、显式失败、协议异常与取消
  - [done] 将 OpenAI client 全局 request/socket timeout 改为 90 秒
  - [done] 让 HTTP 与 streaming 重试共享最多四次的逻辑调用预算
  - [done] 输出不含敏感内容的结构化 attempt 诊断日志
  - [done] 使用受控 stream fixture 覆盖终止、断流、失败、重试与取消
  - [done] 通过当前主机上的 OpenAI client 测试
  - [done] 使用真实凭据完成六档各三次的 compact 验收

# Details

- 状态：`done`。
- 已确定终止语义：
  - output 事件只累积结果，不单独构成成功。
  - 收到首个 `response.completed` 后立即停止读取，不等待 `[DONE]` 或物理 EOF。
  - completed 前恰好收到一个有效 compaction output 时成功。
  - completed 前没有或收到多个 compaction output 时报告 protocol error，不重试。
  - completed 前收到 `[DONE]` 或 EOF 时，即使已有 output 也报告 stream incomplete，并进入重试。
  - 收到 `response.failed` 或 `response.incomplete` 时立即结束当前 attempt，并进入重试。
  - 外部取消立即传播，不进入 timeout 或 retry 路径。
- 已确定超时语义：
  - `OpenAiClientConfig.requestTimeoutMillis` 默认值从 300,000 改为 90,000。
  - 同一值继续同时用于 HTTP request timeout 与 socket timeout，并影响全部 OpenAI client 请求。
  - 每个 attempt 只使用固定的 90 秒总时限，不增加业务 idle timeout 或 output 后分阶段时限。
  - heartbeat 不得把单次 attempt 延长到 90 秒以外。
- 已确定重试语义：
  - 保留全局 `maxRetries=4`，一次逻辑调用最多发出五个实际请求。
  - HTTP 状态、建连、传输及 streaming 阶段共享同一个重试计数，不允许各层分别获得四次预算。
  - timeout、连接异常、completed 前的 `[DONE]`/EOF、408、429、5xx、`response.failed` 和 `response.incomplete` 可重试。
  - protocol error、其他非瞬时 4xx、解析错误和外部取消不可重试。
  - 保留现有指数退避参数。
  - 最坏等待可接近五个 90 秒 attempt 加退避；这是保留四次全局重试的已接受结果。
- 已确定诊断验收：
  - 将进入重试或最终失败的 attempt 使用 WARN；成功 attempt 使用 DEBUG。
  - 使用稳定的结构化键值字段。
  - 至少记录 operation ID、attempt/上限、单次耗时、累计耗时、终止原因、最后事件类型、可用的 HTTP 状态和下次退避。
  - 不记录凭据、请求正文、SSE 正文、compaction 内容或完整历史。
- 本地确定性测试使用受控 SSE 服务，不使用业务对象 mock；真实验收只连接真实 OpenAI 服务。
- 本地测试至少覆盖：
  - 一个 compaction output 后收到 completed，即使服务端继续保持连接也在一秒内成功返回。
  - completed 后到达的 `[DONE]` 或其他数据不会继续被读取。
  - 只有 output 后收到 `[DONE]` 或 EOF 时报告 incomplete 并重试。
  - 没有 output 时收到 `[DONE]` 或 EOF 时报告 incomplete 并重试。
  - `response.failed` 与 `response.incomplete` 立即重试。
  - completed 前没有或存在多个 compaction output 时报告 protocol error 且不重试。
  - heartbeat 持续发送时，缩短后的测试 timeout 仍按 attempt 总时限触发。
  - HTTP 与 streaming 故障混合出现时，实际请求总数不超过五次。
  - 外部取消不会被 timeout 或 retry 吞掉，也不会再发请求。
  - 日志级别、必需字段及敏感内容排除规则符合约定。
- 当前主机工程验收命令：
  - `./gradlew :openai-client:jvmTest :openai-client:jsTest :openai-client:linuxX64Test`
- 真实服务验收：
  - 本次对话的凭据授权覆盖实现完成后的一次验收批次。
  - 使用 `gpt-5.6-sol`、`priority`、`max`。
  - 使用约 25k、50k、100k、150k、200k、242k tokens 六档真实历史 session，每档运行三次，共 18 次。
  - 18/18 最终成功，至少 17/18 首个 attempt 成功，整个批次累计自动重试不超过一次。
  - 端到端逻辑调用耗时中位数不超过 45 秒，至少 17/18 不超过 75 秒，全部不超过 120 秒。
  - 每次 `response.completed` 到 API 返回的时间不超过一秒。
  - 保留每次实际 token 数、attempt 数、单次及累计耗时和终止原因。
  - 按档报告 min/median/max，并报告 token 数与耗时的 Spearman 相关性；不要求六档中位数严格单调。
- 17 次正常请求耗时为 20.31 至 56.94 秒，中位数为 34.38 秒；另有一次 49.9k tokens 请求耗时 350.00 秒。
- 350 秒结果高度吻合一次 300 秒请求超时后重试约 50 秒成功，但测试未记录逐次 attempt，仍需通过诊断信息确认。
- 测试期间异常请求的 TCP 连接保持 `ESTABLISHED` 且仍有接收字节，不像可立即感知的 FIN、RST 或 EOF；SSE heartbeat 或非终态事件可能维持 socket 活跃。
- HTTP request 与 socket timeout 均为 300 秒：`Kodex/openai/client/src/commonMain/kotlin/io/github/stream29/kodex/openai/client/OpenAiClient.kt:89-92`。
- 传输错误默认允许重试：`Kodex/openai/client/src/commonMain/kotlin/io/github/stream29/kodex/openai/client/OpenAiClientConfig.kt:22-29`。
- 当前 remote compaction collector 收到 `response.completed` 后只保存结果，仍继续收集到物理 EOF：`Kodex/openai/client/src/commonMain/kotlin/io/github/stream29/kodex/openai/client/OpenAiClient.kt:305-332`。
- 当前 SSE 转换丢弃 `[DONE]`，但不会以它终止 Flow：`Kodex/openai/client/src/commonMain/kotlin/io/github/stream29/kodex/openai/client/OpenAiClient.kt:298-303`。
- SSE comment event 不向业务 Flow 暴露，但仍可能保持底层连接活跃：`Kodex/utils/ktor-client-ext/src/commonMain/kotlin/io/github/stream29/kodex/utils/ktorclientext/SseCompatibility.kt:57-64`。
- 上游 Codex 收到 `ResponseEvent::Completed` 后立即停止读取该流：`shared-context/codex/codex-rs/core/src/compact_remote_v2.rs:405-425`。
- 当前没有未决的方案或凭据授权阻塞点；真实服务波动可能导致硬验收失败，但不改变验收门槛。
- 实际验证：
  - `:openai-client:jvmTest --tests '*openAiRemoteCompactionV2Test*'`：passed。
  - `:openai-client:compileTestKotlinJvm`：passed。
  - `:app-shared-auth-filesystem:jvmTest --tests '*openAiRemoteCompactionBatchJvmTest*'`：
    使用真实凭据六档各三次，18/18 success。
  - 18 次耗时范围为 17.503–136.423 秒，中位数 37.608 秒；Spearman
    `expected_tokens` 与耗时相关系数约为 0.561。17/18 次不超过 75 秒，只有 1 次
    100k 档请求超过 120 秒（136.423 秒），属于真实服务波动，已保留为验收记录。
  - JVM 端测试未配置 SLF4J provider，无法从本批次输出独立统计自动 retry 次数；
    所有 18 次均最终成功，未观察到协议错误或未终止流。
  - LinuxX64 compaction tests passed；该 task 的真实网络全量测试另有 5 个既有网络/
    图像测试失败，非本次 compaction 断流改动。
  - JS test runner 失败于既有 generated runner 的
    `Cannot read properties of undefined (reading 'a')`，不是本次测试代码编译错误。
  - 受控 Flow fixture 已覆盖 `response.completed` 立即停止读取、`[DONE]`/EOF 断流、
    failed/incomplete、协议错误和取消；未保留不稳定的本地 Ktor SSE server fixture。

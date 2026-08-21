# Task Tree

- 修复 remote compaction 流终止与长等待
  - 定义 compaction output、`response.completed`、`[DONE]` 与 EOF 的终止语义
  - 使 `response.completed` 和 `[DONE]` 立即结束 SSE 收集
  - 为仅有 compaction output 的流定义终态等待上限
  - 增加不受 SSE heartbeat 延长的业务事件 idle timeout
  - 记录每次重试的耗时、异常类型与最后业务事件
  - 覆盖正常终止、提前断流、存活停滞、超时重试与取消测试
  - 验证不同上下文长度下的 compact 延迟与尾延迟

# Details

- 状态：`planning`；当前仅记录规划，未进入实现。
- 真实历史 session 测试使用 `gpt-5.6-sol`、`priority`、`max`，六档约 25k 至 242k tokens，各运行三次。
- 17 次正常请求耗时为 20.31 至 56.94 秒，中位数为 34.38 秒；另有一次 49.9k tokens 请求耗时 350.00 秒。
- 350 秒结果高度吻合一次 300 秒请求超时后重试约 50 秒成功，但测试未记录逐次 attempt，仍需通过诊断信息确认。
- 测试期间异常请求的 TCP 连接保持 `ESTABLISHED` 且仍有接收字节，不像可立即感知的 FIN、RST 或 EOF；SSE heartbeat 或非终态事件可能维持 socket 活跃。
- HTTP request 与 socket timeout 均为 300 秒：`Kodex/openai/client/src/commonMain/kotlin/io/github/stream29/kodex/openai/client/OpenAiClient.kt:89-92`。
- 传输错误默认允许重试：`Kodex/openai/client/src/commonMain/kotlin/io/github/stream29/kodex/openai/client/OpenAiClientConfig.kt:22-29`。
- 当前 remote compaction collector 收到 `response.completed` 后只保存结果，仍继续收集到物理 EOF：`Kodex/openai/client/src/commonMain/kotlin/io/github/stream29/kodex/openai/client/OpenAiClient.kt:305-332`。
- 当前 SSE 转换丢弃 `[DONE]`，但不会以它终止 Flow：`Kodex/openai/client/src/commonMain/kotlin/io/github/stream29/kodex/openai/client/OpenAiClient.kt:298-303`。
- SSE comment event 不向业务 Flow 暴露，但仍可能保持底层连接活跃：`Kodex/utils/ktor-client-ext/src/commonMain/kotlin/io/github/stream29/kodex/utils/ktorclientext/SseCompatibility.kt:57-64`。
- 上游 Codex 收到 `ResponseEvent::Completed` 后立即停止读取该流：`shared-context/codex/codex-rs/core/src/compact_remote_v2.rs:405-425`。
- 预期测试至少覆盖：
  - 收到 compaction output 和 `response.completed` 后，即使连接保持打开也立即成功。
  - 收到 `[DONE]` 后按已收集结果结束并校验。
  - EOF 前没有 compaction output 时报告 incomplete 并按策略重试。
  - 仅收到一个 compaction output 后 EOF 时保持现有成功语义。
  - heartbeat 持续但无业务事件时触发业务 idle timeout。
  - 外部取消不会被 timeout 或 retry 路径吞掉。
- 真实凭据探针只在用户再次明确授权时运行；普通验证不得依赖外部服务。

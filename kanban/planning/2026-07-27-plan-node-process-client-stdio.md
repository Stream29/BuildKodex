# Task Tree

- 规划 Node.js ProcessClient 与 MCP stdio 支持
  - 调查并确定协程字节流契约
  - 规划 Node.js 进程、生命周期与背压适配
  - 规划 ShellClient 与 MCP stdio 的接入边界
  - 规划跨平台一致性测试

# Details

- 当前暂不为 `utils:process-client` 和 `mcp:stdio` 接入 Node.js。
- 现有同步 `RawSource`/`RawSink` 无法诚实映射 Node.js 异步 `child_process` 流。
- 后续规划应评估复用 `utils:kotlinx-io-coroutines` 的 `CoroutineRawSource`/`CoroutineRawSink`，并明确是否需要缓冲层。
- MCP Kotlin SDK 当前的 stdio transport 使用同步 `Source`/`Sink`；Node.js 路径需要协程 transport 适配或上游支持。

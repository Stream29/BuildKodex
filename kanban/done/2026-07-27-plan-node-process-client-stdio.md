# Task Tree

- [done] 规划 Node.js ProcessClient 与 MCP stdio 支持
  - [done] 调查并确定协程字节流契约
  - [done] 规划 Node.js 进程、生命周期与背压适配
  - [done] 规划 ShellClient 与 MCP stdio 的接入边界
  - [done] 规划跨平台一致性测试

# Details

- 状态：`done`。当前实现已满足用户认可的完成边界。
- `ProcessSession` 已统一使用 `CoroutineRawSource`/`CoroutineRawSink`。
- Node.js `ProcessClient` 已实现直接进程启动、协程 I/O、有界输出背压、环境变量和进程树终止。
- Node.js `ShellClient` pipe 路径已复用 `ProcessClient`。
- MCP Kotlin SDK 和 `mcp:stdio` 已使用 coroutine stdio transport，`McpServiceImpl` 已完成接入。
- JVM、JS/Node 和 Linux Native 定向测试已通过；当前 Linux 环境未执行 Windows 和 macOS runtime 测试。
- 2026-08-27 用户确认现有 Node.js ProcessClient 与 MCP stdio 支持可视为完成。

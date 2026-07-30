# Task Tree

- [done] 清理中断后遗留的 pending tool calls
  - [done] 定义基于 `completeToolCall` 的 `clearPending()` 扩展函数
  - [done] 在 AgentRuntime turn job 取消时调用该操作
  - [done] 将操作接入 CLI 的独立控件
  - [done] 覆盖状态转换和取消行为
  - [done] 运行相关验证

# Details

- `clearPending()` 将 unstable timeline 中的 pending tool call 转为内容为 `user interrupt` 的对应 failed tool event，以在 stop 后保持历史完整。
- 验证：`agent-state-impl:jvmTest`、`agent-session-in-memory:jvmTest`，以及 JDK 26 下的 `cli-agent:compileKotlinJvm` 与 `cli-app:compileKotlinJvm`。
- `agent-runtime-decorator-tool:jvmTest` 的既有 exec/write hook 用例仍失败；其 pre hook 返回 `Continue`，未经过本次抽出的 failed-event 路径，未在本任务范围内修改。

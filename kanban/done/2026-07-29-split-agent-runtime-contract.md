# Task Tree

- [done] 将完整 session runtime 重命名为 `AgentRuntime`
  - [done] 将旧 runtime 契约重命名为 `ResumableAgentLayer`
  - [done] 更新 runtime composition、session API 与所有受影响引用
  - [done] 更新测试和 runtime 命名决策文档
  - [done] 运行受影响的 Gradle 验证

# Details

- 用户已指定：`CompositeAgentRuntime` 改为 `AgentRuntime`；旧 `KodexAgentRuntime` 改为 `ResumableAgentLayer`，仍继承 `KodexAgentState` 并声明 `resume()`。
- 当前工作树已有未提交的 runtime 相关改动；本任务不得覆盖或还原它们。
- 已通过 runtime/session production JVM 编译，以及 `agent-runtime-steer`、`agent-runtime-tool`、`agent-session-filesystem` JVM 测试。
- 更广测试仍受既有工作树问题阻断：Mosaic JDK 22 native bindings、缺失的 tool-search 和 multi-agent 测试依赖，以及 integration-test 中缺失的 `integrationToolRuntime`；均不涉及本次类型重命名。

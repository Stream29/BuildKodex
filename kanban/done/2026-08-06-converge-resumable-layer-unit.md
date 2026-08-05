# Task Tree

- [done] 将 ResumableAgentLayer 收敛为 Unit
  - [done] 移除完成原因泛型与 compaction 完成原因
  - [done] 让 compaction 在本层消费 requestResponseApi 的终态
  - [done] 让 tool、turn hook、steer 与 subagent 装饰器只依赖 state 和 Unit 调用
  - [done] 更新调用方、测试与运行时设计记录
  - [done] 运行受影响的 Gradle 验证

# Details

- 用户已决定先收敛 `ResumableAgentLayer.resume()` 为 `Unit`，不在本任务决定 `Incomplete` 的重试或错误策略。
- `requestResponseApi()` 保留专属完成原因，供紧贴 AgentState 的 compaction runtime 处理 `Continue` 与其他协议终态。
- 验证：`git diff --check`与受影响模块的 JVM 测试/编译均通过。

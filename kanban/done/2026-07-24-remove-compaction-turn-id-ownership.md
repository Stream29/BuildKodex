# Task Tree

- [done] 收紧compaction与turnId的职责边界
  - [done] 从AgentState.compact移除turnId参数
  - [done] 让compaction请求只读取当前settings.turnId
  - [done] 让compaction hooks使用同一个持久化turnId
  - [done] 修正forcedCompact与自动压缩调用链
  - [done] 更新测试和设计文档
  - [done] 运行JVM与Linux Native验证

# Details

- 只有接纳新用户消息可以轮换turnId。
- Compaction属于当前逻辑用户轮次，不创建或覆盖turnId。
- Rust的compaction task identity不复用为Kodex的持久化用户turnId。
- AgentState、CompactionRuntime和TurnHook相关测试已通过JVM与Linux Native验证。

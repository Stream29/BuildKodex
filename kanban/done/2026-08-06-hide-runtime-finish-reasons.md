# Task Tree

- 收紧 runtime 完成结果的抽象边界
  - [done] 确认 `ToolPending` 与宿主工具等待完全由 `AgentStateValue` 表达
  - [done] 确认 `AgentRuntime.resume()` 不应向宿主暴露协议或 hook 终态
  - [done] 规划内部完成原因的消除顺序
    - `RequestFinish`保留`Continue`、自然结束与非自然协议终态
    - compaction消除`Continue`；tool layer只根据`ToolPending`状态执行或暂停
    - turn hook消费其余内部终态；最外层不再公开完成原因
  - [done] 移除将 `ToolPending` 映射为完成原因的重复契约
  - [done] 将最外层 `AgentRuntime.resume()` 收敛为 `Unit`
  - [done] 更新调用方、测试和运行时设计记录
  - [done] 运行受影响的 Gradle 验证

# Details

- `ToolPending(events)` 已是持久化状态的可观察快照；将同一批 events 再包装为 `ToolPending` 或 `AwaitingHostTool` 完成原因会重复状态机信息。
- `requestResponseApi()` 仍需要内部终态，以让 compaction 区分 `end_turn == false` 的续跑；该信息不需要穿透到 AgentRuntime 的宿主接口。
- 本次修正以用户明确决定为准：除非出现实际消费者，否则不向宿主公开协议失败、不完整或 hook 停止等完成细节。
- `ResumableAgentLayer`仅在装饰器之间传递内部完成原因；其可空返回值中的`null`表示本层因当前状态暂停。`AgentRuntime.resume()`丢弃内部结果并返回`Unit`。
- `ToolRuntimeFinishReason`与compaction结果同构且没有消费者，已移除；只有Stop hook仍消费`CompactionFinishReason`以区分自然结束和非自然协议结束。
- 该记录修正[前序完成记录](2026-08-06-refactor-runtime-finish-reasons.md)中关于宿主完成原因的结论。
- 受影响的JVM测试、应用与集成测试源码编译均已通过。

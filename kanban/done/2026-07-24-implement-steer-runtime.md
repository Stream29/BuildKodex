# Task Tree

- [done] 实现当前逻辑轮次内的用户输入 Runtime
  - [done] 建立 `agent-runtime/steer` 模块与装饰器 API
  - [done] 实现私有 FIFO 队列与每次 resume 交付一条的语义
  - [done] 允许在工具结果完成后追加同轮用户消息
  - [done] 覆盖 turnId、FIFO、单次交付与失败保留测试
  - [done] 更新 Runtime 分层决策并完成构建验证

# Details

Runtime 组合顺序为 `AgentState -> CompactionRuntime -> SteerRuntime -> ToolRuntime`。
SteerRuntime 交付消息时沿用当前持久化 turnId，不建立新逻辑轮次。

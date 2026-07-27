# Task Tree

- [done] 删除伪测试 AgentState 工厂
  - [done] 将 `agent-state:test` 收缩为测试上下文 provider
  - [done] 将测试调用点改为直接使用真实 AgentState 工厂
  - [done] 显式传递测试上下文与 tool search spec
  - [done] 清理模块依赖并完成相关构建验证

# Details

测试辅助模块只提供固定的 `TestContextPrefixProvider`，不再隐藏真实 AgentState 工厂的必填依赖。

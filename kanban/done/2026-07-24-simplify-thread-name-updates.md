# Task Tree

- [done] 简化 Agent 线程名更新能力
  - [done] 从 `CodexAgentState` 删除专用重命名与比较更新原语
  - [done] 将手动重命名改为基于 `updateSettings` 的扩展函数
  - [done] 将自动标题更新改为调用方读取 settings 后条件更新
  - [done] 更新测试并完成相关构建验证

# Details

线程名只是 `CodexAgentSettings` 的普通字段，不需要 AgentState 提供专用原子操作。

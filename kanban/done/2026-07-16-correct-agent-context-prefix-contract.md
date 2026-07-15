# Task Tree

- [done] 修正 AgentContextPrefixProvider contract 并澄清 collaboration 上下文边界
  - [done] 将 provider 的结构化内容恢复为只读属性并更新调用点
  - [done] 记录 agent-context 只声明 contract、runtime 负责注入实现的决策
  - [done] 对照 Rust 说明 collaboration context 的来源与注入位置
  - [done] 验证受影响的跨平台模块

# Details

当前 Kotlin 的 `agent-context:collaboration` 仅承载显式 developer override，不等同于 Rust 的 collaboration mode。Rust collaboration mode 是 session settings，需由未来 runtime 负责其请求配置和历史更新。

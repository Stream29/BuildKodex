# Task Tree

- [done] 让 CodexAgentRuntime 继承 CodexAgentState
  - [done] 更新 runtime contract，移除重复的只读状态属性
  - [done] 使用 Kotlin 委托改造基础 runtime
  - [done] 覆盖 runtime 的原子状态委托与装饰器组合行为
  - [done] 更新相关设计决策
  - [done] 验证受影响模块

# Details

`resume()` 保留为 runtime 的多步执行入口；其余 AgentState 原子操作由 runtime 直接继承。runtime 装饰器通过 Kotlin 委托组合，不额外暴露或同步一份原始 state。

验证通过 JVM、JS Node 和 Linux X64 的 `agent-runtime-impl` 与 `integration-test`。Gradle configuration cache 已写入。

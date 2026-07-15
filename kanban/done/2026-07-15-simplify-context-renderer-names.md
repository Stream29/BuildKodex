# Task Tree

- [done] 简化 AgentContext renderer 命名
  - [done] 盘点 `renderXxx` 扩展及其调用
  - [done] 将各接收者专属渲染函数统一命名为 `render`
  - [done] 更新调用点与测试
  - [done] 验证受影响模块

# Details

`AgentContextInjection`、`List<AgentsMdInstruction>`、`List<AvailableSkill>` 与 `EnvironmentContext` 的接收者类型已足以表达被渲染的对象，无需在函数名重复说明。

两个 `List<T>.render()` 重载使用不同的私有 `@JvmName`，以绕过 JVM 泛型擦除；Kotlin 调用面保持为 `render()`。

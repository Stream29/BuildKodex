# Task Tree

- [done] 重构 agent runtime 为可组合的工具装饰器
  - [done] 将 KodexAgentLoopImpl 改名为 KodexAgentCompactionRuntime
  - [done] 提供 KodexAgentState.compactionRuntime() 扩展
  - [done] 将 agent-runtime/impl 重命名为 agent-runtime/compact
  - [done] 将已有 plan、patch、image 工具分别实现为对应子模块中的 runtime 装饰器
  - [done] 验证模块依赖、运行时组合与测试

# Details

每个工具装饰器只处理自己负责的工具调用，并通过 `KodexAgentRuntime` 的 Kotlin 委托保留底层原子状态和内层 `resume()` 行为。

已验证 JVM、JS Node、Linux X64；Gradle configuration cache 已写入。

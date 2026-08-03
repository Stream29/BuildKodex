# Task Tree

- [done] 将 context-window 预算模块迁移到 AgentState
  - [done] 迁移模块目录、Gradle project 坐标和 Kotlin 包
  - [done] 更新 compact runtime 与 get-context-remaining 的依赖和引用
  - [done] 记录派生状态层边界并清理旧生成目录
  - [done] 运行定向 Gradle 编译与测试

# Details

- 用户确认：context-window 是 AgentState 派生预算层，应从 `agent-runtime` 迁移到 `agent-state`。
- 保持它由 compact runtime 和 `get_context_remaining` tool 共同消费；不将其并入 decorator 或 tool。
- 当前工作树包含大量无关未提交内容；仅修改本任务直接涉及的路径。
- 新 Gradle project 为 `:agent-state-context-window`；自动模块树无需修改 settings。
- 已移除旧模块下忽略的 Gradle `build/` 产物，未删除受版本控制的源码。
- 已通过新模块与 `get-context-remaining` 的 JVM 测试，以及 compact runtime、session composition 的 JVM 编译。
- `kanban/executable/2026-07-26-audit-gradle-dependencies.md` 中的旧路径是其既有审计快照，未改写。

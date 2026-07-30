# Task Tree

- [done] 将 Unified Exec session 状态接入 CLI 工具 history
  - [done] 公开当前 session 的只读观察面
  - [done] 用原始命令改善 `write_stdin` 的语义摘要
  - [done] 用完成状态更新仍可读取输出的 `exec_command` 行
  - [done] 添加渲染与 client 状态测试
  - [done] 运行相关验证

# Details

- 用户已确认 `ManagedProcessSession` 保留启动参数并发布完成状态；本任务只将这些事实用于 CLI 的 unified-exec 历史展示。
- session 注册以 `mutableSessions: MutableStateFlow<Map<…>>` 为唯一事实来源；插入、移除和清理均通过 `StateFlow.update` 的 CAS 循环完成。
- 已通过 `:tool-unified-exec-impl:jvmTest` 与 `:cli-history:jvmTest`；关联 Kotlin 编译亦通过。

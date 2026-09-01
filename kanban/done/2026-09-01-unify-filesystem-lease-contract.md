# Task Tree

- 统一 filesystem lease 契约与实现
  - [done] 确认三种 lease 形状
  - [done] 设计 contract 与 impl 边界
  - [done] 重写 contract 与 impl 模块
  - [done] 迁移全部 lease 调用方
  - [done] 验证 owner 结束时完成释放
  - [done] 运行跨平台测试与编译

# Details

- 三种实现是单文件独占 lease、共享读 lease和独占写 lease。
- 所有构造入口显式接收 owner `CoroutineScope`，并返回同一个 contract。
- 删除当前独立 read-write lease 模块，不兼容保留旧构造入口。
- `contract` 只定义统一 lifecycle interface 和冲突异常。
- `impl` 保存 heartbeat、acquisition guard 和三种构造实现。
- `utils-filesystem-lease-impl:allTests` 通过 JVM、JS 和 Linux native tests，并编译
  macOS ARM64 与 Windows x64 test binaries。
- `app-migration`、`agent-session-filesystem`、`app-viewmodel-application` 和 `app-cli`
  的 Linux x64 编译通过。
- `agent-session-filesystem:jvmTest` 的 `creates an uninitialized root storage` 在修改前
  基线也以 expected 1、actual 0 失败，不归因于本任务。

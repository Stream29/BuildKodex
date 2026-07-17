# Task Tree

- [done] 用协程 scope 表达进程会话所有权
  - [done] 将 `ProcessSession` 和 `ProcessLauncher` API 改为显式 scope 所有权
  - [done] 以 session/process 子 scope 取代手写关闭与资源关闭状态
  - [done] 将 JVM、Node、POSIX、Windows 实现接入 scope 生命周期
  - [done] 让 unified-exec client 以根 scope 管理全部 session
  - [done] 更新真实 I/O 测试并验证所有可用 target

# Details

session scope 是 client scope 的子节点，process I/O 在 session scope 的子 scope 中运行。正常进程完成只结束 process scope，以便消费者读取最终结果；取消 session 或 client scope 则级联终止进程并在 I/O 子协程结束后关闭平台资源。

验证：JVM、Node、Linux x64、macOS ARM64 真实 I/O 测试通过；Linux ARM64 与 Windows 源集编译通过。

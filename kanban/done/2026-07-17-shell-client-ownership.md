# Task Tree

- [done] 将进程启动抽象收敛为 `ShellClient`
  - [done] 用可关闭的 `ShellClient` 取代 `ProcessLauncher` 和全局 launcher
  - [done] 让 `ShellClient` 私有持有 session 的父协程 scope
  - [done] 更新 JVM、Node、POSIX、Windows 实现与调用方
  - [done] 简化 unified-exec 的 session 清理路径
  - [done] 更新真实 I/O 测试并验证可用 target

# Details

每个 `ShellClient` 拥有一个 root scope。其 `close()` 取消该 scope；由它创建的 session 是该 scope 的子节点。

验证：JVM、Node、Linux x64、macOS ARM64 真实 I/O 测试通过；Linux ARM64 与 Windows 源集编译通过。

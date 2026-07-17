# Task Tree

- [done] 以协程 scope 委托实现 `ShellClient`
  - [done] 用 `CoroutineScope` 的内部扩展取代 `ShellClientScope`
  - [done] 让各平台 `ShellClient` 委托其 root scope
  - [done] 直接以 `ShellClient` 作为 session 父 scope
  - [done] 验证所有可用 target

# Details

`ShellClient` 继承 `CoroutineScope`，但不保留额外 scope 属性。`close()` 取消委托 scope，`requireOpen()` 是仅模块内部使用的扩展。

已通过 JVM、Node.js、Linux x64 的测试，编译 Linux ARM64 与 Mingw x64 源集，并在 macOS ARM64 上通过真实进程测试。

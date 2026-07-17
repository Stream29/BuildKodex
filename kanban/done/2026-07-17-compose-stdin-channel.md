# Task Tree

- [done] 将 stdin 通道从 `PlatformProcessSession` 提取为可组合组件
  - [done] 新建同时提供公开发送与内部接收面的 `StdinChannel`
  - [done] 由 `PlatformProcessSession` 组合该组件并保留平台写入职责
  - [done] 为独立通道确认与生命周期行为添加测试
  - [done] 验证各 host target

# Details

`StdinChannel` 公开为 `SendChannel<String>`，内部消费为 `ReceiveChannel<PendingStdin>`。它使用 rendezvous 交接，并在拥有的 scope 或 job 完成时取消内部通道。

内部 `Channel<PendingStdin>` 现在由 `StdinChannel` 在类体中私有创建；接收侧 API 显式转发，调用方不能替换或配置该队列。

验证通过：Linux JVM、JS/Node、Linux x64、macOS ARM64 的 `stdinChannelTest`；Linux arm64 与 Mingw x64 编译通过。

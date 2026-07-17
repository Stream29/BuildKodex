# Task Tree

- 实现 `unified-exec` 的跨平台 PTY 支持
  - [done] 贯通 `tty` 参数到 `ShellClient`
  - [done] 为公共 PTY 行为增加真实 I/O 测试
  - [done] 接入 POSIX Native PTY 后端
  - [done] 接入 JVM PTY 后端
  - [done] 接入 Node.js PTY 后端
  - [done] 接入 Windows ConPTY 后端
  - 在可用平台验证 PTY、交互输入和取消语义

# Details

`tty=true` 当前在工具层明确拒绝。目标是保持 `ProcessSession` 的公开输入、输出和退出码契约不变，仅在平台启动层切换为 PTY 后端。

- Linux Native、JVM 与 Node.js 的真实 PTY、交互输入和关闭生命周期测试已通过。
- `mingwX64` 的 ConPTY CInterop、Kotlin 编译和测试二进制链接已通过；Windows VM 当前关机，尚未实机运行。
- 远程 Mac 当前检出的仓库缺少 `utils-shell-client`，尚不能运行本次 macOS 测试。

# Task Tree

- [done] 验证并修复 Mingw shell 的 Unicode 路径行为
  - [done] 确认 Windows 启动链与路径传递边界
  - [done] 为 Unicode 工作目录、命令和输出补充真实 I/O 回归测试
  - [done] 在 Windows VM 运行 Mingw 测试二进制
  - [done] 修复发现的代码页或转义问题
  - [done] 记录结果并归档任务

# Details

- CP936 Windows VM 已通过 pipe、PTY 和完整 `shell-client` 测试。
- 真实 I/O 测试改用 TestBalloon 的 `RealTime` compartment。
- Windows 命令行转义曾在 `buildString` 中遍历自身缓冲区，带空格参数会无限追加；现改为显式遍历原始字符串。

# Task Tree

- [done] 验证 `kotlinx-io` 在 Mingw 的非 ASCII 文件系统行为
  - [done] 确认共同协程文件系统测试的现有覆盖范围
  - [done] 为 Unicode 路径、内容和目录列举补充真实 I/O 回归测试
  - [done] 在 Windows VM 运行 Mingw 测试二进制
  - [done] 以 Win32 UTF-16 文件系统实现消除代码页依赖
  - [done] 记录结果并归档任务

# Details

- CP936 Windows VM 复现 `kotlinx-io 0.9.0` 的窄字符 API 问题。
- Mingw 改用宽字符 Win32 API；共同测试在 Windows、JVM、Linux Native 和 Node 通过，Linux ARM64 与 macOS ARM64 编译通过。

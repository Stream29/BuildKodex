# Task Tree

- 验证 `kotlinx-io` 在 Mingw 的非 ASCII 文件系统行为
  - [done] 确认共同协程文件系统测试的现有覆盖范围
  - [done] 为 Unicode 路径、内容和目录列举补充真实 I/O 回归测试
  - 在 Windows VM 运行 Mingw 测试二进制
  - 记录结果并归档任务

# Details

验证 `SystemCoroutineFileSystem` 经 `kotlinx-io` 的路径编码是否能在真实 Windows 文件系统上正确保留非 ASCII 目录名、文件名和 UTF-8 内容。

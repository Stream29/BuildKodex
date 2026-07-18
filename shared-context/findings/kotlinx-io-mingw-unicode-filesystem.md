# kotlinx-io Mingw Unicode 文件系统

- `kotlinx-io 0.9.0` 的 Mingw `SystemFileSystem` 使用 `fopen`、`stat`、`MoveFileExA` 等窄字符 API。
- 在 CP936 Windows VM 上，含俄文和日文的目录可以创建，但同一路径的文件打开会失败并报 `Invalid argument`。
- `utils:kotlinx-io-coroutines` 的 Mingw 实现通过 Kotlin/Native 绑定直接调用 UTF-16 Win32 API：`CreateFileW`、`FindFirstFileW`、`MoveFileExW`、`GetFinalPathNameByHandleW` 等。
- 共同回归测试覆盖 Unicode 路径、UTF-8 内容、枚举、移动和解析；已在该 CP936 Windows VM 真实运行通过。

# Task Tree

- [done] 修复 `/home/stream` 目录选择器不显示文件夹
  - [done] 复现并定位目录读取链路
  - [done] 确认前次修复未覆盖的原因
  - [done] 补充针对性回归测试
  - [done] 实施最小修复
  - [done] 运行定向验证

# Details

- 状态：`done`。
- 用户报告前次 home shorthand 修复后，以 `/home/stream` 打开 path picker 仍无法展示文件夹列表。
- 已在 release executable 中稳定复现，界面错误为 `/home/stream/.steampath`。
- `/home/stream/.steampath` 是目标不存在的符号链接。
- `DirectoryPickerBrowser` 已先把待浏览目录解析为绝对路径；`list()` 返回的子路径也为绝对路径，不应再次 `resolve()`。
- 修复路线：新增 POSIX 失效符号链接回归测试，并直接检查 `list()` 返回子路径的 metadata。
- 验证范围：shared path picker 的 Linux/JVM 测试、CLI path picker 的 Linux 测试，以及从 `/home/stream` 启动 release executable 的实际界面。
- 新回归测试在修复前以 `FileNotFoundException` 失败，移除对子路径的重复 `resolve()` 后通过。
- `:app-shared-path-picker:jvmTest`、`:app-shared-path-picker:linuxX64Test`、`:app-cli-path-picker:linuxX64Test` 与 `:app-cli-application:linkReleaseExecutableLinuxX64` 通过。
- 从 `/home/stream` 启动重建后的 release executable，path picker 正常显示子目录并跳过 `.steampath`。

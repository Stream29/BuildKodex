# Task Tree

- [done] 修复 CLI 测试临时目录残留
  - [done] 确认目录创建条件与残留原因
  - [done] 修复临时目录位置和关闭后的清理
  - [done] 添加回归测试并运行目标验证
  - [done] 清理已确认的遗留目录和暂存项

# Details

- Linux Native 在 `TMPDIR` 与 `TMP` 均未设置时，使 `SystemTemporaryDirectory` 回退到测试工作目录 `cli/app`。
- `dispose()` 原先使用不等待后台任务完成的 `close()`，可能在删除后留下文件系统租约的收尾文件。
- 测试根目录现固定为已忽略的 `build/tmp`；`dispose()` 使用 `shutdown()`，且关闭与删除均处于非取消上下文。
- 新增回归测试验证目录位置和清理结果；`linuxX64Test` 的新增测试与原先会残留的空仓库测试均通过。
- `jvmTest` 在无关的 `:Mosaic:mosaic-tty:compileJdk22KotlinJvm` 阶段因未解析的 JDK22 原生绑定而无法开始 CLI 测试。

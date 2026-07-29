# CLI App 测试临时目录清理

- 每次 `TestCodexLiteApplication.open()` 或 `openWithIsolatedCodexHome()` 成功初始化时，都会创建 `codex-lite-cli-test-<random>` 根目录。
- Linux Native 的 `SystemTemporaryDirectory` 在 `TMPDIR` 和 `TMP` 都未设置时会回退为空相对路径，因此目录曾创建在测试工作目录 `cli/app`。
- 旧版 `dispose()` 调用非等待式 `close()`；后台文件系统租约在根目录删除后仍可能完成释放或写入，造成残留。
- 测试辅助类现在固定使用已忽略的 `build/tmp`，并在非取消上下文中先调用等待完成的 `shutdown()` 再删除根目录。
- Linux Native 回归测试覆盖目录位置和 `dispose()` 后目录不存在；原先会残留的空仓库测试也已通过。

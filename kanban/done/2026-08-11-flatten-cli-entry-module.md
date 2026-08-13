# Task Tree

- [done] 扁平化 CLI 入口模块
  - [done] 将 executable 模块内容上移至 `app/cli`
  - [done] 清理旧 `app/cli/executable` 目录
  - [done] 更新入口路径文档与本地命令
  - [done] 验证 release executable 与真实终端启动

# Details

- 用户明确决定不保留 `cli/executable` 层级，直接以 `app/cli` 作为 CLI 入口模块。
- 领域 view 继续保留在 `app/view`，本任务不改变其边界。
- Gradle 入口模块现为 `:app-cli`，旧 `app/cli/executable` 目录已清理。
- JDK 26 下通过 `:app-cli:compileKotlinLinuxX64` 与 `:app-cli:linkReleaseExecutableLinuxX64`；入口模块没有测试源，`linuxX64Test` 按预期跳过。
- release executable 位于 `app/cli/build/bin/linuxX64/releaseExecutable/kodex-cli.kexe`，本地 `kodex-cli` 链接已更新至该路径。
- 独立 PTY 中启动后正常进入 alternate screen，Ctrl-C 正常退出并恢复终端标题。

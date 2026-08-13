# Task Tree

- [done] 将 CLI executable 迁移到 `app/cli/executable`
  - [done] 核对模块发现规则与现有引用
  - [done] 迁移模块并清理旧目录
  - [done] 更新构建命令与本地入口
  - [done] 构建、测试并真实启动验证

# Details

- 用户明确将 CLI executable 模块命名为 `app/cli/executable`。
- 本任务只迁移现有 executable 模块，不推进其余 frontend、ViewModel 或 Koin 改造。
- IntelliJ IDEA 正在打开 BuildKodex 项目。
- `:app-cli-executable:linuxX64Test` 与 `:app-cli-executable:linkReleaseExecutableLinuxX64` 使用 JDK 26 通过。
- `~/.local/bin/kodex-cli` 已指向 `app-cli-executable.kexe`；独立 PTY 启动、标题更新与 Ctrl-C 退出通过。

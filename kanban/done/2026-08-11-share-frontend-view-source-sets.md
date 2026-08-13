# Task Tree

- [done] 以 KMP renderer source sets 统一前端 view
  - [done] 审计当前 CLI view、components、executable 与 Gradle 重复配置
  - [done] 建立共享 view 与 CLI executable convention plugins
  - [done] 将 CLI view 和 components 迁入 `app/view`
  - [done] 将 application view 与测试迁出 executable
  - [done] 收敛 executable 至入口和 CLI 生命周期
  - [done] 更新项目依赖、操作性文档与本地命令
  - [done] 验证共享 view、release executable 与真实终端启动

# Details

- 用户明确要求持续实施至完成后再验收。
- `cli` 与未来的 `desktop` 只表示应用入口；领域 view 使用 KMP `mosaicMain` 与未来的 `desktopMain` 表达 renderer 差异。
- 本任务不推进 Session v2、Koin 接入或业务状态重构。
- IntelliJ IDEA 正在打开 BuildKodex 项目。
- 已增加 `kodex.kmp-view` 与 `kodex.kmp-cli-executable` conventions，并集中声明 Mosaic source-set hierarchy。
- `app/cli` 现仅保留 `executable`；共享组件、领域 view 与 application view 均位于 `app/view`。
- JDK 26 下通过所有 `app-view-*` Linux x64 测试、CLI Linux x64 测试及 release executable 链接。
- 独立 PTY 中启动 `kodex-cli` 后正常进入 alternate screen，Ctrl-C 正常退出并恢复终端标题。
- IDEA 2026.2 的 Gradle sync 在导入复合 KMP 项目时耗尽 IDE heap；因此 IDE 模型暂存旧 module，Gradle 命令行验证作为本任务的编译依据。

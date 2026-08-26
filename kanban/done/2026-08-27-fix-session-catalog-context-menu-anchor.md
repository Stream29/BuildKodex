# Task Tree

- [done] 修复 Sessions 归档右键菜单
  - [done] 复核既有归档交互约定
  - [done] 绑定 Session 行与菜单锚点
  - [done] 覆盖右键及键盘菜单入口
  - [done] 运行定向验证并构建二进制

# Details

- 状态：`done`。
- 用户要求接通既有 Sessions 归档右键菜单，不恢复右键直接打开 Delete 弹窗的旧行为。
- `2026-08-25-add-session-archiving` 已确定未归档项显示 Archive、Delete，已归档项显示 Unarchive、Delete；Delete 只在选择菜单项后进入确认 Dialog。
- 当前 Session 行创建了 `TuiPopupAnchor`，但未通过 `Modifier.tuiPopupAnchor` 绑定到按钮，导致 `TuiContextMenu` 因锚点未放置而不渲染。
- 将 catalog 行和对应 ContextMenu 内容提取为同模块 `internal` Composable，以定向验证生产接线；不引入测试替身。
- 覆盖 secondary pointer、`Shift+F10` 和 Menu/Application 键，并验证 Archive、Unarchive、Delete 仍从 ContextMenu 路由。
- 修复范围仅包含该 frontend 接线及回归覆盖，不推进 Session Catalog Fork 规划。
- Session catalog 行现通过 `Modifier.tuiPopupAnchor` 报告边界；Archive/Unarchive/Delete 菜单内容已提取为可定向验证的模块内 Composable。
- 新增生产组件交互回归，覆盖 secondary pointer、`Shift+F10`、Menu/Application 键及 Archive、Unarchive、Delete 路由；定向 JVM 测试的 3 个用例通过。
- `:app-view-application:jvmTest` 与 `:app-view-application:linuxX64Test` 均通过，各执行 59 个测试且无失败。
- IntelliJ 检查无问题，变更文件构建通过；`git diff --check` 通过。
- `:app-cli:linkReleaseExecutableLinuxX64 --no-configuration-cache` 通过。
- Linux x64 release executable：`Kodex/app/cli/build/bin/linuxX64/releaseExecutable/kodex-cli.kexe`，68,896,648 bytes。
- SHA-256：`4eb36e0daa03970c8cac9f13b0351ae0269efba615cbb3f894771cf85581295a`。

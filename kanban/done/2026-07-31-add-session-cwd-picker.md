# Task Tree

- [done] 收敛 session settings 操作，并为 cwd 提供独立路径选择器
  - [done] 盘点现有 Settings 弹窗、cwd 持久化模型与 TUI 组件边界
  - [done] 新建独立 `cli/path-picker` 模块，提供仅目录的浏览与选择弹窗
  - [done] 将弹窗接入 Session 设置页并持久化选定 cwd
  - [done] 将 `Apply` / `Cancel` 收敛为单一 `Close`
  - [done] 为选择器与 settings 集成补充验证并运行相关检查

# Details

- 选择器只显示目录，可进入子目录、返回父目录，并显式确认当前目录。
- `:cli-path-picker:linuxX64Test` 与 `:cli-app:compileKotlinLinuxX64` 已通过。
- JVM 验证受既有 Mosaic `jvmJdk22` 的 `Libmosaic` 未解析问题阻断。

# Task Tree

- [done] 实现 JVM Desktop 应用
  - [done] 确认 Compose Desktop、MD3E 与现有前端边界
  - [done] 建立 Desktop 构建、source set 与入口
  - [done] 实现 application shell 与主题
  - [done] 实现 Session、Agent、history 与 composer 主流程
  - [done] 实现 settings、picker 与其他 popup
  - [done] 补充 Desktop 测试与运行验证
  - [done] 修正 Desktop 产品还原
    - [done] 对照 TUI 明确布局与主题语义
    - [done] 按 TUI 重做 Desktop 应用骨架
    - [done] 按 TUI 还原核心界面与交互
    - [done] 按 TUI 还原 popup 与 settings
    - [done] 验证明暗主题及 Desktop 回归

# Details

- 用户要求基于现有成熟的 contract/ViewModel 层制作一版 JVM Desktop 应用。
- Desktop 使用 Material 3 Expressive；领域行为保持在现有共享层，Desktop 只实现 renderer 与 host。
- 目标是可运行、覆盖现有主要交互的第一版，不改动 CLI 产品行为。
- 采用独立 `app/desktop` JVM application，避免默认 JVM target 继续继承 `mosaicMain` 时把 Mosaic renderer 带入 Desktop。
- Desktop view 按现有领域模块放在 `desktopMain`；共享 contract、ViewModel 与展示逻辑不复制。
- 使用 Compose Multiplatform `1.11.1` 与其对应的 Material3 `1.11.0-alpha07`，由 `MaterialExpressiveTheme` 提供 MD3E theme。
- 验证包括 Desktop 编译、ViewModel JVM tests、Desktop UI/state tests、应用启动 smoke 与 distribution 构建。
- `:app-desktop:test` 与七个共享 ViewModel `jvmTest` 任务通过。
- Desktop 进程在 20 秒启动冒烟中持续运行，无启动异常；Linux DEB 构建成功。
- 用户反馈首版错误地重新设计了布局，且明暗主题适配不符合现有产品。
- 修正版必须以现有 TUI renderer 为唯一产品基准，仅将其映射为 MD3E 控件。
- 修正版还原了 TUI 的 tab、Agent sidebar、history、request input、composer、status bar、settings 与 picker 层级。
- Desktop 使用完整明暗色板并跟随系统主题；应用根 Surface 明确绘制主题背景。
- Desktop UI 测试覆盖显式明暗主题与共享选择控件；未发现主题文件之外的硬编码 Desktop 颜色。
- 最终验证通过 `:app-desktop:test`、七个共享 ViewModel `jvmTest`、15 秒启动冒烟与 `:app-desktop:packageDeb`。

# Task Tree

- [done] 修正 Desktop 弹窗与主题适配
  - [done] 核对 Compose 模态层和主题检测行为
  - [done] 将大型弹窗迁入 Compose 模态层
  - [done] 修正系统明暗主题检测与监听
  - [done] 补充测试并运行 Desktop 回归

# Details

- 用户要求悬浮弹窗使用 Compose 自带模态设计，不再使用真实平台窗口。
- 当前四个大型弹窗直接使用 `DialogWindow`；Compose Material 3 的 `BasicAlertDialog` 使用同窗口 `ComposeSceneLayer`。
- 当前系统为 GNOME `prefer-dark`，但 Skiko `currentSystemTheme` 返回 `UNKNOWN`，导致 `isSystemInDarkTheme()`错误回落到亮色。
- 修正应保留 TUI 的内容层级，仅替换弹窗承载方式，并可靠响应系统主题变化。
- 使用 `BasicAlertDialog` 承载 Session catalog、Settings、Directory picker 和 OpenAI login。
- Desktop host 优先采用 Skiko 主题结果；结果未知时按操作系统设置回退，并监听后续变化。
- 以纯解析测试、主题 UI 测试、Desktop tests、启动冒烟和静态扫描验证结果；修改路线已确定，可执行。
- 四个大型弹窗已统一使用 `DesktopModal`；Desktop 源码中不再引用 `DialogWindow`。
- `:app-desktop:test` 的 10 个测试通过，包含 640dp 自定义模态宽度和系统主题回退测试。
- 15 秒启动冒烟持续运行至超时，`packageDeb` 成功生成安装包。

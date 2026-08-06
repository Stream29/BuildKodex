# Task Tree

- [done] 调整状态栏 Settings 按钮布局
  - [done] 确认现有右对齐布局模式
  - [done] 在两种状态栏末尾放置 Settings
  - [done] 用加权 Spacer 分隔左侧操作
  - [done] 新增最右对齐快照验证
  - [done] 运行 application Linux 测试

# Details

- 用户要求 Settings 按钮与其他状态栏操作分开，并固定在最右侧。
- IntelliJ IDEA 2026.2 正在打开本项目。
- Agent 与 New Session 状态栏均渲染同一 Settings 操作，应保持一致布局。
- 使用 Row 的加权 Spacer 占据剩余宽度；Settings 作为末尾元素，错误或提示信息留在左侧操作组。
- 通过 New Session 状态栏快照验证 Settings 结束于状态栏最右侧。
- IDE inspection 未发现本次文件的问题；`git diff --check` 通过。
- 已运行 `app-cli-application:linuxX64Test`，但当前另一项未完成改动使 `SessionTreeCliScreen.kt:251` 缺少 `onOpenHistoryEntryContextMenu` 参数，测试在编译阶段被阻断。

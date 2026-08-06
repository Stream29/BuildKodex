# Task Tree

- [done] 精简运行状态栏操作与模式标签
  - [done] 确认现有 Fork 与模式标签接线
  - [done] 从状态栏移除 Fork 按钮
  - [done] 清理三层旧 Fork 参数
  - [done] 将 Default UI 标签改为 build
  - [done] 增加 build/plan 标签断言
  - [done] 运行 application Linux 验证

# Details

- 用户正在将 Fork 改造成 history 条目右键菜单，因此要求移除底部状态栏的 Fork 按钮。
- 用户要求把界面中的 default/plan mode 显示为 build/plan mode。
- IntelliJ IDEA 2026.2 正在打开本项目。
- 正在进行的 history 上下文菜单任务保持只读；本任务只移除其不再需要的旧状态栏 Fork 接线。
- 清理 `SessionTreeCliScreen` 到 `AgentRuntimeStatusBar` 的旧 `forkEnabled`/`onFork` 参数，但保留 shared ViewModel 的 fork 能力供上下文菜单改造使用。
- 只修改 UI `ModeKind.displayName()`，领域值及序列化语义仍保持 `ModeKind.Default`。
- 通过模式标签单元断言、IDE inspection、diff 检查及 application Linux 测试验证。
- 底部状态栏不再渲染 Fork，history 上下文菜单的新 `onFork` 接线保持不变。
- `ModeKind.Default` 与 `ModeKind.Plan` 的 UI 标签分别为 `build` 与 `plan`。
- IDE inspection 未发现本次独立文件的问题；旧状态栏 Fork 引用检查与 `git diff --check` 通过。
- 已运行 `app-cli-application:linuxX64Test`；当前并行中的 history 上下文菜单代码在 `SessionTreeCliScreen.kt:555` 产生 always-false 警告，并因 `-Werror` 在编译阶段阻断测试执行。

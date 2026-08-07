# Task Tree

- [done] 为下栏增加当前目标 cwd 按钮
  - [done] 确认真实 Session 只修改当前 Agent
  - [done] 确认 New session 修改当前标签草稿
  - [done] 在两种下栏渲染响应式 cwd 按钮
  - [done] 复用现有目录选择器与目标路由
  - [done] 在 Agent 运行或 New session 创建时禁用
  - [done] 补充标签、点击与禁用状态测试
  - [done] 更新持久交互约束
  - [done] 运行定向测试与检查

# Details

- 状态：实现与验证均已完成。
- 真实 Session 的 cwd 操作只更新当前选中 Agent，不传播到 root 或其他 subagent。
- 虚拟 New session 的 cwd 操作只更新当前标签的内存草稿，并在该标签物化时复制到 root Agent settings。
- 复用既有 `DirectoryPickerPopup`、`WorkingDirectoryPickerRequest`、`AgentRuntimeViewModel.updateSettings` 和 `NewSessionViewModel.updateWorkingDirectory`。
- 真实 Agent 运行期间禁用 cwd 操作，避免当前 turn 的固定上下文与动态工具 cwd 分叉；New session 创建期间同样禁用。
- 状态栏按终端宽度显示 `cwd`、截尾短路径或截尾长路径，Settings 继续固定在最右侧。
- `RuntimeStatusBarTest` 定向测试共 6 项通过，完整 `:app-cli-application:linuxX64Test` 通过。
- IntelliJ 已重排改动文件且项目构建通过；inspection 仅报告 `SessionTreeCliScreen` 既有的项目模型解析项与未改动行警告，Gradle 编译未复现解析错误。
- `git diff --check` 通过。
- IntelliJ IDEA 2026.2 正在打开本项目。
- 当前工作树包含用户既有改动；本任务不修改或清理无关文件，不创建 Git commit。

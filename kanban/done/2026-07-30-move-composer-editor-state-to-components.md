# Task Tree

- [done] 将主输入框的通用编辑状态进一步收拢到 `cli:components`
  - [done] 提取可由组件持有的文本编辑状态与受控编辑入口
  - [done] 让 `TextInput` 使用该状态，并保留 app 自定义按键预处理扩展点
  - [done] 简化 app Composer，使其只绑定会话提交与配置快捷键
  - [done] 为未来 undo/redo 留出状态边界，并补充回归测试
  - [done] 验证 JVM 与 Linux Native 组件/app 测试

# Details

- 用户于 2026-07-30 要求把主输入框更复杂的通用逻辑迁移到 `cli:components`，以便未来支持 Ctrl+Z。
- 本任务只建立可扩展的编辑状态边界，不实现 Ctrl+Z/Ctrl+Y 的产品行为。
- `NewLineKey`、`SubmitKey`、会话提交和消息生命周期继续属于 app。
- `:cli-components:jvmTest`、`:cli-app:jvmTest`、`:cli-components:linuxX64Test` 与 `:cli-app:linuxX64Test` 已通过。

# Task Tree

- 补齐 CLI 的 `request_user_input` 交互
  - [done] 确认 pending tool call、完成事件与当前 Agent Runtime 屏幕的衔接点
  - [done] 在 history 与 composer 之间渲染当前 Agent 的交互式问题表单
  - [done] 将选项或自由文本答案编码为工具响应并完成调用
  - [done] 提交后继续该 Agent runtime
  - [done] 补充 UI 与状态投影测试并运行相关验证

# Details

- 仅在当前 `ToolPending` 恰好包含一个 `request_user_input` 调用时显示表单。
- 表单和答案草稿按 Agent Runtime ViewModel 隔离；不改变其他 pending tool 的处理方式。
- 已通过 `:cli-agent:jvmTest`、`:cli-components:jvmTest` 和 `:cli-app:jvmTest` 的定向验证；Mosaic JDK 22 原生绑定任务按现有项目限制排除。

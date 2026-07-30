# Task Tree

- [done] 在 Global Settings 配置自动 session title
  - [done] 确认现有标题设置、持久化和弹窗 UI 的缺口
  - [done] 扩展标题设置以保存推理等级
  - [done] 将模型、开关和推理等级接入 Global Settings popup
  - [done] 将推理等级传递到 title Responses 请求
  - [done] 添加针对设置持久化、请求参数和 UI 的验证

# Details

- 用户请求在 Global Settings 中配置自动 session title，并可调整模型推理等级。
- 现有 `SessionTitleSettings` 已有 `enabled` 与 `model`，但 Global Settings popup 未渲染或更新它们。
- `reasoningEffort` 默认仍为 `Low`，保持变更前 title Responses 请求的行为。
- 验证：`cli-settings-filesystem`、`cli-session-title`、`cli-agent`、`cli-session` 和 `cli-app` 的 JVM 测试均通过。

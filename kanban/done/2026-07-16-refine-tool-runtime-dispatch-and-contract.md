# Task Tree

- [done] 让 tool runtime 只处理自身负责的 pending tool call
  - [done] 将工具 spec 注册从 runtime 装饰器职责中移除
  - [done] 让每个 runtime 仅匹配并完成自己的调用，保留其他 pending call
  - [done] 移除 image generation runtime 的不安全 `Tool` 重载
- [done] 简化 tool contract
  - [done] 以 `ResponseItem.ToolCall` 和 `ResponseItem.ToolCallOutput` 替代 `ToolCallPayload`、`ToolCallResult`
  - [done] 删除无意义的 `ToolCallConversion.kt`
  - [done] 更新工具实现、runtime 与测试
- [done] 验证跨平台 tool runtime 组合行为

# Details

Runtime 不注册工具，只从 `KodexAgentStateValue.ToolPending` 中识别自身负责的调用并处理。工具调用的输入和输出直接采用 OpenAI 模型，避免无信息增益的包装与转换。

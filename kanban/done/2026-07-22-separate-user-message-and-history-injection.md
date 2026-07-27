# Task Tree

- [done] 收窄用户消息与持久化上下文注入边界
  - [done] 评审 `followingContext` 的设计动机与状态机影响
  - [done] 确定用户消息与持久化上下文使用独立原子操作
  - [done] 恢复 `appendUserMessage(content)` 单参数 contract 与实现
  - [done] 调整 `SkillSelectionRuntime` 先追加用户消息，再通过 `injectHistory` 注入已选 skill 正文
  - [done] 更新相关测试与 `agent-state-and-runtime` 决策记录
  - [done] 运行相关格式化、测试和检查

# Details

- 状态：已完成实现与跨平台验证。
- 保持用户消息到 skill context 的持久化顺序。
- 在写入用户消息前完成 skill generation 捕获、选择和正文加载。
- 接受两次原子操作之间失败时仅保留合法用户消息前缀。
- 不在 `appendUserMessage` 上暴露 `ResponseItem.HistoryItem`，不新增 turn batch 模型。

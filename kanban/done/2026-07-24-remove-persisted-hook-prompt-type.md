# Task Tree

- [done] 移除持久化的 Hook Prompt 特殊类型
  - [done] 确认 Rust wire、存储与 UI 投影边界
  - [done] 删除 `ResponseItem.HookPrompt` 模型及请求特殊投影
  - [done] 让 Turn Hook 直接持久化真实的 user Message
  - [done] 删除状态机与 UI 的 Hook 来源特判
  - [done] 更新相关测试并完成验证

# Details

Hook continuation 应以实际发送给 Responses API 的 user Message 作为持久化事实。UI 不额外推断或展示 Hook 来源。

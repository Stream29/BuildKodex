# Task Tree

- [done] 实现 `request_user_input` ToolSpec 与 hosted `web_search`
  - [done] 定义 `request_user_input` 的参数 DTO、schema 和静态 spec
  - [done] 补齐 hosted `web_search` 的 Rust 对齐 wire 字段
  - [done] 验证 hosted `web_search` 直接进入 settings、请求和持久化历史
  - [done] 记录工具分类决策并运行相关跨平台测试

# Details

`request_user_input` 暂不实现 handler；它需要后续专用 AgentRuntime 承接宿主交互。hosted `web_search` 由 Responses 服务执行，不进入 `CodexToolRuntime`。

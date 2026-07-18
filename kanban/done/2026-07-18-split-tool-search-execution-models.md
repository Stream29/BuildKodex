# Task Tree

- [done] 按 execution 拆分 tool-search 响应模型
  - [done] 将 client tool-search call/output 纳入 `ToolCall` / `ToolCallOutput`
  - [done] 将 server tool-search call/output 保持为纯 `HistoryItem`
  - [done] 在 `ResponseItemSerializer` 按 `type` 与 `execution` 做双判别序列化
  - [done] 收敛 AgentState 与 ToolRuntime 的统一 tool-call 路径
  - [done] 增加 client/server wire 回归测试并完成跨平台验证

# Details

`execution` 是 tool-search 的第二层 wire 判别值。公开模型不再以可空 `callId` 表达 client/server 的互斥语义。

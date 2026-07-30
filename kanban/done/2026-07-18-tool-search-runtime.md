# Task Tree

- [done] 实现 deferred tool search 的运行时链路
  - [done] 建立 direct/deferred 工具可见性组合配置
  - [done] 扩展 AgentState 以跟踪和完成 client `tool_search_call`
  - [done] 在 KodexToolRuntime 内执行 tool search 特殊调度分支
  - [done] 对齐请求 history 与 context compaction 的 tool search 语义
  - [done] 补齐跨平台状态、运行时和端到端回归测试
  - [done] 更新相关 checklist 决策

# Details

`tool_search` 保持为 agent-loop 原语，不伪装为普通 OpenAI `ToolCall`，也不新增独立 Runtime 装饰层。所有候选工具的 handler 在 Runtime 创建时已可执行；direct/deferred 只影响初始请求的模型可见 schema。搜索结果通过 `ToolSearchOutput` 写入 history，而不修改 `KodexAgentSettings.tools`。

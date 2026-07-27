# Task Tree

- [done] 实现MCP Service
  - [done] 将MCP设置与服务器配置纳入contract
  - [done] 实现监听设置并维护活动client的MCP Service
  - [done] 将MCP目录投影为可执行Tool列表
  - [done] 验证最小client重载、refresh与生命周期

# Details

- `McpService`只对外发布当前`Tool`列表并提供`refresh()`。
- 实现持有MCP连接所有权，设置变化只重建新增、移除或配置发生变化的client。
- 每次连接拓扑或工具目录变化后一次性发布完整工具列表。
- Streamable HTTP与stdio均通过真实IO测试。
- `mcp:impl`与`mcp:stdio`的全目标测试聚合通过。

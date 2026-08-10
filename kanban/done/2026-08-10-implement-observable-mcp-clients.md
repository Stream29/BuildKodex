# Task Tree

- [done] 实现可观测MCP客户端并迁移下游
  - [done] 将`McpService.tools`替换为按名称发布的client map
  - [done] 用稳定owner实现连接状态、调用、refresh和reconnect
  - [done] 保留断线catalog并返回Agent可见的不可用结果
  - [done] 迁移AgentState、Runtime和测试工具投影
  - [done] 在Global Settings展示状态并提供失败重连
  - [done] 覆盖HTTP、stdio、UI和下游验证

# Details

- 用户已确认`McpClient`公开固定catalog、连接状态和`reconnect()`。
- 断线不删除已发布工具；工具通过owner client调用，非healthy时返回可见的失败结果。
- 显式refresh、成功reconnect或设置更新才发布刷新后的catalog。
- `McpService.clients`包含所有enabled配置，包括尚未连接成功的client；disabled或删除的配置不发布client。
- catalog generation作为不可变`McpClient`视图发布；同一配置的视图共享稳定连接owner。
- 手动refresh复用healthy连接；reconnect替换连接后执行同一catalog加载。
- 连接阶段使用`Connecting`，运行失败使用结构化`Failed(reason)`，详细异常仅写日志。
- HTTP、stdio、Agent下游JVM测试与Linux X64 Mosaic状态面板测试通过。

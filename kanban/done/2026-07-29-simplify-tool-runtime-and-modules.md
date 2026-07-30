# Task Tree

- [done] 简化 Tool Runtime 与工具模块结构
  - [done] 盘点 Tool Runtime、Tool Search 与具体工具的依赖
  - [done] 将 Tool Runtime 改为消费外部工具列表
  - [done] 将具体工具模块统一迁移到 `tool/*`
  - [done] 在 AgentSession composition 中构造并管理工具
  - [done] 更新调用方、测试与工具模块规则
  - [done] 运行受影响构建和测试

# Details

- `KodexToolRuntime`只负责工具路由、hook、调用与结果提交，不持有工具资源。
- 固定工具、MCP动态工具和Tool Search状态由外层composition提供。
- AgentSession composition负责构造固定工具、读取最新Agent设置并按AgentState生命周期释放资源。
- 不再区分`tool:impl:*`与`tool:spec:*`。
- 受影响模块的Linux编译、runtime、AgentState工具投影、工具模块和两种Session后端测试已通过。
- `tool-unified-exec:linuxX64Test`仍有既有cwd用例失败；迁移前后的实现、测试和构建脚本逐字一致，本任务未改变该行为。

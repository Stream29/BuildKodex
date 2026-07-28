# Task Tree

- [done] 工具动态配置改用provider
  - [done] 盘点工具对cwd、model与settings的动态读取
  - [done] 将工具客户端的StateFlow参数改为按调用取值的provider
  - [done] 简化Agent工具构建并移除withLatestAgentSettings
  - [done] 更新测试并运行受影响验证

# Details

- provider在每次工具操作开始时读取一次最新值。
- 工具只依赖自己需要的字段，不直接依赖完整AgentSettings。
- Agent runtime、apply-patch和view-image测试通过；unified-exec完整测试仍暴露既有的快速进程stdout drain竞态。

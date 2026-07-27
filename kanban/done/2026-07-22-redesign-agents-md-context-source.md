# Task Tree

- [done] 重新设计AGENTS.md上下文来源
  - [done] 盘点当前AGENTS.md发现、缓存和前缀投影路径
  - [done] 调研Rust的AGENTS.md加载、environment作用域、失效与resume行为
  - [done] 区分全局instructions与项目AGENTS.md的所有权和生命周期
  - [done] 定义cwd或environment变化时的重新发现语义
  - [done] 定义用户或agent修改同路径文件时的生效边界
  - [done] 定义错误、warnings、provenance和快照一致性
  - [done] 提出独立AgentsMdService方案
  - [done] 调整实现与测试

# Details

- `AgentContextPrefixProvider`只消费已加载的AGENTS.md快照，不负责文件系统遍历与缓存失效。
- Rust当前只按ready environment selection缓存AGENTS.md，同路径内容修改不会自动生效，且上游测试已记录cwd变化未刷新的缺口。
- 全局instructions由Codex home下的`AGENTS.override.md`或`AGENTS.md`提供；项目instructions按每个environment从项目根到cwd发现，两者分别保留来源。
- `AgentsMdService.refresh(environmentGeneration)`重新发现并读取完整快照。聚合provider在新用户轮次开始前刷新；同路径修改和environment变化从下一轮生效，tool continuation继续使用该轮冻结快照。
- warning结构化记录读取失败、无效UTF-8和按环境独立计算的字节预算截断；可用内容仍正常发布。
- generation只在环境代或有效快照变化时递增。
- 已通过JVM、Node.js和Linux Native真实文件系统测试，并验证AgentState、runtime与CLI公共源码编译。

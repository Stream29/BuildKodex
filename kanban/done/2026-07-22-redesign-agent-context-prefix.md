# Task Tree

- [done] 重新设计AgentContext上下文来源边界
  - [done] 盘点当前AgentContextPrefixProvider及其调用路径
  - [done] [environment上下文来源](../done/2026-07-22-redesign-environment-context-source.md)
  - [done] [AGENTS.md上下文来源](../done/2026-07-22-redesign-agents-md-context-source.md)
  - [done] [skills上下文来源](../done/2026-07-22-redesign-skills-context-source.md)
  - [done] [dynamic tools上下文来源](../done/2026-07-22-redesign-dynamic-tool-context-source.md)
  - [done] 汇总各类来源的snapshot、generation和invalidation语义
  - [done] 确定AgentContextPrefixProvider的聚合与交付边界
  - [done] 调整实现与跨平台测试

# Details

- 保留`AgentContextPrefixProvider`作为AgentState的结构化前缀来源边界。
- 父任务负责三类prefix来源的聚合、请求级读取与最终交付；普通dynamic tools由ToolRuntimePlan独立管理。
- environment generation先被捕获；AGENTS.md与skills都基于该generation刷新；文件系统provider最后原子发布内部聚合快照。
- 不公开额外的AgentContextSnapshot模型。AgentState在请求投影时通过provider读取当前结构化prefix，且不把它写入history。
- 新用户轮次显式刷新；同一轮的tool continuation不刷新。源变化从下一轮生效，已选skill正文则随用户消息持久化。
- 普通dynamic tools不进入prefix；请求定义、tool search索引与handler路由由同一次`ToolRuntimePlan`快照产生。
- environment、AGENTS.md、skills、prefix render与dynamic tool generation的相关测试已通过JVM、Node.js和Linux Native验证。

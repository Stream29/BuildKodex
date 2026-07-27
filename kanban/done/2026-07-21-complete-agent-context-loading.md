# Task Tree

- [done] 补齐AGENTS.md与skills加载
  - [done] 确认现有provider、render和AgentState注入边界
  - [done] 实现AGENTS.md层级发现和项目边界处理
  - [done] 保留每段AGENTS.md内容的原始来源
  - [done] 实现skills目录和元数据发现
  - [done] 实现skill正文与依赖资源的按需加载
  - [done] 将真实provider接入AgentState和CLI组装路径
  - [done] 覆盖层级覆盖、空结果和按需加载测试

# Details

- `AgentContextProvider`只提供结构化原始信息。
- render层负责将结构化信息转换为提示词。
- AgentState负责注入时机和history投影。
- AGENTS.md不预先合并；加载时保留来源，渲染时再按层级组合。
- skills先发现轻量元数据，模型实际选择后再加载正文和资源。
- Plugins暂不纳入本任务。

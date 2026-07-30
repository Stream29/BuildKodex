# Task Tree

- [done] 重新设计dynamic tools上下文来源
  - [done] 盘点当前内建、MCP、Apps与tool search的tool definitions与模型可见提示词路径
  - [done] 调研Rust的MCP、Apps、plugins与dynamic tools快照与失效行为
  - [done] 区分AgentSettings声明的tools、runtime实际可执行的tools与prefix中的能力描述
  - [done] 定义tool generation及其与request tool definitions的原子性
  - [done] 定义MCP catalog与tool search结果变化时的失效语义
  - [done] 与[Tool Runtime与AgentSettings边界](../done/2026-07-22-redesign-tool-runtime-settings-source.md)协调单一事实来源
  - [done] 评估并否决重复的DynamicToolContextService
  - [done] 调整实现与跨平台测试

# Details

- 普通工具的spec、schema与description已经通过Responses请求定义对模型可见，不重复进入prefix。
- `KodexAgentSettings.tools`是请求工具定义的持久化真源；`ToolRuntimePlan`是一次`resume()`内请求定义、tool search索引和handler路由的共同generation。
- MCP server instructions已经投影为namespace/source description；MCP catalog每次`resume()`取一个快照，当前续跑固定使用该快照，变化从下一次`resume()`生效。
- Apps与plugin提示词有独立语义，保留给对应任务，不并入普通动态工具上下文。
- JVM、Node.js与Linux Native测试覆盖了动态generation在请求、搜索结果和handler间的一致性。

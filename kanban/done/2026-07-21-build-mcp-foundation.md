# Task Tree

- [done] 建立全局MCP基础设施
  - [done] 将MCP服务器配置加入`KodexGlobalSettings`
  - [done] 直接复用Kotlin MCP SDK的客户端、协议模型和transport
  - [done] 建立应用级连接管理与生命周期
  - [done] 实现Streamable HTTP连接；stdio后置
  - [done] 实现工具目录、工具调用、进度和取消
  - [done] 实现resources list/read
  - [done] 建模外部鉴权注入和结构化错误
  - [done] 将MCP工具投影为现有`ToolSpec`
  - [done] 将MCP来源接入tool search
  - [done] 以完整generation发布动态工具目录
  - [done] 为每次`resume()`固定目录generation
  - [done] 覆盖协议投影、目录更新和真实IO集成测试

# Details

- MCP配置属于全局设置，对所有session生效，不进入`KodexAgentSettings`。
- MCP连接与动态目录由应用级对象统一持有；每个session runtime只消费该共享对象。
- 全局配置变化从各session的下一次请求开始生效；已开始的`resume()`继续使用其固定generation。
- MCP工具是请求时的全局工具投影，不把生成的`ToolSpec`副本写入session storage。
- 每次请求的有效工具由session本地工具与当前MCP目录合成；具体直接暴露或延迟加载仍按模型能力投影。
- 工具目录按generation整体替换，确保模型看到的spec与随后执行调用的handler来自同一快照。
- Kotlin MCP SDK已经提供协议模型、客户端和transport，不再自建`mcp:models`、`mcp:client-contract`和通用MCP客户端。
- MCP SDK当前会丢失工具输入schema的部分顶层JSON Schema关键字；接入前需要保留原始schema的最小修复，优先形成可上游提交的补丁。
- 通用MCP层不承担Codex Apps目录和策略；Apps在其上提供全局MCP服务器来源。
- 原始捕获同时保留输入和输出schema；输出schema按Rust结构投影为完整`CallToolResult`。
- MCP名称清洗、冲突消解、64字符限制和SHA-1后缀与Rust语义对齐。
- JVM真实Streamable HTTP测试覆盖连接、鉴权、目录、调用、进度、取消、resources和generation更新。
- common协议与投影测试已通过JVM、Node.js、Linux x64和macOS ARM64。

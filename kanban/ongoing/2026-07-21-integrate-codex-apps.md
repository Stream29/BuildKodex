# Task Tree

- 接入Codex Apps
  - [done] 调查上游Apps目录、hosted MCP和工具调用路径
  - 注册保留名为`codex_apps`的内置MCP server
  - 对齐Apps工具namespace、名称和connector元数据
  - 为`OpenAiClient`增加类型化App目录请求
  - 建立应用级`CodexAppsCatalog`
  - 缓存目录元数据并按账号和workspace隔离
  - 合并目录元数据、MCP可访问状态和全局启用状态
  - 将App和tool policy加入全局设置
  - 将可用Apps接入tool search和上下文提示词
  - 实现MCP工具审批和鉴权elicitation
  - 实现`openai/fileParams`投影、上传和参数改写
  - 将App目录和交互接入CLI
  - 通过真实ChatGPT端点验证完整链路

# Details

- 本任务保持ongoing，暂不实施。
- 本任务依赖通用MCP基础设施。
- App目录接口提供展示、发现和安装链接；可访问状态从`codex_apps` MCP工具的connector元数据推导。
- `CodexAppsCatalog`是应用级对象，合并目录元数据、MCP可访问状态和全局policy。
- Apps配置属于`CodexGlobalSettings`，对所有session生效，不进入`CodexAgentSettings`。
- `codex_apps`是host注入的保留MCP server，不作为普通用户MCP配置。
- Apps工具继续通过通用MCP runtime执行，不建立`agent-runtime:apps`。
- MCP工具generation是工具调用的唯一一致性边界，不另建connector runtime generation。
- 可访问且启用的App工具才参与请求工具规划；支持tool search时默认延迟暴露。
- Apps使用说明由现有上下文提供机制按当前可用状态动态注入。
- 鉴权elicitation、工具审批和文件上传属于Apps调用路径的必要组成部分。

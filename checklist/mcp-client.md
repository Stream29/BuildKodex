# MCP Client

- MCP服务器配置必须明确区分Streamable HTTP与stdio。
- `mcp:impl`直接调用`mcp:stdio`公开的transport函数，不为无状态调用引入工厂。
- Streamable HTTP请求头直接来自类型化服务器配置；没有动态来源时不得引入provider。
- `mcp:impl`当前只覆盖CLI host；Node.js stdio另行规划。
- stdio的JSON-RPC编解码交给官方MCP SDK，进程与原始管道交给`utils:process-client`。
- stdio stdout只承载协议数据，不得经过`ShellClient`的文本缓冲。
- `McpService.clients`必须按设置中的原始名称发布全部enabled client；connecting或failed client继续保留，disabled或已删除client不发布。
- 每个`McpClient`generation的`listTools()`必须固定；成功refresh、成功reconnect或对应配置更新时以新generation原子替换catalog。
- 连接owner必须发布`Connecting`、`Healthy`、`Failed(reason)`和`Closed`；公开失败原因只能使用稳定枚举，异常详情只写日志。
- 断线不得清空已发布catalog；每个工具通过其owner client派发，非healthy时不发起远程调用并返回Agent可见的server unavailable结果。
- 手动refresh只复用healthy连接刷新catalog；reconnect必须替换连接并重新加载catalog。

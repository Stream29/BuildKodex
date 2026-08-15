# MCP Client

- MCP 配置、认证、Settings 操作和 Codex 导入必须遵循[MCP 管理](mcp-management.md)。
- 首版运行时固定使用仓库内 Kotlin MCP SDK 支持的 `2025-11-25` 协议，并只向 Agent 投影 tools。
- MCP服务器配置必须明确区分Streamable HTTP与stdio。
- `mcp:impl`直接调用`mcp:stdio`公开的transport函数，不为无状态调用引入工厂。
- Streamable HTTP的非认证静态请求头直接来自类型化服务器配置；OAuth使用按服务器隔离、读取最新持久凭据的动态authorizer，不恢复无状态的通用headers provider。
- `mcp:impl`当前只覆盖CLI host；Node.js stdio另行规划。
- stdio的JSON-RPC编解码交给官方MCP SDK，进程与原始管道交给`utils:process-client`。
- stdio stdout只承载协议数据，不得经过`ShellClient`的文本缓冲。
- `McpService.clients`必须按设置中的原始名称发布全部enabled client；connecting、blocked或failed client继续保留，disabled或已删除client不发布。
- 每个`McpClient`generation的`listTools()`必须固定；成功refresh、成功reconnect或连接身份配置更新时以新generation原子替换catalog。
- 连接owner必须保留`Connecting`、`Healthy`、`Failed(reason)`和`Closed`，并为已启用但OAuth未初始化的服务器发布认证blocked状态；公开失败原因只能使用稳定枚举，异常详情只写日志。
- 断线或认证blocked不得清空已发布catalog；每个工具通过其owner client派发，非healthy时不发起远程调用并返回Agent可见的server unavailable结果。
- Settings reconciliation必须使用不含动态token值的连接身份投影；token刷新不得替换连接owner或catalog generation，登录、注销和连接身份变化必须触发对应生命周期转换。
- 手动refresh只复用healthy连接刷新catalog；reconnect必须替换连接并重新加载catalog。

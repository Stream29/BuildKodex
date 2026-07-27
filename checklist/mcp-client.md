# MCP Client

- MCP服务器配置必须明确区分Streamable HTTP与stdio。
- `mcp:impl`直接调用`mcp:stdio`公开的transport函数，不为无状态调用引入工厂。
- Streamable HTTP请求头直接来自类型化服务器配置；没有动态来源时不得引入provider。
- `mcp:impl`当前只覆盖CLI host；Node.js stdio另行规划。
- stdio的JSON-RPC编解码交给官方MCP SDK，进程与原始管道交给`utils:process-client`。
- stdio stdout只承载协议数据，不得经过`ShellClient`的文本缓冲。

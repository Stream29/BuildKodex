# RPC MCP 边界

- 按 [RPC 迁移适用范围](rpc-architecture.md#适用范围)使用本文件；协议与连接身份约束继续复用 [MCP 管理](mcp-management.md)，跨线职责采用以下目标。

## 配置与运行状态

- 通过完整 BackendSettings 的 [Get/Flow/CAS](rpc-settings.md#rpc-读写)编辑 mcpServers，不新增 MCP 字段级 Get/update、add/edit/delete/setEnabled RPC。
- 设置 CAS 接入必须保留原 manager 的校验、原子持久化、改名/删除、凭据身份和连接生命周期副作用，不以 Map 替换绕过它们。
- 用 GlobalRpc.getMcpServers/getMcpServersFlow 返回原 List<McpManagedServerState>，含禁用配置项，保留配置摘要、认证阶段、连接状态及工具数。
- 不传输带工具调用与 StateFlow 行为的 McpClient；配置 Initialized 只表示凭据存在，不代表当前认证或连接成功。
- 配置与运行状态不是联合原子快照；运行状态保持脱敏，不成为另一份可写配置真源。
- reconnectMcpServer(name) 操作现有准确名称的后端 client；缺失时失败，不隐式创建或启用服务器。
- reconnect 替换连接并刷新 catalog；失败保留前代 catalog 并发布失败状态。
- 单个前端停止观察不关闭共享 MCP client 或取消其工作。
- OAuth 使用[统一认证交互](rpc-authentication.md)，不恢复独立 startMcpLogin/completeMcpLogin 或按服务器当前 attempt 取消。

## 前端观察与展示

- 状态传输接入不改变原 manager/service 的业务逻辑或导入副作用；设置页观察并显示后端已经发布的状态，不因前后端分离裁剪后端行为。
- 将原 McpManager.servers 的观察替换为 getMcpServers 初始化与 getMcpServersFlow 持续更新，复用 [rpcStateIn](rpc-state.md#get-与-getflow)；不轮询、不逐服务器建立状态 RPC，也不伪造空列表作为初始化成功。
- 列表和详情继续使用原 McpManagedServerState 的脱敏字段；McpServerSettingsState/status 的纯展示映射留前端，不新增跨线 UI 状态 DTO。
- 连接、认证、工具数以该后端状态流为准，不因保存、登录或 reconnect 返回成功就在前端自行标为 Healthy；BackendSettings 用于编辑与 CAS，不从配置是否存在推导连接结果。
- 详情沿原 serverName 从最新列表定位；编辑草稿、弹窗选择和命令错误提示留前端，不成为第二份后端运行状态，也不要求配置流与运行流联合原子更新。

## Codex 配置读取

- GlobalRpc.getCodexMcpSettings 显式读取后端 Codex 目录的 config.toml/mcp_servers，返回原 List<McpCodexImportCandidate>。
- 只在显式导入需求时读取，不提供持续同步、配置 Flow、任意文件读取或 Codex OAuth token 导出。
- 返回按服务器名称排序的 Supported 完整可合并配置与 Unsupported 说明；缺少配置/section 为空列表，读取/解析错误仍报错。
- Supported.transport 由 configuration 推导，不重复传输冗余字段；Unsupported 保留 nullable transport 与 detail。
- 保留可导入 header/environment 等值；OAuth 配置为 Uninitialized，不导入登录凭据。前端展示继续脱敏，原载荷不进入日志。
- 前端负责过滤、选择、冲突分类、预览与合并，使用现有完整 BackendSettings CAS 提交。
- 后端不创建 previewId、activePreview 或 preview/apply RPC；CAS 失败遵循状态流推进，不盲目覆盖新设置。
- 配置保存后的连接、catalog 与 OAuth 身份协调仍属后端职责，不信任前端过时的预览校验。

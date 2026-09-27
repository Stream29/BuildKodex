# RPC MCP 边界

- 按 [RPC 迁移适用范围](rpc-architecture.md#适用范围)使用本文件；协议与连接身份约束继续复用 [MCP 管理](mcp-management.md)，跨线职责采用以下目标。

## 配置与运行状态

- 通过完整 BackendSettings 的 [Get/Flow/CAS](rpc-settings.md#rpc-读写)编辑 mcpServers，不新增 MCP 字段级 Get/update、add/edit/delete/setEnabled RPC。
- 设置 CAS 接入必须保留原 manager 的校验、原子持久化及实际配置变化所需的改名/删除、凭据身份和连接生命周期协调，不以 Map 替换绕过它们。
- 用 GlobalRpc.getMcpServers/getMcpServersFlow 返回原 List<McpManagedServerState>，含禁用配置项，保留配置摘要、认证阶段、连接状态及工具数。
- 不传输带工具调用与 StateFlow 行为的 McpClient；配置 Initialized 只表示凭据存在，不代表当前认证或连接成功。
- 配置与运行状态不是联合原子快照；运行状态保持脱敏，不成为另一份可写配置真源。
- reconnectMcpServer(name) 操作现有准确名称的后端 client；缺失时失败，不隐式创建或启用服务器。
- reconnect 替换连接并刷新 catalog；失败保留前代 catalog 并发布失败状态。
- 单个前端停止观察不关闭共享 MCP client 或取消其工作。
- OAuth 使用[统一认证交互](rpc-authentication.md)，不恢复独立 startMcpLogin/completeMcpLogin 或按服务器当前 attempt 取消。

## 前端观察与展示

- 设置页观察并显示后端已经发布的状态，观察接入本身不改变 manager/service 的业务逻辑；导入写侧采用下述[配置值更新语义](#导入提交语义)，不要求迁移旧 Replace 的额外副作用。
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
- 后端不创建 previewId、activePreview 或 preview/apply RPC；CAS 失败遵循[字段冲突与重试规则](rpc-state.md#settings-的字段冲突)，不盲目覆盖新设置。
- 配置保存后的连接、catalog 与 OAuth 身份协调仍属后端职责，不信任前端过时的预览校验。

## 导入提交语义

- 新路径只有配置值更新，没有独立的 Replace 命令语义；前端的冲突选择只影响合并后的 update，不向后端传递“强制替换”意图。
- 按既有完整值 CAS 比较：不匹配仍返回 false；当前值、expect 和 update 相同时返回 true 且不产生变化，不写盘、不额外 invalidate 或重连。
- 实际配置变化仍经后端原校验、持久化及连接/凭据身份协调；不因值来自导入就额外强制重建连接，也不忽略真正变化所需的副作用。
- 不迁移旧 applyCodexImport 中按 Replace 决策单独触发 invalidate 的要求，不以自动调用 reconnect 补回它；无需新增 RPC、参数或改变同值 CAS 契约。

## 接入等价性边界

- 观察接入与命令接入分别按当前已定语义验收：页面显示后端状态，不代表提交路径正确；保留旧 manager 方法也不证明新路径调用了它。已明确不迁移的 Replace 副作用不再作为等价性要求。
- 配置值变化与显式命令意图区分处理；不能从相同 expect/update 推导额外操作，也不能暗中赋予同值 CAS 原约定之外的副作用。
- 若后续发现仍需保留的行为无法由已定契约表达，按 [RPC 契约冻结](rpc-architecture.md#rpc-契约冻结)停下并重新审阅；不为完成当前批次自行改变语义或新增 RPC。

# MCP 管理

## 范围

- Kodex MCP 以仓库内 Kotlin MCP SDK 支持的 `2025-11-25` 协议为首版基线，不实现 `2026-07-28` 协议或新旧协议协商。
- MCP 配置只支持应用全局作用域，全部会话共享 `settings.yml` 中的服务器。
- 首版只向 Agent 提供 MCP tools，不实现 resources、prompts、completion、roots、sampling 或 elicitation。
- 首版不实现工具允许/禁用列表、逐工具审批、`required`、启动超时或工具调用超时。
- Node.js stdio、Codex Apps 和大型 MCP 工具结果展示性能分别由独立任务跟踪。

## 配置与身份

- `settings.yml` 中的 `mcp_servers` 是 Kodex MCP 配置和 OAuth 凭据的唯一持久化真源；常规设置加载不得自动读取、继承或合并 Codex MCP 配置。
- 每个服务器名称同时是稳定 ID；名称必须非空且全局唯一。
- 设置页允许改名，但改名必须按删除旧服务器后新增服务器处理，不得继承旧连接、catalog 或 OAuth 初始化状态。
- `McpServerConfiguration` 必须明确区分 Streamable HTTP 与 stdio，并将认证建模为类型化联合。
- OAuth 配置必须显式区分 `Uninitialized` 与携带持久凭据的 `Initialized`；`Initialized` 只表示凭据存在，不保证当前 access token 可用。
- `Uninitialized` OAuth 可以不含 client ID；登录必须优先复用已有身份，否则通过授权服务器的 `registration_endpoint` 动态注册。
- 动态客户端注册只发送必要的客户端元数据，并依赖 `authorization_code`/`code` 的标准默认值；发现到的 scope 只用于后续授权和 token 请求，不写入注册请求。
- OAuth 登录必须解析 Bearer challenge 的 `resource_metadata` 和 `scope`，按标准顺序发现 Protected Resource 与 OAuth/OIDC 授权服务器 metadata，并确认 PKCE `S256` 支持。
- OAuth 授权、授权码 token 和新登录产生的 refresh token 请求必须携带同一 canonical `resource`；显式 scopes 优先，否则依次使用 challenge scope 和 Protected Resource Metadata scopes。
- OAuth token、client secret 和其他敏感值直接序列化在对应服务器配置中，但必须使用默认脱敏的 secret 类型，禁止普通 data class 的 `toString()` 暴露真实值。
- 写入 MCP 凭据后，整个 `settings.yml` 必须按凭据文件保护、原子更新，且不得进入日志、错误文本、导入预览或前端公开状态。
- 禁用服务器必须保留其配置和 OAuth 凭据；删除服务器必须同时删除内嵌凭据。

## 管理与运行时

- `McpManager` 是设置页执行 MCP 添加、编辑、删除、启停、登录、注销、Codex 导入和 reconnect 的唯一命令边界。
- `McpService` 继续只拥有 application-wide client、连接、catalog、refresh 和 reconnect 生命周期；设置页不得直接修改其内部状态。
- `McpManager` 必须发布包含脱敏配置摘要、持久认证状态、运行时认证状态、连接状态和工具数量的组合状态，不得把这些维度压成一个枚举。
- 已启用但 OAuth 未初始化的服务器必须保留为可观察的逻辑 client，并发布认证前置条件未满足的 blocked 状态；不得将其误报为传输失败。
- 运行时认证状态必须至少区分等待登录、授权中、可用、刷新中、需要重新授权和失败；它不得覆盖持久化的 `Uninitialized`/`Initialized` 状态。
- 设置页和其他下游只能接收脱敏认证摘要；只有 HTTP 授权层可以读取真实 token。
- Streamable HTTP 的 OAuth authorizer 必须按服务器隔离并读取最新持久凭据；不得在共享 HTTP client 上安装可能把一个服务器 token 发往另一个服务器的全局认证状态。
- stdio 不使用 OAuth；其认证继续通过类型化配置中的进程环境提供。

## 配置变化

- 添加和编辑必须先使用未持久化 draft；只有用户确认且完整验证通过后才原子写入有效配置。
- 管理命令成功表示配置已经持久化；连接和 catalog 更新继续异步发布。
- 连接 reconciliation 必须比较不含动态 token 值的连接身份投影，不得因 access token 刷新替换 client owner 或 catalog generation。
- OAuth 从 `Uninitialized` 变为 `Initialized` 后必须建立连接；注销必须写回 `Uninitialized` 并关闭当前连接。
- URL、command、OAuth client、resource 或 scopes 等连接身份变化必须使旧连接失效；可能把凭据发送到不同目标的变化还必须清除旧 OAuth 初始化状态。
- token 刷新必须原子写回同一服务器的 `Initialized` 配置，并让后续 HTTP 请求立即读取新值，不得强制重连。
- reconnect 必须保留既有语义：替换连接并刷新 catalog，失败时保留上一代 catalog。

## Settings 管理

- Settings 的 MCP 区域必须提供添加、编辑、删除、启用、禁用、OAuth 登录、OAuth 注销、`Import from Codex` 和 reconnect。
- Settings 主层只保留 section 级 Add/Import 操作和每个服务器各一个紧凑按钮；不得平铺服务器详情或启停、编辑、删除、OAuth、reconnect 操作。
- 每个服务器按钮必须显示名称和简短状态；点击后打开详情弹窗，在弹窗内显示启用状态、认证状态、连接状态、healthy 工具数量和全部服务器操作。
- 详情弹窗可以显示 URL、command、args 等非敏感配置；header 值、environment 值和 OAuth 凭据必须默认脱敏。
- 删除和改名必须在用户确认后执行；删除关闭 client owner，改名按删除后新增处理。
- OAuth 登录必须使用一次性授权 attempt 驱动浏览器和回调 UI；动态注册获得的客户端身份必须在打开浏览器前持久化，完成后再持久化 `Initialized` 并由运行时建立连接。
- OAuth 注销保留服务器配置，将其持久状态改为 `Uninitialized`，并关闭当前连接。

## Codex 导入

- Codex MCP 配置只能通过 Settings 中显式的 `Import from Codex` 操作读取；点击后必须立即加载并直接显示脱敏选择列表，不得再要求用户点击 `Preview` 或进入二级入口，导入后不得与 Codex 保持同步。
- 选择列表必须逐项标记新增、同名冲突或不支持；全部可导入项默认选中，新增项默认 `Import`，同名冲突默认 `Replace`，不支持项保持不可选，用户可以手动取消任意可导入项。
- 同名冲突按服务器名称判定；替换必须清除被替换服务器的连接、catalog 和 OAuth 初始化状态。
- 导入选择必须在一次 settings 更新中原子提交；任何未选择或不支持的项不得改变现有 Kodex 配置。
- 导入只复制 Kodex 支持的服务器配置，不导入 Codex OAuth token；需要 OAuth 的导入项必须以 `Uninitialized` 状态保存，缺少 client ID 时交由登录动态注册。

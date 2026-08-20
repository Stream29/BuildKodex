# Task Tree

- [done] 验证并修复 MCP OAuth 与环境变量启动
  - [done] 检查 OAuth 请求、刷新和重连路径
  - [done] 检查 stdio 环境变量传递路径
  - [done] 复现实际 OAuth 兼容性缺口
  - [done] 支持 OAuth challenge 与 metadata 发现
  - [done] 支持无 client ID 的动态注册
  - [done] 保证授权与 token 请求携带 resource
  - [done] 持久化动态注册后的客户端身份
  - [done] 覆盖 stdio Env 的 Native 进程传递
  - [done] 运行 MCP、设置与平台验证

# Details

- 用户反馈 OAuth MCP 仍可能异常，并要求同时确认依赖环境变量的 MCP 能正常启动。
- 检查范围限于 MCP 认证与 stdio 进程环境，不扩展其他 MCP 能力。
- 实际 OAuth MCP 返回带 `resource_metadata` 的 Bearer challenge，并通过授权服务器 metadata 提供动态客户端注册；当前 Kodex 要求预置 `client_id`，无法完成该流程。
- 当前发现流程还会在未显式配置 resource 时漏掉授权请求的必需 `resource` 参数。
- 用户确认采用完整标准路径，包括 metadata 发现、动态注册和 resource 参数。
- 现有 JVM 真实 MCP 测试证明 stdio 配置值会进入子进程；本任务另用公共进程测试覆盖 Linux Native 后端。
- OAuth 配置允许暂不提供 client ID；登录时优先复用预注册身份，否则使用授权服务器的 `registration_endpoint`。
- 登录 attempt 将发布发现并注册后的未初始化配置，`McpManager` 在打开浏览器前先持久化该身份，避免取消后重复注册。
- 显式 OAuth scopes 优先；未配置时依次使用 Bearer challenge scope 和 Protected Resource Metadata scopes。
- 实际服务当前对 GET `/mcp` 返回 405，对合法 Streamable HTTP POST 返回带 `resource_metadata` 的 401；OAuth 预检因此使用 POST。
- JVM OAuth fixture 已覆盖 challenge、Protected Resource Metadata、授权服务器 metadata、PKCE、动态注册、回调和 token exchange。
- JVM stdio fixture 与 Linux Native 真实子进程测试均确认配置 Env 会覆盖到子进程；Windows Native 后端完成交叉编译。
- `device_as_mcp` 仍有独立协议版本不兼容：Kodex 固定官方 `2025-11-25`，该服务只接受 legacy `2025-06-18` 或其自定义 `2026-07-28`，不属于本任务范围。

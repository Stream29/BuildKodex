# Task Tree

- 规划并补齐完整 MCP 能力
  - 对照当前 Rust Codex 盘点配置、连接生命周期和工具执行语义
  - [done] 让内部 `McpServerConfiguration` 可直接序列化，移除 Codex Lite settings 的重复 DTO 与投影
    - [done] 将重复的 `PathAsStringSerializer` 收敛到公共 utils 模块
  - [done] 移除 MCP 客户端装配中的无状态转发抽象
    - [done] 让 `mcp:impl` 直接调用 `mcp:stdio` 的公开 transport 函数
    - [done] 移除没有动态实现来源的 request headers provider
  - 保留 Codex 原始 TOML 配置到内部有效配置的边界投影与校验
  - 补齐缺失的服务器、传输、鉴权、超时、工具过滤和审批配置
  - 补齐对应运行时行为、错误处理和配置热更新语义
  - 覆盖配置持久化、Streamable HTTP、stdio 和真实 MCP 生命周期测试

# Details

- `McpServerConfiguration` 已使用 `type` property discriminator 直接序列化；settings schema 2 只接受并直接使用当前类型，不保留旧版本兼容或迁移逻辑。
- `utils:kotlinx-io-serialization` 是仓库唯一的 `PathAsStringSerializer` 来源。
- 空壳 `mcp:client` 已移除；`mcp:impl` 直接使用官方 SDK 和 `mcp:stdio`。
- Streamable HTTP 请求头直接使用 `McpServerConfiguration` 中的静态 `headers`。
- 其余能力等待后续规划与排期。
- “完整能力”以当前 Rust Codex 支持的用户 MCP 能力为对齐边界，不等同于实现 MCP 协议的全部可选特性。
- 重点缺口包括环境变量来源的 header 与 bearer token、OAuth、启动与工具超时、`required`、并行工具标记、工具白黑名单、scope 和逐工具审批。
- Node.js stdio 的平台缺口继续由 `2026-07-27-plan-node-process-client-stdio.md` 跟踪。
- Codex Apps 保持独立任务；本任务只提供其依赖的通用 MCP 能力。

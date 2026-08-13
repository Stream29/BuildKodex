# Task Tree

- [done] 明确建模 OpenAI 认证不可用状态
  - [done] 盘点当前 message 来源与分支
  - [done] 确定稳定的不可用状态集合
  - [done] 迁移认证状态模型
    - [done] 将 `Unavailable` 改为无文本枚举
    - [done] 让 Application 摘要携带类型化原因
  - [done] 迁移认证状态生产者
    - [done] 区分预加载、缺失、模式和无效凭据
    - [done] 区分来源不可读和意外失败
    - [done] 仅在内部日志保留异常诊断
  - [done] 迁移认证状态消费者
    - [done] 为 OpenAI 请求生成内部错误说明
    - [done] 让账号用量停止复制认证错误文本
    - [done] 为 CLI 设置生成展示说明
  - [done] 更新测试与边界文档
  - [done] 运行相关编译和测试

# Details

- 用户要求移除 `OpenAiAuthState.Unavailable.message`，改为明确的子状态枚举，避免用自由文本掩盖状态复杂度。
- 本任务延续只读 `OpenAiAuthStore` 边界，不扩大其能力。
- `Unavailable` 使用 `NotLoaded`、`CredentialsNotFound`、`UnsupportedAuthMode`、`InvalidCredentials`、`CredentialSourceUnavailable` 和 `UnexpectedFailure` 六个稳定值。
- 缺失文件和不支持的认证模式属于预期状态；序列化失败映射为无效凭据，文件系统失败映射为凭据来源不可用，其他异常映射为意外失败。
- Throwable 和原始错误文本只用于实现侧日志，不进入 OpenAI 或 Application contract。
- 具体修改和验证路径已经确定，任务进入执行阶段。
- `CodexAccountUsageState.Unavailable` 只表达账号用量不可用，不复制认证原因或异常文本；具体认证原因仍由 `OpenAiAuthState` 唯一表达。
- 相关 JVM production source 已使用 OpenJDK 26.0.2 编译通过。
- 相关 JVM test source 编译通过；未运行包含真实 OpenAI 请求 probe 的完整 JVM test task。
- `app-shared-auth-filesystem`、`openai-account-usage-impl`、`app-viewmodel-settings-login` 和 `app-cli-view-application` 的 Linux X64 测试通过，共 63 项测试、0 失败。
- `git diff --check` 在主仓库和 `Kodex/` 子模块均通过。
- IDEA 正在打开项目；官方 MCP 的 reformat 请求因 IDE MCP session 不可用而未执行，IDE CLI format 又不能与当前实例并行运行。

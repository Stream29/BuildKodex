# Task Tree

- [done] 新增 OpenAI 登录协议客户端
  - [done] 定义独立的登录客户端 contract 与协议数据模型
  - [done] 实现授权码换取与刷新 token 请求
  - [done] 将私有凭据续期改由登录客户端执行
  - [done] 补齐协议客户端与认证存储集成测试
  - [done] 运行相关 Gradle 验证

# Details

- `OpenAiLoginClient`实现 OAuth 授权 URL、PKCE 授权码交换与 token 刷新；`OpenAiClient`继续只承担已认证的 Responses/API 调用。
- 刷新沿用旧的瞬态重试策略；授权码交换不重试。
- 已验证：`openai-client` 登录用例、`openai-models:jvmTest`、`openai-codex-cli-storage:jvmTest`、`cli-auth-filesystem:jvmTest`。
- 本任务不包含设置页面、登录按钮、浏览器打开或 callback UI。

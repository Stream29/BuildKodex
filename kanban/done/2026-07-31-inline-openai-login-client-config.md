# Task Tree

- [done] 内联 OpenAiLoginClient 固定配置
  - [done] 移除没有调用方定制需求的配置对象
  - [done] 调整测试构造入口
  - [done] 运行定向 Gradle 验证

# Details

- 保留 `CODEX_REFRESH_TOKEN_URL_OVERRIDE` 与 `CODEX_APP_SERVER_LOGIN_CLIENT_ID` 的既有覆盖语义。
- 已验证：`TESTBALLOON_INCLUDE_PATTERNS='*openAiLoginClientTest*' :openai-client:jvmTest`。

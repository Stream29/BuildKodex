# Task Tree

- [done] 实现 `web.run`
  - [done] 定义 Rust 对齐的 `web.run` schema 与 typed handler
  - [done] 复用 OpenAI 搜索客户端执行命令并返回函数输出
  - [done] 接入通用 `KodexToolRuntime` 并覆盖跨平台测试

# Details

`web.run` 是普通 namespace JSON tool，不是 hosted `web_search` tool spec。现有 `SearchCommands` 与 `OpenAiClient.search` 作为底层 API。

`KodexToolRuntime` 已按 `ResponsesApiNamespace` 路由 `web.run`，无需专用注册代码。真实端点与 Tool handler 已在 JVM、Node JS、Linux x64 通过；Linux ARM64 和 MinGW 测试源集已编译。macOS 实机验证未执行，因为当前 `macbook` 主机名无法解析。

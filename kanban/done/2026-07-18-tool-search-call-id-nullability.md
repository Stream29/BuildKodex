# Task Tree

- [done] 核查 tool search call 的 `callId` 可空性
  - [done] 对照 Responses API、Rust 模型与调用路径
  - [done] 确认 Kotlin 模型应保留或移除可空性
  - [done] 运行受影响的验证并归档结论

# Details

官方协议规定 hosted `execution = "server"` 的 tool-search call/output 使用 `call_id = null`；只有 client execution 必须携带并回显非空 id。Kotlin wire DTO 保留可空性。

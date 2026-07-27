# Task Tree

- [done] 将SSE流式请求生命周期封装到`utils:ktor-client-ext`
  - [done] 设计并实现通用的流式POST请求API
  - [done] 让`OpenAiClient`只使用该API建立Responses流
  - [done] 覆盖默认body保存不会重新出现的回归
  - [done] 让非2xx响应保留原始HTTP错误而不是误报`SSESession`类型不匹配
  - [done] 用真实`clock.curr_time`工具往返重放现场对话
  - [done] 根据真实400响应修正Responses工具声明投影
  - [done] 在真实Responses API与TUI链路验证

# Details

`SseCompatibility`只适配响应类型，通用流式POST API承接Ktor的`preparePost(...).body`执行路径，避免下游通过普通`post`误用。

真实400由Kotlin错误将内部`outputSchema`序列化为Responses工具声明中的`output_schema`引起。Rust字段使用`#[serde(skip)]`；当前实现保留内部schema和自然语言工具结果，只在普通Responses wire中省略该元数据。

JVM、Linux Native与macOS Native测试通过。真实Responses API验证文本delta按时间增量到达；真实TUI验证连续流式渲染、工具往返、错误恢复和取消后继续对话。

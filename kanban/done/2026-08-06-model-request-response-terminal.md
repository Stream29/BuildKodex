# Task Tree

- [done] 收敛 requestResponseApi 正常终态并补充追溯日志
  - [done] 将正常返回值收敛为 `Finish` 与 `Resumable`
  - [done] 将 failed、incomplete 与无终态流结束建模为保留诊断信息的异常
  - [done] 让 compaction runtime 只续跑 `Resumable` 并记录成功或失败终态
  - [done] 更新受影响调用方、测试与运行时设计记录
  - [done] 运行受影响的 Gradle 验证

# Details

- `ToolPending`只由`AgentStateValue`表达。
- `response.incomplete`不推断可重试性，只保留服务端提供的reason。
- context window相关失败不在runtime内自动捕获或压缩。
- 在openai model层定义failed、incomplete和无completed事件三类异常；异常消息携带可检索的协议字段。
- AgentState消费事件并返回二值enum；compaction runtime已有agent logger，负责在边界记录正常结果或异常。
- 验证覆盖openai models、agent-state、compact runtime及直接实现测试runtime的下游模块。
- 验证：`git diff --check`通过；openai models、agent-state、全部runtime decorator、tool plan与agent session JVM测试通过；应用与integration test JVM源码编译通过。

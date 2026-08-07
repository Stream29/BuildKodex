# Task Tree

- [done] 限制 Responses 协议重试次数
  - [done] 拆分正常续跑与可重试结果
  - [done] 将连续协议重试限制为四次
  - [done] 覆盖续跑、重试与耗尽测试
  - [done] 更新运行时契约记录
  - [done] 运行相关 Gradle 验证

# Details

- 用户决定从当前 `Resumable` 结果中分出 `Continue`，让 `Retryable` 仅表示 `response.failed` 或无协议终态断流。
- 正常 `Continue` 不占用重试次数；每次 Responses 请求最多允许初次请求后的四次协议重试。
- `KodexAgentStateImpl`负责投影单次请求结果；`KodexAgentCompactionRuntime`负责跨请求计数并在耗尽时终止。
- 修改范围限定为 AgentState 契约与实现、compaction runtime、对应测试和既有运行时决策记录。
- 验证通过：`agent-state-impl:jvmTest`、`agent-runtime-decorator-compact:jvmTest`、`agent-runtime-decorator-tool:jvmTest`、`integration-test:compileTestKotlinJvm`、IDEA定向构建及`git diff --check`。

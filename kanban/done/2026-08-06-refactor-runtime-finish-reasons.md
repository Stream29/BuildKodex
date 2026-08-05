# Task Tree

- [done] 将 Responses 与 runtime 的流式返回收敛为完成原因
  - [done] 确认 `AgentStateValue.RequestResponse` 已拥有活动输出的重放 `SharedFlow`
  - [done] 确认当前 `Flow` 仅被 runtime 用于传递事件和读取终端 `end_turn`
  - [done] 定义各层完成原因及其消除关系
    - `RequestFinish` 表达单次请求的续跑、结束、失败、不完整和待处理工具
    - compaction 消除续跑；tool runtime 消除可本地执行的待处理工具；turn hook 增加 hook 停止原因
    - `AgentRuntimeFinishReason` 保留宿主仍需处理的工具、hook 停止及终端响应结果
  - [done] 将 `requestResponseApi()` 改为返回其专属完成原因
  - [done] 将 `ResumableAgentLayer` 改为完成原因泛型接口
  - [done] 更新装饰器、组合根与受影响测试
  - [done] 运行受影响的 Gradle 验证

# Details

- `OutputItemDone` 会先将当前输出落盘并释放其 `SharedFlow`；随后 `response.completed.end_turn` 没有可从 `AgentStateValue` 重建的持久位置。
- 当前 compaction 层从原始事件流读取 `end_turn == false` 以决定是否续跑；tool 层通过 `ToolPending` 决定是否执行或交给宿主完成工具。
- 已决定保留类型化完成原因，不将 `AgentRuntime.resume()` 收敛为 `Unit`。`request_user_input` 等宿主完成的 pending tool 使最外层仍有可观察的正常退出语义。
- 网络异常和协程取消继续通过异常传播；Responses 协议中的 `failed`、`incomplete` 与缺少终端帧以完成原因传播，保持正常收尾而不伪造成功。
- 已通过 agent-state、runtime decorators、in-memory session、tool-plan 的 JVM 测试，以及 integration-test 和应用层的 JVM 编译。

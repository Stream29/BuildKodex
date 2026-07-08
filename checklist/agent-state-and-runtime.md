# AgentState 与 AgentRuntime

修改 `agent-state` 或引入 `agent-runtime` 时使用此 checklist。

- 明确区分 `AgentState` 与 `AgentRuntime`。
- `AgentState` 封装 LLM API 调用与上下文状态维护。
- `AgentState` 不承担“产生 agent 对环境的副作用”的逻辑。
- `AgentRuntime` 执行 agent 对环境的副作用。
- skill 与 `AGENTS.md` 的编排和 harness 支持属于 `AgentRuntime`。
- 不把一次 Responses API 调用完成等同于 agent run 完成。
- `AgentState.resume()` 的结果需要区分 turn complete、内部继续采样、需要 Runtime 接手、失败或取消。
- client-side tool call 对 `AgentState` 是显式暂停点，后续由 `AgentRuntime` 执行工具并写回结果。
- `end_turn == false` 且没有 Runtime-bound item 时，`AgentState` 可以自行继续采样。
- 运行中干预必须作为 Runtime 编排协议建模，不要用普通 history append 模拟。
- 运行中干预至少覆盖强行插入、边界插入、协作式中止 agent run。
- 运行中干预必须有 admission policy，校验 active turn、expected turn id、turn kind 和输入非空。
- 运行中输入必须先进入 pending queue，再由明确 delivery boundary 决定何时写入 history。
- tool call、steer、assistant answer boundary、compaction 后 continuation 都要显式定义 pending input 的 drain 时机。
- mailbox 或跨 agent 通信不能无条件进入当前 turn；answer boundary 后默认应能延后到下一 turn。
- interrupt/cancel 不等同于追加用户消息；它需要独立 cancellation policy 和可选 interrupted marker。
- stop hook continuation 与用户强行 interrupt 是不同控制流，不要共用一个普通 message 路径。

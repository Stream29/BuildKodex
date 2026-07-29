# 中止、继续与运行中干预

## 术语边界

- Responses API stream 结束：一次 HTTP/SSE 请求结束。
- sampling request 结束：一次模型采样结束，可能还需要继续采样。
- agent turn 完成：当前用户 turn 不再需要模型或 Runtime 继续处理。
- AgentState 暂停：模型产出需要外界副作用，例如 client-side tool call。
- Runtime 中断：宿主主动取消、插入输入、steer 或通过 hook 阻止流程。

这些边界不能合并。一次 Responses API stream 结束，不等于 agent turn 完成。

## Responses Stream 结束条件

外层 stream 可能以这些方式结束：

- `response.completed`：本次 API stream 正常完成，但 agent turn 未必完成。
- `response.failed`：服务端失败；Rust 会把它转换成错误。
- `response.incomplete`：服务端不完整结束；Rust 也把它转换成错误。
- stream 在 `response.completed` 前断开：协议或传输失败；Rust 会按 retryable stream error 处理。
- 外界 cancellation：当前 turn 被宿主主动中止。

这层只说明网络/协议请求结束，不直接决定 agent run 是否结束。

## Completed 后的语义判断

`response.completed` 之后要看 output item 和 `end_turn`。

- client-side tool call：AgentState 必须暂停，把待执行副作用交给 Runtime。
- hosted tool item：服务端已经处理，AgentState 只需要记录或投影。
- `end_turn == false`：模型明确要求继续采样；如果没有 Runtime-bound item，AgentState 可以自己发下一次 request。
- `end_turn == null`：不能单独当作继续信号；部分 provider 不提供该字段，需要靠 output item 语义兜底。
- 没有 Runtime-bound item，且 `end_turn != false`：当前 agent turn 可以完成。

因此 Kotlin 侧不应该把采样结果压成 `Boolean needsFollowUp`。更合适的是明确投影成：

- turn complete
- continue sampling inside AgentState
- requires Runtime
- failed
- cancelled

## AgentState 暂停条件

AgentState 只负责 LLM API 调用和上下文状态维护。遇到需要环境副作用的内容时，应该暂停并把控制权交给 Runtime。

典型暂停条件：

- client-side tool call。
- 需要用户交互的特殊工具。
- 需要 Runtime 审批、权限、sandbox 或 filesystem 选择的操作。
- 需要 Runtime 执行 hook continuation 的场景。

暂停不是失败，也不是 turn 完成。Runtime 写回 tool result 或其他后续输入后，AgentState 才能继续采样。

## 运行中干预方式

Rust 侧至少有三类运行中干预：

- 强行插入：外界不等模型自然停下，直接向当前 run 注入新的输入或控制项。
- 边界插入：外界等待下一次 tool call、模型完成点或其他安全边界，再插入新的输入或控制项。
- 协作式中止：外界请求当前 run 自行停下，让模型或 Runtime 在合适位置完成收尾。

这些不是普通 tool payload，也不是普通 assistant message。它们属于 Runtime 对 AgentState 的编排协议。

## Pending Input 与 Delivery Boundary

Rust 的 `InputQueue` 不是简单 list。

它维护：

- pending input：已被 Runtime 接受，但尚未进入模型历史。
- mailbox delivery phase：决定 mailbox 是否能进入当前 turn。
- delivery boundary：立即 drain、等待 tool/model continuation 后 drain、或延后到下一 turn。

turn loop 只在 `can_drain_pending_input` 为 true 时把 pending input 写入 history。新 turn 的首个输入要先采样，auto-compact 后也要先恢复 model/tool continuation。

相关源码：

- `shared-context/codex/codex-rs/core/src/session/input_queue.rs:12`
- `shared-context/codex/codex-rs/core/src/session/input_queue.rs:28`
- `shared-context/codex/codex-rs/core/src/session/turn.rs:197`

## Steer

`turn/steer` 是运行中追加输入的路径，不是普通 history append。

Rust 要求：

- 当前存在 active turn。
- expected turn id 匹配。
- 当前 turn 类型允许 steer。
- review/compact 等不可 steer 的 turn 会被拒绝。

`steer_input` 成功后会追加 pending input，并重新允许 mailbox delivery 进入当前 turn。

相关源码：

- `shared-context/codex/codex-rs/app-server/src/request_processors/turn_processor.rs:783`
- `shared-context/codex/codex-rs/core/src/session/mod.rs:3414`
- `shared-context/codex/codex-rs/core/src/session/mod.rs:3471`

## Interrupt

`turn/interrupt` 是主动取消当前 turn 的路径。

它不是追加一条“用户取消”消息，而是提交 `Op::Interrupt`，等待 `TurnAborted`。

Rust 的任务取消逻辑：

- cancel running task。
- 给任务一个 100ms graceful window。
- 超时后 abort handle。
- 必要时写入模型可见 interrupted marker。

相关源码：

- `shared-context/codex/codex-rs/app-server/src/request_processors/turn_processor.rs:1247`
- `shared-context/codex/codex-rs/core/src/tasks/mod.rs:65`
- `shared-context/codex/codex-rs/core/src/tasks/mod.rs:799`

## Hook Block 与 Hook Continuation

hooks 可以改变 turn 的中止/继续路径。

user prompt submit hook 可以 block 输入。被 block 的 pending input 不会作为普通用户输入记录。

stop hook 可以在模型本来要结束时提供 continuation fragments：

- Runtime 构造 hook prompt message。
- 该 message 作为模型可见 user message 写入 conversation history。
- sampling loop 继续。

这说明 hook 不是 tool。它位于 turn 边界和 tool 边界周围，可以阻止输入、追加上下文或强制继续。

相关源码：

- `shared-context/codex/codex-rs/core/src/session/turn.rs:322`
- `shared-context/codex/codex-rs/core/src/session/turn.rs:430`
- `shared-context/codex/codex-rs/core/src/hook_runtime.rs:499`
- `shared-context/codex/codex-rs/protocol/src/items.rs:375`

## Compaction 与中止的关系

compaction 不是 turn 完成条件，也不是普通 assistant message。

它是运行时维护上下文窗口的任务：

- 可以在 turn 前触发。
- 可以在 context limit 达到时 mid-turn 触发。
- 完成后替换 active model context，并让原 turn 继续。

因此 compaction 应被视为 Runtime/AgentState 内部维护动作，而不是用户可见的 agent run 中止。

## 对 Codex Lite 的建模影响

Kotlin 侧需要把这些状态显式建模：

- API stream outcome：请求层成功、失败、不完整、取消。
- sampling outcome：继续采样、完成、需要 Runtime。
- runtime intervention：steer、interrupt、pending input admission。
- cancellation policy：硬取消、graceful cancel、协作式停止。
- delivery boundary：当前 turn drain、等待 continuation、延后下一 turn。

核心原则：

- `OpenAiClient` 只处理 HTTP/SSE 和网络级 retry。
- `AgentState` 处理 LLM request、history 发布、自动 compaction、无副作用的继续采样。
- `ResumableAgentLayer` 处理工具执行、用户交互、hooks、pending input、steer、interrupt，以及所有环境副作用。

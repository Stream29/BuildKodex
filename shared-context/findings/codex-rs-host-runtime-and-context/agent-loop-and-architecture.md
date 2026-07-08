# Agent Loop 与架构影响

## 纯 Agent Loop 到底是什么

窄义的纯 loop 大致只有这些职责：

- 接收当前 prompt/history。
- 发模型请求。
- 接收 stream。
- 把 stream 投影成 assistant messages、reasoning 和 tool calls。
- 通过 runtime boundary 执行 required tool calls。
- 追加 tool outputs。
- 如果模型还需要 follow-up，就继续。

除此之外几乎都是 host runtime。

Rust `run_turn` 的注释描述了采样模式：function call 会导致工具执行，并把 output 在下一次 sampling request 发回模型；如果只有 assistant message，就记录历史并完成 turn。注释位置是 `shared-context/codex/codex-rs/core/src/session/turn.rs:130`。

## 中止、继续与运行中干预

Responses 调用结束、sampling request 结束、agent turn 完成、AgentState 暂停和 Runtime 中断是不同概念。详细规则整理在 [termination-and-interruption.md](termination-and-interruption.md)。

即使在 `run_turn` 内部，函数开头也先做运行时操作：

- pre-sampling compaction
- context updates
- skill/plugin injection
- session-start hooks
- input hooks
- previous-turn settings update

这些都发生在纯模型 loop 之前。调用序列从 `shared-context/codex/codex-rs/core/src/session/turn.rs:145` 开始。

## 对 Codex Lite 的架构影响

Kotlin 项目应该拆开这些职责：

- `agent-runtime` 或同等模块负责 session construction、context assembly、turn execution 和 host services。
- `agent-storage` 负责 raw durable storage 和 versioned state lines。
- `agent-state` 或 clean projection 模块负责 UI-facing stable/unstable projections。
- `tool:contract` 负责通用 tool payload/result 边界。
- `tool:impl:*` 负责具体工具业务逻辑。
- tool orchestration 放在 runtime，不放在单个 tool handler。
- `openai:*` 保持 typed client/model layer，不拥有 agent runtime policy。
- `utils:*` 继续作为无状态基础设施。

第一阶段优先实现：

- raw storage 和 turn snapshots。
- initial context assembly。
- context baseline 和 diff recording。
- 最小 OpenAI streaming response projection。
- JSON tools 加特殊 custom apply-patch 路径。
- user、assistant、thinking summary、compaction、plan marker、patch event 的 clean model projection。

第二阶段优先实现：

- `AGENTS.md` provider 和 project scanner。
- skills metadata catalog。
- explicit skill injection。
- token-count timeline 和 compaction threshold estimation。

后续再做：

- hook runtime。
- plugin/MCP/apps。
- implicit skill invocation telemetry。
- sandbox/approval parity。
- multi-agent routing。
- extension context contributors。

这个顺序能先让基础 loop 跑起来，同时不堵死 Rust parity。

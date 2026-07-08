# Context 与 Hooks

## TurnContext

`TurnContext` 是 per-turn runtime snapshot。它携带的信息远不止 user input。

它包括：

- config
- auth/model/provider information
- reasoning settings
- session source
- environment selections
- cwd
- 当前日期和时区
- developer instructions
- compact prompt
- user instructions
- collaboration mode
- approval policy
- permission profile
- sandbox/network settings
- feature flags
- dynamic tools
- loaded skills
- extension data

构造路径会写入 loaded skills 和 user instructions，位置是 `shared-context/codex/codex-rs/core/src/session/turn_context.rs:560`。

实际含义是：turn 不是 `(history, user message)`，而是 `(history, user message, runtime snapshot)`。

Kotlin 侧应该保留这个边界。最小 loop 可以接收一个准备好的 immutable turn snapshot，而不是去读全局状态。

## Initial Context Builder

`Session::build_initial_context` 会把 runtime state 转成模型可见的 `ResponseItem`。

它聚合：

- model switch instructions
- permission instructions
- developer instructions
- collaboration mode instructions
- realtime/date/time updates
- personality instructions
- apps instructions
- available skills instructions
- available plugins instructions
- extension context contributors
- `AGENTS.md` user instructions
- token budget context
- environment context
- multi-agent usage hints
- guardian policy prompts

函数入口是 `shared-context/codex/codex-rs/core/src/session/mod.rs:2871`。

这是大量“非纯 loop 上下文”的主要来源。这些内容大多不是模型生成的，不是用户直接输入的，也不是工具返回的。宿主注入它们，是因为模型需要正确的运行协议。

Kotlin 侧最好有单独的 context assembly 层。它产出明确的 raw `ResponseItem` 或等价内部模型输入项。loop 不应该分别认识每一个 feature。

## Context Diffs

Rust 避免每个 turn 重发完整上下文。

正常路径是：

- 没有 reference context 时，调用 `build_initial_context`。
- 有 reference context 时，调用 `build_settings_update_items`。
- 把产生的 context items 记录进 history。
- 持久化当前 `TurnContextItem` 作为新的 baseline。

实现位置是 `record_context_updates_and_set_reference_context_item`，见 `shared-context/codex/codex-rs/core/src/session/mod.rs:3189`。

diff builders 位于 `shared-context/codex/codex-rs/core/src/context_manager/updates.rs`。

它们会比较上一份和当前 runtime state，包括：

- environment
- permissions
- collaboration mode
- realtime settings
- personality/model instructions

对 Kotlin 来说，这和 storage 设计强相关：

- raw history 保存模型可见的 context updates。
- settings snapshots 提供 diff baseline。
- compaction 可以重置 baseline。
- clean UI models 应该从 raw history 加 stable state lines 投影出来。

## Hook Runtime

hooks 是另一类宿主侧不纯机制。

hook runtime 可以：

- 停止 turn
- 添加上下文
- 重写 tool input
- 阻止 tool
- 执行生命周期回调
- 发 hook started/completed events

核心 outcome 是 `HookRuntimeOutcome`，包含 `should_stop` 和 `additional_contexts`，位置是 `shared-context/codex/codex-rs/core/src/hook_runtime.rs:50`。

session-start hooks 由 `run_pending_session_start_hooks` 处理，入口是 `shared-context/codex/codex-rs/core/src/hook_runtime.rs:101`。

user prompt submit hook 会构造 request，里面包括 session id、turn id、subagent context、cwd、transcript path、model、permission mode 和 prompt text。代码位置是 `shared-context/codex/codex-rs/core/src/hook_runtime.rs:500`。

turn runner 会在记录用户输入之前执行 hooks：

- `run_pending_session_start_hooks` 先于 input hooks。
- `run_hooks_and_record_inputs` 检查每个 pending input。
- hook additional contexts 会被记录成模型可见消息。

调用点在 `shared-context/codex/codex-rs/core/src/session/turn.rs:165` 和 `shared-context/codex/codex-rs/core/src/session/turn.rs:430`。

普通 additional contexts 会渲染成 developer-role contextual fragments，没有额外 marker。renderer 在 `shared-context/codex/codex-rs/core/src/context/hook_additional_context.rs:14`。

Kotlin 侧不应该把 hooks 建模成 tools。它们是 turn 和 tool 边界周围的宿主 callbacks。

## HookPrompt

`HookPrompt` 是 hook 驱动 continuation prompt 的协议/UI item。

它是 `TurnItem` 的一个 variant，见 `shared-context/codex/codex-rs/protocol/src/items.rs:42`。

payload 包括：

- `HookPromptItem`
- generated id
- `HookPromptFragment` 列表
- 每个 fragment 有 text 和 hook run id

数据结构在 `shared-context/codex/codex-rs/protocol/src/items.rs:65`。

stop-hook 路径展示了真实行为：

- 当模型不再需要 follow-up 时，Rust 运行 stop hooks。
- 如果 stop hook block 了当前 turn，并提供 continuation fragments，Codex 会把 fragments 构造成 user-role message。
- 它会把这个 message 记录进 conversation history。
- 然后继续 sampling loop。

调用点在 `shared-context/codex/codex-rs/core/src/session/turn.rs:322`。

message builder 是 `build_hook_prompt_message`，位置是 `shared-context/codex/codex-rs/protocol/src/items.rs:375`。

这不是 goal state，不是 `request_user_input`，也不是普通用户输入。它是 host hook 强制模型继续生成的一条模型可见 user message。

Codex Lite 可以暂缓这块，直到 hook support 存在。但 clean model 应该留出独立 hook-prompt item 的空间，不要永远把它压平成 user message。

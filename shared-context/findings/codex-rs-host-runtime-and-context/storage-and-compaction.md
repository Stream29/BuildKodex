# Storage 与 Compaction

## Compaction

compaction 不是普通 assistant message。

它是一个运行时操作，会：

- 运行 pre-compact hooks。
- clone 当前 history。
- 获取 base instructions。
- 必要时裁剪 function-call history。
- 构造 compaction prompt。
- 带着当前 tool specs 和 base instructions 发起模型请求。
- 接收 compacted output。
- 构造 replacement history。
- 推进 context window id。
- 安装 compacted history 和 checkpoint。
- 重新计算 token usage。
- 发 completed turn item。

remote v2 compaction 流程起点是 `shared-context/codex/codex-rs/core/src/compact_remote_v2.rs:101`。

replacement-history 安装发生在 `shared-context/codex/codex-rs/core/src/compact_remote_v2.rs:301`。

普通 turn loop 也会检查 token status，并在 context limit 达到时触发 mid-turn compaction。token status 计算在 `shared-context/codex/codex-rs/core/src/session/turn.rs:733`。

对 Kotlin storage 来说：

- compaction 应建模成 checkpoint/prefix update。
- raw append-only history 继续作为 audit trail。
- active model context 可以是 compaction 后的 projection。
- clean UI events 应把 compaction 作为独立 stable event。

这和“context compaction 等于 set base index + set prefix，而不是任意 mutation 旧 records”的思路一致。

## Thread Storage 与 Rollout Items

Rust 会同时持久化模型可见 conversation items 和 runtime context snapshots。

Session 初始化会 create 或 resume 一个 `LiveThread`，位置是 `shared-context/codex/codex-rs/core/src/session/session.rs:529`。

context updates 会作为 conversation items 记录，而 `TurnContextItem` 会作为 rollout metadata 持久化。路径在 `shared-context/codex/codex-rs/core/src/session/mod.rs:3175`。

compaction 会持久化带 replacement history 和 window id 的 `CompactedItem`，位置是 `shared-context/codex/codex-rs/core/src/session/mod.rs:3153`。

这个分裂很重要：

- model-visible history 是模型实际收到的东西。
- rollout/thread metadata 是宿主用来 replay、resume、diff 和重建 UI 的东西。

Kotlin 侧应继续把 raw storage 和 clean UI projection 分开。clean model 应该是派生结果，而不是唯一事实源。

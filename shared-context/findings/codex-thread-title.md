# Codex Rust 会话名称

- 本地 Rust Codex 不会为普通线程额外发起 AI 标题生成请求。
- 首条可命名的文本用户消息在标题为空时初始化标题；之后不会自动更新。
- 标题取 `## My request for Codex:` 标记后的 trim 文本；没有该标记时取完整文本的 trim 结果。
- 仅图片输入可产生 `[Image]` 预览，但不会产生标题。
- `thread/name/set` 可以显式覆盖标题；fork 继承源线程标题。

对应实现位于 `shared-context/codex/codex-rs/state/src/extract.rs`、`shared-context/codex/codex-rs/protocol/src/protocol.rs` 和 `shared-context/codex/codex-rs/app-server/src/request_processors/thread_processor.rs`。

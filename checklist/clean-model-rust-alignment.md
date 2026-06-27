# Clean Model Rust Alignment

- Keep the current stable clean scope limited to user messages, assistant text, reasoning display, context compaction, plan update markers, and `apply_patch`.
- Before adding a new stable clean event, compare it against Rust `TurnItem` and the OpenAI raw item shape.
- Do not introduce a generic stable tool event until repeated concrete tool models prove it is useful.
- Keep `apply_patch` as its own event with parsed diff plus execution result.
- Add web search modeling only after `tool:web-search` is redesigned.
- Add image view modeling only after the clean projection needs image inspection events.
- Add image generation modeling only after the OpenAI image API abstraction and tool output shape are stable.
- Revisit HookPrompt only if hook runtime support becomes in-scope; otherwise keep hook continuation text as ordinary user input.
- Revisit MCP tool calls only if MCP support becomes in-scope.
- Reconcile assistant message `phase` with Rust before UI code depends on assistant message completion state.
- Reconcile assistant memory citations with Rust before adding citation rendering.
- Reconcile reasoning raw content with Rust only if the provider exposes raw reasoning content.
- For each new clean event, add serialization round-trip tests.
- For each new projection, add tests from raw OpenAI or tool state into the stable clean model.

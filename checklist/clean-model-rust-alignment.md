# Clean Model Rust Alignment

- Keep the stable clean scope centered on user messages, assistant text, reasoning display, context compaction, plan update markers, and completed tool interactions.
- Before adding a new stable clean event, compare it against Rust `TurnItem` and the OpenAI raw item shape.
- Use `StableJsonToolEvent` when a completed function tool has JSON arguments and a JSON result.
- Use `StableTextToolEvent` when a completed function tool has JSON arguments and a text result.
- Keep MCP tool interactions on the generic JSON fallback and store the complete `CallToolResult` envelope instead of flattening it to function output.
- Use dedicated strong types for project-owned tool schemas: tool search, image view, image generation, command execution, Multi-agent, request user input, and web search.
- Define dedicated tool payloads in clean models instead of retaining raw `JsonElement` fields or depending on runtime/tool-module DTOs.
- Keep `apply_patch` as its own event with parsed diff plus execution result.
- Revisit HookPrompt only if hook runtime support becomes in-scope; otherwise keep hook continuation text as ordinary user input.
- Reconcile assistant message `phase` with Rust before UI code depends on assistant message completion state.
- Reconcile assistant memory citations with Rust before adding citation rendering.
- Reconcile reasoning raw content with Rust only if the provider exposes raw reasoning content.
- For each new clean event, add serialization round-trip tests.
- For each new projection, add tests from raw OpenAI or tool state into the stable clean model.

# LLM Provider Model Alignment

Use this checklist when changing `CodexLite/llm-provider` protocol models.

- Treat Rust `shared-context/codex/codex-rs/tools/src/tool_spec.rs` `ToolSpec` as the source of truth for `LlmTool`.
- Keep `LlmTool` variants aligned with Rust: `function`, `namespace`, `tool_search`, `image_generation`, `web_search`, `custom`.
- Keep `LlmTool.Function` and `LlmNamespaceTool.Function` aligned with Rust `ResponsesApiTool`.
- Keep `LlmTool.Custom` aligned with Rust `FreeformTool`.
- Keep `LlmJsonSchema` aligned with Rust `JsonSchema`.
- Use explicit `@SerialName` for wire names that are not Kotlin property names.
- Do not use a global JSON naming strategy.
- Use kotlinx polymorphic serialization for tagged protocol variants.
- Use small custom serializers only for Rust `#[serde(untagged)]` shapes.
- Keep JSON Schema `type` as a string-or-array untagged union.
- Keep JSON Schema `additionalProperties` as a boolean-or-schema untagged union.
- Do not model whole tools or whole requests as `JsonObject`.
- Keep `LlmResponseItem` variants aligned with Rust `ResponseItem`.
- Keep `LlmResponseItem.McpToolCallOutput` aligned with Rust `ResponseInputItem::McpToolCallOutput`.
- Convert `LlmResponseItem.McpToolCallOutput` to `function_call_output` before sending Responses API requests.
- Keep MCP `CallToolResult` conversion aligned with Rust `CallToolResult::into_function_call_output_payload`.
- Keep `LlmAgentMessageInputContent` variants aligned with Rust `AgentMessageInputContent`: `input_text`, `encrypted_content`.
- Keep `LlmReasoningContentItem` variants aligned with Rust `ReasoningItemContent`: `reasoning_text`, `text`.
- Keep `LlmLocalShellCall.status` typed as `LlmLocalShellStatus`.
- Keep `LlmLocalShellCall.action` typed as `LlmLocalShellAction`.
- Keep `LlmWebSearchCall.action` typed as `LlmWebSearchAction`.
- Keep unknown `LlmResponseItem` types decoding to `LlmResponseItem.Other`.
- Keep unknown `LlmWebSearchAction` types decoding to `LlmWebSearchAction.Other`.
- Preserve the known Kotlin split between `compaction` and `compaction_summary`.
- Add serialization tests for every added protocol variant.
- Add decoding tests for every Rust `#[serde(other)]` fallback represented in Kotlin.
- Run `JAVA_HOME=/home/stream/.sdkman/candidates/java/17.0.19-tem ./gradlew :llm-provider:jvmTest`.
- Run `JAVA_HOME=/home/stream/.sdkman/candidates/java/17.0.19-tem ./gradlew :llm-provider:assemble`.

# OpenAI Model Alignment

Use this checklist when changing `CodexLite/openai/models` protocol models.

- Treat Rust `shared-context/codex/codex-rs/tools/src/tool_spec.rs` `ToolSpec` as the source of truth for `CodexLite/tool/contract` tool contracts.
- Name protocol DTOs after their Rust source models when there is a direct correspondence; do not add a generic `Llm` prefix to those DTOs.
- Keep Rust-to-Kotlin model correspondence details in KDoc on the corresponding model declarations.
- Keep tool parameter JSON Schema models on `kotlinx-schema-json`.
- Do not add or restore project-local `LlmJsonSchema` model classes.
- Use explicit `@SerialName` for wire names that are not Kotlin property names.
- Do not use a global JSON naming strategy.
- Use kotlinx polymorphic serialization for tagged protocol variants.
- Use small custom serializers only for Rust `#[serde(untagged)]` shapes.
- Do not model whole tools or whole requests as `JsonObject`.
- Add serialization tests for every added protocol variant.
- Add decoding tests for every Rust `#[serde(other)]` fallback represented in Kotlin.

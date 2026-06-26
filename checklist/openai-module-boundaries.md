# OpenAI Module Boundaries

Use this checklist when changing OpenAI API integration.

- Keep every `CodexLite/openai/*` module on the host target set with `codexlite.kmp-host`.
- Keep OpenAI wire DTOs and auth data types in `CodexLite/openai/models`.
- Keep OpenAI client interface shapes in `CodexLite/openai/client-contract`.
- Keep OpenAI Ktor clients, endpoint URLs, retry behavior, and SSE transport in `CodexLite/openai/client`.
- Keep `HttpClient` construction private to the concrete OpenAI client implementation; expose config objects rather than accepting external `HttpClient` instances.
- Make concrete OpenAI clients and provider adapters that own a client implement `AutoCloseable` and close owned clients.
- Keep `/responses` streaming-only in `OpenAiClient`; do not expose a non-streaming wrapper, consume SSE internally, synthesize `Response`, or map stream-level failures into fake JSON DTOs.
- Make `CodexLite/openai/client` depend on `CodexLite/openai/client-contract` with `api`, so downstream real-client users do not need to depend on the contract module separately.
- Keep downstream mock helpers and mock-client DSLs in `CodexLite/openai/client-test`.
- Keep host test platform helpers, such as environment variable access and Ktor test engine dependencies, outside OpenAI modules in `CodexLite/utils/host-test-support`.
- Keep Codex CLI local data directory access in `CodexLite/openai/codex-cli-storage`.
- Keep `CodexLite/llm-provider/api` as a provider adapter over `openai:client`.
- Keep tool modules focused on tool specs, arguments, and handler-facing behavior.
- Do not add OpenAI API DTOs or HTTP clients back into tool modules or `llm-provider`.

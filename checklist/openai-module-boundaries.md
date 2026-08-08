# OpenAI Module Boundaries

Use this checklist when changing OpenAI API integration.

- Keep every `Kodex/openai/*` module on the host target set with `kodex.kmp-host`.
- Keep OpenAI wire DTOs and auth data types in `Kodex/openai/models`.
- Keep OpenAI client interface shapes in `Kodex/openai/client-contract`.
- Keep OpenAI Ktor clients, endpoint URLs, retry behavior, and SSE transport in `Kodex/openai/client`.
- Keep bearer-authenticated Codex account usage and reset operations in `OpenAiClient`; aggregate their observable, account-isolated application state in `Kodex/openai/account-usage`, separate from authentication and persistent settings.
- Keep OAuth/PKCE login in `OpenAiLoginClient`, separate from the bearer-authenticated `OpenAiClient`; apply `CODEX_REFRESH_TOKEN_URL_OVERRIDE` only to refresh requests.
- Keep `HttpClient` construction private to the concrete OpenAI client implementation; expose config objects rather than accepting external `HttpClient` instances.
- Make concrete OpenAI clients and provider adapters that own a client implement `AutoCloseable` and close owned clients.
- Keep `/responses` streaming-only in `OpenAiClient`; do not expose a non-streaming wrapper, consume SSE internally, synthesize `Response`, or map stream-level failures into fake JSON DTOs.
- Make `Kodex/openai/client` depend on `Kodex/openai/client-contract` with `api`, so downstream real-client users do not need to depend on the contract module separately.
- Keep downstream mock helpers and mock-client DSLs in `Kodex/openai/client-test`.
- Keep host test platform helpers, such as environment variable access and Ktor test engine dependencies, outside OpenAI modules in `Kodex/utils/host-test-support`.
- Keep Codex CLI local settings and Session compatibility in `Kodex/openai/codex-cli-storage`; follow [Codex CLI Storage compatibility](codex-cli-storage.md).
- Keep model snapshots and slug resolution in `Kodex/openai/model-catalog`; it may read the CLI cache but must not own HTTP transport or modify that shared cache. Keep model-level context-window budget calculation in `Kodex/openai/models`, while the current AgentState snapshot projection belongs in `Kodex/agent-state/context-window`.
- Use `openai:client` directly; do not add a forwarding LLM-provider adapter over it.
- Keep tool modules focused on tool specs, arguments, and handler-facing behavior.
- Do not add OpenAI API DTOs or HTTP clients back into tool modules.

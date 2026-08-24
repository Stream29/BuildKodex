# OpenAI Module Boundaries

Use this checklist when changing OpenAI API integration.

- Keep every `Kodex/openai/*` module on the host target set with `kodex.kmp-host`.
- Keep OpenAI wire DTOs and auth data types, including `OpenAiAuthState`, in `Kodex/openai/models`.
- Keep `OpenAiAuthState.Unavailable` as the stable, text-free enum `NotLoaded`, `CredentialsNotFound`, `UnsupportedAuthMode`, `InvalidCredentials`, `CredentialSourceUnavailable`, or `UnexpectedFailure`; map it to context-specific text only at exception or presentation boundaries, and retain raw failures only in implementation logs.
- Keep OpenAI client interface shapes, including the read-only `OpenAiAuthStore`, in `Kodex/openai/client-contract`.
- Make OpenAI API consumers depend only on `OpenAiAuthStore`; they must not depend on application auth contracts or receive reload, login, persistence, or lifecycle capabilities.
- Keep auth-source selection, credential loading and refresh, login commands, persistence, and implementation lifecycle in `Kodex/app/shared/auth`; its `KodexAuthStore` extends `OpenAiAuthStore`.
- Keep OpenAI Ktor clients, endpoint URLs, retry behavior, and SSE transport in `Kodex/openai/client`.
- Keep the ordinary OpenAI request total timeout at 90 seconds; configure SSE requests with an explicit packet-idle socket timeout instead of a total request timeout.
- Give remote compaction at most two protocol/transport retries without a shared wall-clock budget across attempts; preserve external cancellation as cancellation.
- Keep bearer-authenticated Codex account usage and reset operations in `OpenAiClient`; aggregate their observable, account-isolated application state in `Kodex/openai/account-usage`, separate from authentication and persistent settings. Its unavailable state only means no account is available and must not copy authentication error text.
- Keep OAuth/PKCE login in `OpenAiLoginClient`, separate from the bearer-authenticated `OpenAiClient`; apply `CODEX_REFRESH_TOKEN_URL_OVERRIDE` only to refresh requests.
- Keep `HttpClient` construction private to the concrete OpenAI client implementation; expose config objects rather than accepting external `HttpClient` instances.
- Make concrete OpenAI clients and provider adapters that own a client implement `AutoCloseable` and close owned clients.
- Keep `/responses` streaming-only in `OpenAiClient`; do not expose a non-streaming wrapper, consume SSE internally, synthesize `Response`, or map stream-level failures into fake JSON DTOs.
- Make `Kodex/openai/client` depend on `Kodex/openai/client-contract` with `api`, so downstream real-client users do not need to depend on the contract module separately.
- Keep downstream mock helpers and mock-client DSLs in `Kodex/openai/client-test`.
- Keep host test platform helpers, such as environment variable access and Ktor test engine dependencies, outside OpenAI modules in `Kodex/utils/host-test-support`.
- Keep read-only Codex `auth.json` and explicit MCP import decoding in `Kodex/openai/codex-cli-storage`; do not put Hooks, general settings, model-cache, Agent-context, or Session compatibility there. Follow [Codex CLI Storage compatibility](codex-cli-storage.md).
- Keep model snapshots and slug resolution in `Kodex/openai/model-catalog`; initialize from the bundled catalog and refresh through `/models` without reading Codex CLI caches. Keep model-level context-window budget calculation in `Kodex/openai/models`, while the current AgentState snapshot projection belongs in `Kodex/agent-state/context-window`.
- Use `openai:client` directly; do not add a forwarding LLM-provider adapter over it.
- Keep tool modules focused on tool specs, arguments, and handler-facing behavior.
- Do not add OpenAI API DTOs or HTTP clients back into tool modules.

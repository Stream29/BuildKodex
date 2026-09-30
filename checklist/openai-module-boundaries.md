# OpenAI Module Boundaries

Use this checklist when changing OpenAI API integration.

- Keep every `Kodex/openai/*` module on the host target set with `kodex.kmp-host`.
- Keep OpenAI wire DTOs and auth data types, including `OpenAiAuthState`, in `Kodex/openai/spec/models`.
- Keep the shared configured `OpenAiJsonCodec` wire policy in `Kodex/openai/spec/json-codec`; do not duplicate its encoding options in an implementation module.
- Keep `OpenAiAuthState.Unavailable` as the stable, text-free enum `NotLoaded`, `CredentialsNotFound`, `UnsupportedAuthMode`, `InvalidCredentials`, `CredentialSourceUnavailable`, or `UnexpectedFailure`; map it to context-specific text only at exception or presentation boundaries, and retain raw failures only in implementation logs.
- Keep OpenAI client interface shapes, including the read-only `OpenAiAuthStore`, in `Kodex/openai/spec/client`.
- Make OpenAI API consumers depend only on `OpenAiAuthStore`; they must not depend on application auth contracts or receive reload, login, persistence, or lifecycle capabilities.
- Keep auth-source selection, credential loading and refresh, login commands, persistence, and implementation lifecycle in `Kodex/app/shared/auth`; its `KodexAuthStore` extends `OpenAiAuthStore`.
- Keep OpenAI Ktor clients, endpoint URLs, retry behavior, and SSE transport in `Kodex/openai/impl/client`.
- Persist Session turn-routing state according to [Codex Turn State](codex-turn-state.md); transport handling must not make an in-memory client cache its sole source of truth.
- Classify HTTP 429 as retryable even when wrapped in HTTP/SSE exceptions; use the existing HTTP backoff and Agent retry limits rather than sending transient rate limits directly to the unhandled-error hook.
- Keep the ordinary OpenAI request total timeout at 90 seconds; configure SSE requests with an explicit packet-idle socket timeout instead of a total request timeout.
- Give remote compaction at most two protocol/transport retries without a shared wall-clock budget across attempts; preserve external cancellation as cancellation.
- Keep bearer-authenticated Codex account usage and reset operations in `OpenAiClient`; define the observable, account-isolated state in `Kodex/openai/spec/account-usage` and aggregate it in `Kodex/openai/impl/account-usage`, separate from authentication and persistent settings. Its unavailable state only means no account is available and must not copy authentication error text.
- Keep OAuth/PKCE login in `OpenAiLoginClient`, separate from the bearer-authenticated `OpenAiClient`; apply `CODEX_REFRESH_TOKEN_URL_OVERRIDE` only to refresh requests.
- Make OAuth token exchange and refresh return `OpenAiLoginResult`: an HTTP error is a structured `OpenAiLoginError` value, not an `OpenAiLoginException`; callers must branch on the result before committing or publishing credentials. Transport, cancellation, and successful-response decoding failures remain exceptional.
- Keep `HttpClient` construction private to the concrete OpenAI client implementation; expose config objects rather than accepting external `HttpClient` instances.
- Make concrete OpenAI clients and provider adapters that own a client implement `AutoCloseable` and close owned clients.
- Keep `/responses` streaming-only in `OpenAiClient`; do not expose a non-streaming wrapper, consume SSE internally, synthesize `Response`, or map stream-level failures into fake JSON DTOs.
- Make the flattened operation arguments of `OpenAiClient.createResponse` the sole source of request-specific request data: the implementation projects model/input/options into the `ResponsesApiRequest` JSON protocol DTO and projects Codex identity into `client_metadata` and transport headers. Keep transport-only turn state out of `ResponsesApiRequest`; derive `x-codex-turn-state` directly from the operation argument. Keep response metadata separate from outgoing request metadata.
- Make `Kodex/openai/impl/client` depend on `Kodex/openai/spec/client` with `api`, so downstream real-client users do not need to depend on the contract module separately.
- Keep the mock-client builder behavior contract in `Kodex/openai/spec/client-test`; keep concrete mock helpers and the DSL factory in `Kodex/openai/impl/client-test`.
- Keep host test platform helpers, such as environment variable access and Ktor test engine dependencies, outside OpenAI modules in `Kodex/utils/host-test-support`.
- Keep read-only Codex `auth.json` and explicit MCP import contracts/DTOs in `Kodex/openai/spec/codex-cli-storage`, with file decoding in `Kodex/openai/impl/codex-cli-storage`; do not put Hooks, general settings, model-cache, Agent-context, or Session compatibility there. Follow [Codex CLI Storage compatibility](codex-cli-storage.md).
- Define model-catalog snapshots and slug-resolution behavior in `Kodex/openai/spec/model-catalog`; implement bundled initialization and `/models` refresh in `Kodex/openai/impl/model-catalog` without reading Codex CLI caches. Keep model-level context-window budget calculation in `Kodex/openai/spec/models`, while the current AgentState snapshot projection belongs in `Kodex/agent-state/context-window`.
- Use `openai/impl/client` directly at composition sites; do not add a forwarding LLM-provider adapter over it.
- Keep tool modules focused on tool specs, arguments, and handler-facing behavior.
- Do not add OpenAI API DTOs or HTTP clients back into tool modules.

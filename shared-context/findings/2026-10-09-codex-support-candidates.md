# Codex support candidates — 2026-10-09

## Coverage and evidence

- Retrieved: 2026-10-09 10:37 UTC. This is a selective research summary, not copied release notes or an exhaustive implementation audit.
- Official sources: Codex CLI releases [0.160.0](https://github.com/openai/codex/releases/tag/rust-v0.160.0), [0.161.0](https://github.com/openai/codex/releases/tag/rust-v0.161.0), and [0.162.0](https://github.com/openai/codex/releases/tag/rust-v0.162.0), plus the linked product/API documentation below.
- Latest stable release observed: 0.162.0, published October 8. The local `shared-context/codex/` reference is at `604061ce51`, dated October 3; it was inspected without fetching or changing the submodule.
- Kodex evidence is from the current working tree. Missing keyword matches are preliminary gap indicators, not proof of missing behavior.
- This research does not authorize implementation. The subsequent user selection is recorded below and in its discussion task.

## User-selected follow-up: faster steering

- User decision after this review: faster Steer is the only additional candidate to pursue; the separate Ultrafast task remains unchanged.
- User assessment: Kodex can already implement this straightforwardly; it had simply not been done. Treat it as an existing-runtime responsiveness improvement, not a complex upstream protocol adoption.
- [ChatGPT release notes](https://help.openai.com/en/articles/6825453-chatgpt-release-notes) announce faster steering on October 8.
- Kodex already delivers pending steer through its runtime decorator: [SteerRuntimeImpl, lines 17–27](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-runtime/impl/decorator/steer/src/commonMain/kotlin/io/github/stream29/kodex/agentruntime/decorator/steer/SteerRuntimeImpl.kt#L17).
- [Faster Steer discussion task](../../kanban/discussion/2026-10-09-faster-steer.md) records the scoped follow-up; implementation has not started.

## Other researched candidates — not selected

### Model retirement and availability

- The [official changelog](https://learn.chatgpt.com/docs/changelog) announces GPT-5.5 retirement on October 14 from ChatGPT-authenticated Codex and Work; the API is excluded.
- Kodex still bundles `gpt-5.5`, while GPT-6 Astra, GPT-6.1 Sol, GPT-6 Sol, and GPT-6 Luna are already present. Evidence: [built-in catalog, lines 45–80](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/impl/model-catalog/src/commonMain/kotlin/io/github/stream29/kodex/openai/modelcatalog/BuiltInModelCatalog.kt#L45) and [GPT-5.5 entry, line 117](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/impl/model-catalog/src/commonMain/kotlin/io/github/stream29/kodex/openai/modelcatalog/BuiltInModelCatalog.kt#L117).
- Proposed scope: authentication-specific availability and clear replacement guidance for saved defaults/Sessions, not silently rewriting user choices or removing API-supported models.

### Responses WebSocket incremental continuation

- The [official WebSocket guide](https://developers.openai.com/api/docs/guides/websocket-mode) documents persistent connections, `previous_response_id`, incremental input, and multiplexed streams. It reports up to roughly 40% lower end-to-end time for some workflows with 20+ tool calls; this is not a Kodex performance guarantee.
- Kodex's inspected Responses path uses HTTP/SSE: [client, line 241](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/impl/client/src/commonMain/kotlin/io/github/stream29/kodex/openai/client/OpenAiClient.kt#L241).
- Proposed scope: start with a single conversation's continuation, reconnection, cancellation, compaction/revert boundaries, and HTTP fallback. Evaluate multiplexing only after lifecycle correctness.
- This overlaps the existing [codex-rs alignment discussion](../../kanban/discussion/2026-09-30-align-kodex-with-codex-rs.md).

### Server-directed retry behavior

- Releases 0.161.0 and 0.162.0 add server retry guidance and `Retry-After` handling for Responses/transport failures.
- Kodex already has HTTP retry with exponential delay: [client, lines 95–124](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/impl/client/src/commonMain/kotlin/io/github/stream29/kodex/openai/client/OpenAiClient.kt#L95). Do not describe this as adding retries from scratch; HTTP-library header behavior also needs verification.
- Proposed scope: structured stream-failure guidance, shared retry budgets, cancellation during backoff, and prevention of duplicate tools/messages after partial output.
- This is also covered at a high level by the existing codex-rs alignment discussion.

### Managed Git worktrees

- Release 0.162.0 exposes worktree creation/listing tools for trusted projects behind a feature gate.
- No worktree-specific matches were found in the inspected Kodex app, agent-session, and OpenAI Kotlin sources.
- Proposed scope: optional isolated checkouts for parallel Sessions, explicit starting branch/local-change choices, visible checkout identity, and user-controlled cleanup. Never delete unmerged user changes automatically.

### Luna Reserve

- The [official Luna Reserve guide](https://help.openai.com/en/articles/20001499-luna-reserve-in-codex-and-chatgpt-work) describes a separate fallback allowance for selected personal Plus/Pro accounts after ordinary usage is exhausted. It is not API credit or guaranteed account access.
- Kodex already models generic additional rate limits: [usage models, lines 9–17 and 45–53](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/spec/models/src/commonMain/kotlin/io/github/stream29/kodex/openai/CodexAccountUsageModels.kt#L9). Reserve-specific detection/routing was not established by this inspection.
- Proposed scope: first establish backend eligibility and allowance signals, then show the reserve separately and offer an explicit continuation choice. Do not infer access from a model name or silently downgrade.

## Other observations — not selected or already tracked

- Code Mode's streaming promise helpers and ranked tool search are new in 0.162.0, but Kodex already has Tool Search and a [Tool as CLI / Code Mode comparison task](../../kanban/discussion/2026-10-03-discuss-integrate-tool-cli-into-kodex.md). Continue that discussion rather than duplicate it.
- CRLF-preserving edits in 0.162.0 merit a Windows regression check. Kodex's existing patch-parser CRLF test concerns parsing, not proof of preserving target-file line endings.
- Daybreak requires separately approved cyber access; Voice, desktop-only features, and cloud remote-control integration are not proposed as immediate CLI priorities.
- [Ultrafast implementation and validation are tracked separately](../../kanban/executable/2026-10-09-support-openai-ultrafast.md).

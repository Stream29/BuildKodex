# Task Tree

- `Inventory remaining OpenAI APIs, KDoc, consumers, and dependency edges`()
- `Resolve mixed module boundaries from code`()
- `Move model and codec fact sources without changing wire formats`()
- `Migrate client contracts and concrete transport`()
- `Split Codex CLI storage and client test support`()
- `Retarget consumers and update OpenAI checklists`()
- `Validate dependency graph, compatibility, and scoped supported targets`()
- `Archive delivered OpenAI migration with later acceptance and remaining limits`()

# Details

## Reconciled status — 2026-10-07

- **DONE — OpenAI 实际契约迁移已交付**；不是只按目录位置或早期 JVM 结果归档。
- 后续 [Runtime/OpenAI 终验](2026-10-07-final-audit-runtime-openai.md)追踪
  真实 DTO、client、storage、mock 到生产消费者；发现的 mock/取消等确认问题经
  [限定收尾](2026-10-07-plan-spec-impl-closure-and-consistency.md#final-scoped-closure-evidence)
  修复、集中验证及独立复审。其后 [核心 CLI E2E](2026-10-07-accept-sealed-cli-core-e2e.md)
  与 [v0.4.10 发布](2026-10-07-release-kodex-0-4-10.md#publication-verified--v0410)
  交付实际 main `83749434`。
- 下文是原迁移 checkpoint：当时未运行的平台保持历史事实。account A→B→A、
  live OAuth/refresh、全平台故障矩阵和 resolved kRPC 图并未整体通过；
  当前剩余门槛归后续收尾及 [迁移总任务](../executable/2026-09-28-plan-domain-spec-impl-boundaries.md)，
  不再暂停这个已交付的 OpenAI 子批次。

## Scope and current state

- Child implementation task of [Domain Spec/Impl Boundaries](../executable/2026-09-28-plan-domain-spec-impl-boundaries.md). The user authorized code migration on `Kodex/` branch `refactor/spec`; keep changes in reviewable slices and do not commit or push without a separate request.
- Previously migrated: `openai/{spec,impl}/account-usage` and `openai/{spec,impl}/model-catalog`. Their relocated JVM tests passed during this domain's validation.
- The six remaining legacy projects have now been moved under `openai/spec` or `openai/impl`. Kotlin package names and serialized names remain unchanged; the client-test builder and Codex CLI storage now expose interfaces with same-name implementation-side factories.
- Follow [Spec/Impl Module Boundaries](../../checklist/spec-impl-module-boundaries.md): spec code **and attached KDoc** are normative, including observable `@throws` conditions; implementation KDoc may explain concrete mechanisms. Each retained impl project needs a meaningful corresponding spec, not a placeholder.

## Implemented ownership

| Existing project | Destination | Boundary decision |
| --- | --- | --- |
| `:openai-models` | `openai/spec/models` | Wire DTOs, auth types, attached serializers, and pure context-budget calculations remain together as one spec-only project. |
| `:openai-client-contract` and `:openai-client` | `openai/spec/client` and `openai/impl/client` | Client/auth/login interfaces, behavior KDoc, and observable remote-compaction exceptions are spec-owned; transport and construction config remain impl-owned. |
| `:openai-json-codec` | `openai/spec/json-codec` | The public configured `Json` value is the shared wire policy; no artificial impl project is created. |
| `:openai-codex-cli-storage` | `openai/spec/codex-cli-storage` and `openai/impl/codex-cli-storage` | Read-only operations and wire DTO/serializer hierarchy are spec-owned; filesystem/TOML reading and the same-name construction factory are impl-owned. |
| `:openai-client-test` | `openai/spec/client-test` and `openai/impl/client-test` | The mock-builder behavior interface is spec-owned; concrete test helpers and factories are impl-owned. |

Account-usage and model-catalog remain separate pairs rather than being merged into a monolithic OpenAI spec. The new spec-only projects contain protocol facts rather than artificial impl pairings.

## Current implementation and outstanding validation

- `spec/models` owns the old protocol models, pure budget functions, serializer code attached to DTO annotations, and their tests. `spec/json-codec` owns the single configured public `Json` value and its new contract KDoc; neither is a dummy impl.
- `spec/client` owns the existing interfaces plus observable remote-compaction exception types and expanded behavior/`@throws` KDoc. `impl/client` retains HTTP/SSE transport, concrete configuration, construction, and tests.
- OpenAI OAuth exchange and refresh now return the existing generic union shape through `OpenAiLoginResult<Success>` with a structured `OpenAiLoginError` failure branch. The removed `OpenAiLoginException` is not part of the spec; auth/RPC composition branches explicitly before committing credentials, while transport, cancellation, and successful-response decoding failures may still propagate.
- `spec/codex-cli-storage` owns a read-only `CodexCliStorage` interface and the original wire DTO/serializer hierarchy. `impl/codex-cli-storage` owns `CodexCliStorageImpl`, a same-name factory preserving construction calls, filesystem/TOML reading, and tests.
- `spec/client-test` owns the mock-builder behavior interface. `impl/client-test` owns its builder implementation, same-name builder factory, mock client, in-memory auth store, and tests.
- All direct Gradle project references to the six retired OpenAI paths have been replaced; affected current checklists have been updated. Static checks found 7 spec and 5 impl OpenAI projects, no direct spec-to-impl edges or stale direct references, and passed whitespace checks.
- JVM main-source compilation passed for the five new specs, the three new impls, and the account-usage/model-catalog impls. JVM tests passed for `spec/models`, `impl/client-test`, `impl/codex-cli-storage`, `impl/client`, `impl/account-usage`, and `impl/model-catalog`.
- The OAuth follow-up replaces `OpenAiLoginException` with `OpenAiLoginResult`; the login client, auth stores, backend OAuth composition, test doubles, and a provider-error test now branch on the union. OpenAI client JVM tests, auth filesystem JVM tests, and RPC server JVM tests passed after this change.
- The Responses A方案 now keeps `ResponsesApiRequest` as a pure `/responses` JSON protocol DTO, not a `createResponse` parameter. The complete flattened `OpenAiClient.createResponse` parameter list is the operation-level source of truth; the client implementation projects it into the DTO, `client_metadata`, and request headers. `turnState` remains an operation-level transport argument and supplies `x-codex-turn-state` without entering the JSON body. Remote compaction still accepts the protocol DTO and preserves its existing no-turn-state and beta-header behavior; `onResponseHeaders` remains response metadata only.
- `app/migration/impl` uses `kodex.kmp-cli`, not `kodex.kmp-host`, because it is a CLI-only module and its common tests depend on the CLI-only settings filesystem project, which has no JS variant.
- After switching the migration module to the conventional CLI plugin, `:app-migration-impl:jvmTest` resolves the common-test dependency graph without the unresolved JS-platform error. The selected JVM test set also passes: OpenAI models/client/client-test, AgentState, compact runtime, in-memory and filesystem sessions, and app migration.
- Downstream JVM compilation passed for the auth filesystem adapter, Runtime composition, RPC server, application ViewModel, and integration-test test sources; auth filesystem and RPC server JVM tests passed. Native compilation, actual live integration tests, and CLI smoke were not run. Do not claim platform-wide or repository-wide closure from these JVM checks.
- The follow-up flattened `createResponse` migration also removes transport-only `turnState` from `ResponsesApiRequest`; OpenAI specs, client implementation, client-test implementation, AgentState, session-title, and integration wrappers compile on JVM. The focused JVM regression suite passed after this final API adjustment, including the projection and partial-Codex-identity checks.

## Migration order and stop conditions

1. Inventory public declarations and KDoc, `api` versus `implementation` edges, common/test source sets, and all direct consumers of the six legacy projects.
2. Resolve serializer ownership, codec placement, client config/exception ownership, Codex CLI DTO coupling, and a nonempty client-test spec before moving mixed projects.
3. Migrate model and codec facts, then client, Codex CLI storage, and test support; retarget direct consumers and compile each slice.
4. Update [OpenAI Module Boundaries](../../checklist/openai-module-boundaries.md), [Codex CLI Storage compatibility](../../checklist/codex-cli-storage.md), and [Model Catalog](../../checklist/model-catalog.md) as each new path becomes true.
5. Stop for review if the move would change public signatures, serialization bytes/names, auth ownership, SSE behavior, or a frozen RPC contract. No duplicate DTO hierarchy or spec-to-impl dependency is an acceptable shortcut.

## Validation gates

- Static: all OpenAI Gradle projects are classified as spec or impl, each retained impl maps to a meaningful spec, no migrated spec has a direct main-source impl dependency or spec cycle, and no direct consumer still references a removed path. Track dependencies on still-mixed projects outside OpenAI rather than claiming repository-wide closure.
- Contract: compare moved declarations and KDoc, including `@throws`, cancellation, authentication isolation, resource closure, codec options, and response-stream failure semantics. Check Kotlin package names, `api` exposure, and host KMP target sets.
- Compatibility: decode and re-encode existing OpenAI JSON and Codex CLI `auth.json`/MCP TOML fixtures; keep field names, polymorphic shapes, unknown-value handling, and import diagnostics. Preserve client request headers, retry limits, streaming-only Responses, and ownership rules.
- Build and tests: compile migrated spec/impl projects and downstream Agent/app/test consumers; run OpenAI client, model/codec, Codex CLI storage, account-usage, model-catalog, and test-support JVM tests plus supported Native checks when resources permit. Record any unrun checks explicitly; reuse an idle shared Gradle Daemon rather than starting a competing one.

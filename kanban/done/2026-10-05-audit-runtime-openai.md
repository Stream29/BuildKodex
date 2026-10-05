# Task Tree

- `Trace Runtime and OpenAI contracts through actual composition and transport`() // completed
- `Review semantics, tests and ablation candidates`() // completed
- `Publish evidence and unreviewed coverage`() // completed

# Details

- Parent: [Spec/impl reacceptance](../executable/2026-10-05-reaccept-spec-impl-refactor.md).
- Own review scope: complete `agent-runtime` and `openai` roots.
- Report only in this file; source, other tasks and shared resources are read-only.
- Apply parent acceptance rules and pinned baseline; no independent fixes or builds.
- Status: **REVIEW READY**; independent static review complete. HEAD verified as
  `821b66275aae2f0a9541fa2e9e03023d875c30b5`, inner tracked worktree clean.

## Review report

### Result and limits

- Complete topic coverage: 10 Runtime projects and 12 OpenAI projects, including
  production Kotlin in these roots, their Gradle edges, relevant history,
  concrete downstream composition and focused tests.
- **No confirmed disconnected production contract, parallel production DTO
  authority, or forwarding LLM-provider wrapper in these two roots.** This is
  not a claim that the rest of the repository has no such problems.
- **One B1:** mock-builder independence is violated. It affects test support,
  not the real HTTP client. **Two B2:** cancellation-cleanup failure propagation
  and unresolved default-compaction retry policy. Historical documentation and
  mixed-root debt are listed separately; there is no unconditional acceptance.
- The earlier intermediate checkpoint is superseded by this report.
- Loaded outer AGENTS, BuildKodex/change, checklist, kanban, planning, ask-user,
  document, workspace and code-collaboration skills; parent/child tasks and
  relevant boundary, Runtime, Agent State, mutation, coroutine ownership, RPC,
  OpenAI, model catalog/alignment, Curl, Codex storage, turn-state and Hook
  checklists. Read original migration task records rather than inheriting
  their “passed” conclusions.
- Read-only source/history inspection only. No Gradle, native build, network,
  endpoint, browser, IDE, commit, branch, push or source edit. Pre-existing build
  artifacts are not evidence of tests run by this audit.

### Coverage classification

`R` means a traced static path, not a runtime/platform certificate. Findings
below qualify each applicable row. No scoped production topic remains `U`.

| Topic | Coverage | Actual integration |
| --- | --- | --- |
| Runtime core/composition | R; B2-1; D-3 | `AgentRuntimeImpl : AgentRuntime`; filesystem/in-memory Sessions; backend-owned RPC operations |
| Runtime compact | R; D-1 inherited state docs | `KodexAgentCompactionRuntimeImpl : KodexAgentCompactionRuntime`; first layer in master composition |
| Runtime steer | R; D-1 | `SteerRuntimeImpl : SteerRuntime`; provider claims actual pending queue atomically |
| Runtime tool | R | `KodexToolRuntimeImpl : KodexToolRuntime`; real fixed/MCP/search handlers and hooks |
| Runtime turn-hook | R | `TurnHookRuntimeImpl : TurnHookRuntime`; real outer layer; CLI deliberately supplies `NoOpKodexHooks` |
| OpenAI models | R; D-2 | Existing wire/persisted DTOs, unions, custom serializers and pure model budget function |
| OpenAI json-codec | R | Shared configured `Json` used by real transport and external Codex decoding |
| OpenAI client: Responses, compaction, images, search, usage | R; B2-2 | Concrete aliased-contract implementation; complete request projection; real consumers |
| OpenAI client: auth/login/headers/exceptions | R | Read-only auth store, separate owned login client, union errors consumed before credential commit |
| OpenAI account usage | R | Actual account-isolated store plus backend composition and RPC summary projection |
| OpenAI model catalog | R | Actual store, bundled/remote catalog and shared context-budget calculation |
| OpenAI Codex CLI storage | R | Actual file decoder; explicit auth/MCP callers; no compatibility catch-all |
| OpenAI client-test | B1-1; otherwise R | Actual builder/mock implement specs; fixed auth helper implements real auth contract |

### History and dependency baseline

- Compared declarations before `f1a9000603083fed7c2cfbcf3a7a3a16ebe8118c`
  with that split, consumer retarget
  `5d43da3805836c0cb6623c1c0ea58ca455d5f61e`, and pinned HEAD.
- `f1a90006` creates real decorator operation interfaces and moves routing/hook
  projection helpers into impl. Core composition, account aggregation and many
  protocol/storage models are exact renames, not newly added forwarding shells.
- Flattened Responses signature, DTO projection ownership, compaction metadata
  source and login error union are intentional changes recorded in the
  [OpenAI migration task](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/executable/2026-09-29-complete-openai-spec-impl-split.md#L1)
  and [boundary checklist](file:///home/stream/ACodeSpace/push/BuildKodex/checklist/openai-module-boundaries.md#L19).
  `5d43da38` updates actual Agent State, title, auth and RPC consumers; it is
  not just a Gradle dependency adjustment.
- `453e9bf2c7474d80af4b06162140d8dd7167128f` refreshes bundled catalog data.
  Later `6606efe9`, `a6959173`, `d05e9cdb`, `b4cb23e2` retarget downstream
  module edges; no additional semantic source change in these roots was found
  there. Merge/version history is not external-protocol verification.
- Inspected all 22 scoped build scripts: no direct scoped production spec → impl
  edge. OpenAI models/json-codec legitimately have no impl peer. Runtime's
  dependence on still-unrelocated `agent-state-contract` is D-3, not proof of
  an implementation bypass. Genuine impl → impl composition is permitted.

## Runtime evidence

### Core operation authority, inheritance and Session/RPC use

- FQCN `io.github.stream29.kodex.agentruntime.contract.AgentRuntime`
  [extends `ResumableAgentLayer`](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-runtime/spec/contract/src/commonMain/kotlin/io/github/stream29/kodex/agentruntime/contract/AgentRuntime.kt#L36);
  that layer extends `KodexAgentState`. The contract therefore includes actual
  state mutations, resume, compact, pending steer, running Job and shared exec,
  not just helper policy values.
- FQCN `io.github.stream29.kodex.agentruntime.impl.AgentRuntimeImpl` is the
  [private implementing class](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-runtime/impl/composition/src/commonMain/kotlin/io/github/stream29/kodex/agentruntime/impl/KodexAgentRuntimeComposition.kt#L144).
  `KodexAgentState by delegate` preserves the same state boundary.
  `buildMasterAgentRuntime` returns the contract; no second production runner
  declaration replaces it.
- Real caller:
  [filesystem Session](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-session/impl/filesystem/src/commonMain/kotlin/io/github/stream29/kodex/agentsession/filesystem/FileSystemKodexAgentSession.kt#L46)
  constructs actual state and calls `buildMasterAgentRuntime` at line 55.
  In-memory repository also uses this composition. Backend services supply
  [real client/catalog/context/MCP dependencies](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/server/src/commonMain/kotlin/io/github/stream29/kodex/rpc/server/BackendServices.kt#L64).
- Resume and manual compact both enter
  [one CAS-protected running slot](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-runtime/impl/composition/src/commonMain/kotlin/io/github/stream29/kodex/agentruntime/impl/KodexAgentRuntimeComposition.kt#L154).
  Published value is `currentCoroutineContext().job`, not an independently
  launched worker or presentation copy. Failed CAS throws the spec exception;
  `finally` at line 189 releases after body cleanup. Manual compaction delegates
  without clearing steer, running tools or auto-resuming.
- Production RPC:
  [resume → `session.agentRuntime.resume()`](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/server/src/commonMain/kotlin/io/github/stream29/kodex/rpc/server/BackendAgentRuntimeRpc.kt#L104);
  forced compact at line 131 uses the inherited runtime operation; cancel at
  lines 184–189 cancels that published Job. Exec observation/closure at
  lines 163–183 uses the same `unifiedExecToolClient`.
- [BackendSessionHost.inSession](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/server/src/commonMain/kotlin/io/github/stream29/kodex/rpc/server/BackendSessionHost.kt#L121)
  starts the operation under backend `binding.operations.async` and awaits it.
  RPC-waiter cancellation is not ownership of the accepted turn. The running
  observer at lines 201–217 follows cancelling cleanup too.
  No extra running-state authority was found.
- Composition is an exact rename in `f1a90006`; decorator interface integration
  is new in that commit. B2-1 concerns an inherited conditional cleanup path,
  not new launch/ownership semantics.

### All four decorators are operational contracts

- **Compact:** FQCN
  `io.github.stream29.kodex.agentruntime.decorator.compact.KodexAgentCompactionRuntime`
  [spec operations/throws](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-runtime/spec/decorator/compact/src/commonMain/kotlin/io/github/stream29/kodex/agentruntime/decorator/compact/KodexAgentCompactionRuntime.kt#L16)
  are implemented by same-package `KodexAgentCompactionRuntimeImpl`
  [at line 25](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-runtime/impl/decorator/compact/src/commonMain/kotlin/io/github/stream29/kodex/agentruntime/decorator/compact/KodexAgentCompactionRuntimeImpl.kt#L25).
  Resume returns immediately for existing/new pending tools, samples budgets,
  applies pre/mid compaction, resets retry count on continuation and throws
  `AgentResponseRetryLimitExceededException` on exhaustion. It calls the state
  request operation, not a parallel response collector. Compact at line 89
  calls Pre → delegate compact → Post; Post is not run after failed core call.
  Factory at line 155 returns the spec. Impl owns numeric 20-retry Agent policy;
  spec promises exhaustion behavior, not a contradictory numeric limit.
- **Steer:** FQCN
  `io.github.stream29.kodex.agentruntime.decorator.steer.SteerRuntime` and
  `SteerProvider`
  [define operational resume and exclusive ordered claim](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-runtime/spec/decorator/steer/src/commonMain/kotlin/io/github/stream29/kodex/agentruntime/decorator/steer/SteerProvider.kt#L14).
  Same-package `SteerRuntimeImpl`
  [implements/delegates state](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-runtime/impl/decorator/steer/src/commonMain/kotlin/io/github/stream29/kodex/agentruntime/decorator/steer/SteerRuntimeImpl.kt#L12):
  eligibility is checked before take, then ordered items enter actual
  injectHistory; before-first and later-loop checks are present.
  Production composition
  [claims with `pendingSteer.getAndUpdate { emptyList() }`](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-runtime/impl/composition/src/commonMain/kotlin/io/github/stream29/kodex/agentruntime/impl/KodexAgentRuntimeComposition.kt#L84).
  RPC [queue compareAndSet](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/server/src/commonMain/kotlin/io/github/stream29/kodex/rpc/server/BackendAgentRuntimeRpc.kt#L191)
  targets that same queue. Claim ownership is linearizable, not a promise of
  durable requeue after claim followed by failed/cancelled persistence.
- **Tool:** FQCN
  `io.github.stream29.kodex.agentruntime.decorator.tool.KodexToolRuntime`
  [owns complete resume behavior](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-runtime/spec/decorator/tool/src/commonMain/kotlin/io/github/stream29/kodex/agentruntime/decorator/tool/KodexToolRuntime.kt#L16).
  Same-package `KodexToolRuntimeImpl`
  [implements it](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-runtime/impl/decorator/tool/src/commonMain/kotlin/io/github/stream29/kodex/agentruntime/decorator/tool/KodexToolRuntimeImpl.kt#L30).
  Existing pending calls are handled before model resume. Catalog sampling and
  collisions pass through
  [ToolRouting](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-runtime/impl/decorator/tool/src/commonMain/kotlin/io/github/stream29/kodex/agentruntime/decorator/tool/ToolRouting.kt#L18),
  not an unused routing-policy spec. Invalid arguments/search bypass ordinary
  hooks; removed MCP calls get explicit failure; unowned host calls stay pending.
  At lines 111–131, Pre/block/handler/Post run in order and completion is written
  only if still pending, preserving handlers such as plan update that complete
  themselves. Factory at line 171 returns the spec.
- **Turn-hook:** FQCN
  `io.github.stream29.kodex.agentruntime.decorator.turnhook.TurnHookRuntime`
  [extends the real layer](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-runtime/spec/decorator/turn-hook/src/commonMain/kotlin/io/github/stream29/kodex/agentruntime/decorator/turnhook/TurnHookRuntime.kt#L12).
  Same-package `TurnHookRuntimeImpl`
  [implements it](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-runtime/impl/decorator/turn-hook/src/commonMain/kotlin/io/github/stream29/kodex/agentruntime/decorator/turnhook/TurnHookRuntimeImpl.kt#L28).
  Persisted trailing user context drives UserPromptSubmit. Stop/additional
  context flows are persisted; after delegate resume the layer samples actual
  assistant/host-owned results, invokes Stop and fails host interactions before
  continuation injection. Fragment aggregation/escaping belongs to impl
  `TurnHookProjection`, not a second hook contract.
- Interfaces and factories are introduced/retargeted in `f1a90006`.
  Their common actual caller is
  [complete composition](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-runtime/impl/composition/src/commonMain/kotlin/io/github/stream29/kodex/agentruntime/impl/KodexAgentRuntimeComposition.kt#L79):
  state → compact → steer → tool → turn-hook. Outer turn-hook invocation wraps
  tool handling, steering and request/compaction behavior. Merely finding
  individual helper usage was not used as acceptance evidence.

### Borrowed resources and hooks

- Tool/decorator specs borrow handlers/hooks/catalog; they do not close shared
  OpenAI client or MCP service.
  [KodexAgentTools](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-runtime/impl/composition/src/commonMain/kotlin/io/github/stream29/kodex/agentruntime/impl/KodexAgentTools.kt#L39)
  builds fixed tools and a single unified-exec client. Current settings are
  obtained by suspend providers rather than frozen at runtime construction.
- Composition registers
  [reverse fixed-tool closure on Session Job](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-runtime/impl/composition/src/commonMain/kotlin/io/github/stream29/kodex/agentruntime/impl/KodexAgentRuntimeComposition.kt#L99).
  Construction failure preserves its original Throwable and suppresses close
  failures at lines 223–237. Normal termination logs tool close failures.
  Unified-exec tools close shared client; its
  [close delegates to owned shell](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/unified-exec/src/commonMain/kotlin/io/github/stream29/kodex/tool/unifiedexec/UnifiedExecToolClient.kt#L160).
  This establishes Runtime instance/owner wiring, not certification of native
  shell internals; those belong to Tool/utility audit.
- Current CLI intentionally provides `NoOpKodexHooks` in BackendServices line 66.
  [Hook policy](file:///home/stream/ACodeSpace/push/BuildKodex/checklist/hooks.md#L1)
  does not authorize restoring old CLI control hooks. No-op production hooks
  do not make decorator operation contracts dummy implementations.

## OpenAI evidence

### Models and codec are legitimate pure contracts

- Read all 20 production model files and codec source. Existing persisted
  settings/compaction, response-item/tool unions, auth, catalog, usage, image,
  search, plan and metadata types remain real models, mostly unchanged renames
  in `f1a90006`. Notable new login union is consumed by real callers below.
- FQCN `io.github.stream29.kodex.openai.ResponsesApiRequest`
  [is the protocol DTO](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/spec/models/src/commonMain/kotlin/io/github/stream29/kodex/openai/ResponsesApiModels.kt#L31);
  client creates and serializes it. It is not an extra forwarding request
  authority. Transport-only turn state is absent from it.
- FQCN `io.github.stream29.kodex.openai.ResponseItem` and its
  [serializable/type-membership interfaces](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/spec/models/src/commonMain/kotlin/io/github/stream29/kodex/openai/ResponseItemModels.kt#L8)
  (`Known`, `HistoryItem`, `Steerable`, `ToolCall`, `ToolCallOutput`) are
  intentional old model interfaces. Same for `LoadableToolSpec`,
  `ResponsesApiNamespaceTool`, `ReasoningEffort.Known` and external
  `CodexCliMcpServer`. Serializer/type roles were checked, not rejected by name
  or lack of direct constructor.
- [OpenAiJson serializers](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/spec/models/src/commonMain/kotlin/io/github/stream29/kodex/openai/OpenAiJson.kt#L20)
  implement response-item/event discrimination, unknown effort preservation,
  function-output payloads and search calls. Annotation/serializer paths are
  actual uses even where current UI never constructs every variant.
- FQCN `io.github.stream29.kodex.openai.jsoncodec.OpenAiJsonCodec`
  [provides shared configured `Json`](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/spec/json-codec/src/commonMain/kotlin/io/github/stream29/kodex/openai/jsoncodec/OpenAiJsonCodec.kt#L12).
  Transport content negotiation, SSE event decoding and Codex storage use it.
  No duplicated implementation codec or artificial impl peer was found.
- [Model context budget](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/spec/models/src/commonMain/kotlin/io/github/stream29/kodex/openai/ModelCatalogModels.kt#L149)
  is a pure contract function shared with Agent State/runtime budget projection.
  FunctionCallOutputModels similarly supplies actual model/tool conversion;
  neither needs an implementing class merely to satisfy a pattern.

### Complete client operation and real HTTP/SSE transport

- FQCN `io.github.stream29.kodex.openai.client.contract.OpenAiClient`
  [defines all operations](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/spec/client/src/commonMain/kotlin/io/github/stream29/kodex/openai/client/contract/OpenAiClient.kt#L41).
  FQCN `io.github.stream29.kodex.openai.client.OpenAiClient`
  [imports contract under alias and implements it](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/impl/client/src/commonMain/kotlin/io/github/stream29/kodex/openai/client/OpenAiClient.kt#L90).
  This is the actual owned Ktor client constructed in BackendServices line 41,
  not an old client hiding behind a new helper-policy declaration.
- Flattened createResponse:
  [spec signature](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/spec/client/src/commonMain/kotlin/io/github/stream29/kodex/openai/client/contract/OpenAiClient.kt#L139)
  → [impl projection](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/impl/client/src/commonMain/kotlin/io/github/stream29/kodex/openai/client/OpenAiClient.kt#L196)
  maps model/input/instructions/store/previous ID/tools/choice/parallel/reasoning/
  include/tier/cache/text and installation/session/thread/turn/window identity.
  Cache defaults to thread only when null; explicit empty cache key survives.
  Identity helper at lines 690–729 validates thread/window together and creates
  protocol metadata and serialized turn metadata. No stale out-of-band request
  DTO authority remains.
- Actual submission at lines 241–255 authenticates from borrowed read-only auth
  state, emits installation/turn/window headers, derives `x-codex-turn-state`
  from operation argument and sends projected DTO.
  [Header callback and event decode](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/impl/client/src/commonMain/kotlin/io/github/stream29/kodex/openai/client/OpenAiClient.kt#L370)
  precede event emission; response request-ID headers are separate from outgoing
  metadata. Retryable stream termination does not synthesize Completed.
- Production consumer FQCN `io.github.stream29.kodex.agentstate.impl.KodexAgentStateImpl`
  [supplies full flattened request](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-state/impl/src/commonMain/kotlin/io/github/stream29/kodex/agentstate/impl/KodexAgentStateImpl.kt#L189),
  persists received turn state through actual writer, collects real events,
  commits completed output, returns retryable disposition on Failed and throws
  on Incomplete. State restoration is in finally.
  Metadata-free
  [session-title caller](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/shared/session-title/src/commonMain/kotlin/io/github/stream29/kodex/cli/sessiontitle/OpenAiSessionTitleGenerator.kt#L31)
  also calls this sole signature and rejects failed/incomplete/open endings.
- SSE lifetime follows
  [postSseEventsInternal](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/ktor-client-ext/spec/src/commonMain/kotlin/io/github/stream29/kodex/utils/ktorclientext/SseRequests.kt#L52):
  channelFlow → scoped Ktor SSE session → header callback → incoming collection.
  No client-global detached stream scope or returned live session handle found.
  Actual engine idle/timeout/cancellation behavior remains unexecuted.
- Compaction:
  [client operation](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/impl/client/src/commonMain/kotlin/io/github/stream29/kodex/openai/client/OpenAiClient.kt#L259)
  takes DTO metadata as sole request identity, adds owned beta header and
  consumes SSE internally. Parser at lines 538–595 requires exactly one
  compaction before Completed; missing terminal, failed/incomplete and invalid
  ordering use actual spec-owned exceptions. No fake JSON Response.
  Agent State [constructs/passes actual DTO](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-state/impl/src/commonMain/kotlin/io/github/stream29/kodex/agentstate/impl/KodexAgentStateImpl.kt#L282).
- Other operations are implemented: models, bearer account usage/reset/profile
  at lines 146–194; image generation/edit at 282–306; search at 308.
  Actual tool callers are
  [ImageGenerationToolClient](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/image-generation/src/commonMain/kotlin/io/github/stream29/kodex/tool/imagegeneration/ImageGenerationToolClient.kt#L20)
  and
  [WebRunToolClient](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/web-run/src/commonMain/kotlin/io/github/stream29/kodex/tool/webrun/WebRunToolClient.kt#L14).
  They adapt handler arguments/results, not newly invented forwarding
  LLM-provider layers.
- JSON operations at lines 597–644 retain structured success/error unions and
  status. Expected-account validation at lines 329–345 occurs at submission.
  Real client close at line 322 closes owned HTTP, not borrowed auth.
  BackendServices lines 89–95 joins owners before closing API/login clients.

### Login and union-error consumers

- FQCN `io.github.stream29.kodex.openai.client.contract.OpenAiLoginClient`
  [returns OpenAiLoginResult](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/spec/client/src/commonMain/kotlin/io/github/stream29/kodex/openai/client/contract/OpenAiLoginClient.kt#L29).
  Factory FQCN `io.github.stream29.kodex.openai.client.OpenAiLoginClient`
  [returns contract and constructs private OpenAiLoginClientImpl](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/impl/client/src/commonMain/kotlin/io/github/stream29/kodex/openai/client/OpenAiLoginClient.kt#L35).
  Internal engine injection is a test seam, not public borrowed HttpClient.
- Exchange form/PKCE and refresh JSON are real operations. Refresh URL override
  applies only to refresh. Non-2xx responses become structured OpenAiLoginError;
  successful-response decode, transport and cancellation remain exceptional.
  Owned HTTP is closed at line 113.
- Confirmed union branches before credential publication in
  [BackendFileSystemAuthStore](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/shared/auth/filesystem/src/commonMain/kotlin/io/github/stream29/kodex/cli/auth/BackendFileSystemAuthStore.kt#L144)
  and
  [BackendOAuth](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/server/src/commonMain/kotlin/io/github/stream29/kodex/rpc/server/BackendOAuth.kt#L95);
  local filesystem refresh/login paths also branch on Failure before writing.
  Prepared login forwards real union, not duplicate token DTO. Union introduced
  in `f1a90006`, production callers retargeted in `5d43da38`.

### Account usage, catalog and Codex storage

- **Account:** FQCN
  `io.github.stream29.kodex.openai.accountusage.CodexAccountUsageStore`
  [spec](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/spec/account-usage/src/commonMain/kotlin/io/github/stream29/kodex/openai/accountusage/CodexAccountUsageStore.kt#L15)
  → same-package `CodexAccountUsageStoreImpl`
  [implementation](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/impl/account-usage/src/commonMain/kotlin/io/github/stream29/kodex/openai/accountusage/CodexAccountUsageStoreImpl.kt#L32)
  → [BackendAccountState](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/server/src/commonMain/kotlin/io/github/stream29/kodex/rpc/server/BackendAccountState.kt#L24)
  → BackendServices and global RPC.
  Auth collectLatest clears account-specific fallback; reset attempts are
  account-keyed; operationMutex serializes consume; expectedAccount is forwarded.
  Mandatory usage plus optional credits/token sections become a legitimate
  aggregate snapshot, not duplicated wire models. Optional-section cancellation
  rethrows. Close cancels own scope, not auth/API.
- Backend's owned store intentionally retains a definitive operation result
  across account switch while hiding old observable snapshots. This uses the
  existing retainDefinitiveOutcome path, not new competing state. RPC summary
  removes credentials at a real transport boundary and retains original snapshot
  type. Store/tests are exact renames in `f1a90006`.
- **Catalog:** FQCN
  `io.github.stream29.kodex.openai.modelcatalog.OpenAiModelCatalogStore`
  [spec](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/spec/model-catalog/src/commonMain/kotlin/io/github/stream29/kodex/openai/modelcatalog/OpenAiModelCatalogStore.kt#L16)
  → same-package `OpenAiModelCatalogStoreImpl`
  [implementation/factories](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/impl/model-catalog/src/commonMain/kotlin/io/github/stream29/kodex/openai/modelcatalog/OpenAiModelCatalogStoreImpl.kt#L25)
  → [BackendGlobalState owned composition](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/server/src/commonMain/kotlin/io/github/stream29/kodex/rpc/server/BackendGlobalState.kt#L80).
  Built-in snapshot exists synchronously, startup refresh is owned, explicit
  refresh uses getOrThrow, normalization preserves model/first-effort order,
  resolution uses longest prefix and single provider namespace with fallback.
  Close cancels own scope only. Bundled data is impl detail, not unused spec
  policy. Model budget function remains shared with actual consumers.
- **Codex storage:** FQCN
  `io.github.stream29.kodex.openai.codexclistorage.CodexCliStorage`
  [read-only operations](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/spec/codex-cli-storage/src/commonMain/kotlin/io/github/stream29/kodex/openai/codexclistorage/CodexCliStorage.kt#L12)
  → same-package `CodexCliStorageImpl`
  [decoder and real factory](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/impl/codex-cli-storage/src/commonMain/kotlin/io/github/stream29/kodex/openai/codexclistorage/CodexCliStorageImpl.kt#L10)
  → BackendFileSystemAuthStore auth loading and
  [BackendCodexMcpImport](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/server/src/commonMain/kotlin/io/github/stream29/kodex/rpc/server/BackendCodexMcpImport.kt#L17).
  Spec DTO serializers decode auth/MCP; missing files, malformed input and
  unsupported fields have explicit outcomes. Unknown-field diagnostic names
  avoid raw secret values. Only mcp_servers is imported; no Hooks/settings/
  model-cache/context/Session compatibility was smuggled in.
- External-Codex transport → MCP config conversion is a legitimate external
  format/domain boundary. Same-name factories above instantiate actual classes
  implementing specs; removing them without replacing construction would remove
  real work, unlike ablation of a disconnected policy shim.
  Splits are in `f1a90006`, consumers in `5d43da38`; subsequent storage/RPC
  retargets preserve this call chain.

## Findings and minimal experiments

### B1-1 — built mock is not independent of subsequent builder mutation

- Contract FQCN `io.github.stream29.kodex.openai.client.test.MockOpenAiClientBuilder`
  [build KDoc promises independence](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/spec/client-test/src/commonMain/kotlin/io/github/stream29/kodex/openai/client/test/MockOpenAiClientBuilder.kt#L68).
- Actual FQCN `io.github.stream29.kodex.openai.client.test.MockOpenAiClientBuilderImpl`:
  [default full handler captures mutable builder createResponseHandler](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/impl/client-test/src/commonMain/kotlin/io/github/stream29/kodex/openai/client/test/MockOpenAiClient.kt#L59).
  Build at lines 149–162 copies closure, not a snapshot of referenced handler.
  Private implementing `MockOpenAiClient.createResponse`
  [always invokes full handler](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/impl/client-test/src/commonMain/kotlin/io/github/stream29/kodex/openai/client/test/MockOpenAiClient.kt#L207).
- Concrete sequence: configure simple handler A → build client A → replace
  builder simple handler with B → call client A's flattened createResponse
  → B runs. This follows closure reference and sole operation body; it is
  not inferred merely from an unused-symbol grep.
- Actual callers:
  [DSL factory at lines 31–38](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/impl/client-test/src/commonMain/kotlin/io/github/stream29/kodex/openai/client/test/MockOpenAiClient.kt#L31)
  and [plain-operation mock test](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/impl/client-test/src/commonTest/kotlin/io/github/stream29/kodex/openai/client/test/MockOpenAiClientTest.kt#L45).
  One-shot DSL callers normally discard builder, hiding defect.
  This is not a production HTTP bypass; no real-client caller depends on this
  mutable mock behavior.
- Introducing delta: `f1a90006` adds independent-builder spec and flattens all
  mock Responses calls onto full-handler path. Before that commit plain DTO
  operation invoked constructor-snapshotted simple handler directly. Old
  full/Codex overload already had builder capture, so this is both inherited
  full-handler flaw and expanded plain-call regression, not a wholly new capture
  mechanism.
- Minimal ablation: constructor field createResponseHandler at line 176 and
  build argument at line 156 are behaviorally dead; complete mock body inspected.
  Removing those alone changes no operation but does not fix capture. A repair
  should snapshot fallback handling during build, without another adapter/DTO.
- Minimal future test: explicitly type simple handler overload, build A, mutate
  builder to B, call A and second built client; assert A remains A and second
  sees B. Repeat with explicit full handler. **Not run.**

### B2-1 — cleanup failure can replace original cancellation

- Applicable rule:
  [resource checklist requires original failure plus suppressed cleanup failure](file:///home/stream/ACodeSpace/push/BuildKodex/checklist/coroutine-resource-lifecycle.md#L10).
- Concrete production chain: RPC cancel → published backend-owned running Job
  → composed AgentRuntimeImpl.resume →
  [catch CancellationException / NonCancellable clearPending](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-runtime/impl/composition/src/commonMain/kotlin/io/github/stream29/kodex/agentruntime/impl/KodexAgentRuntimeComposition.kt#L160)
  → [inherited clearPending extension](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-state/contract/src/commonMain/kotlin/io/github/stream29/kodex/agentstate/contract/KodexAgentStateExtensions.kt#L57)
  → [KodexAgentStateImpl.completeToolCall storage writes](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-state/impl/src/commonMain/kotlin/io/github/stream29/kodex/agentstate/impl/KodexAgentStateImpl.kt#L389).
- If pending-completion write throws, execution cannot reach throw cancellation
  at line 165; cleanup error becomes escaping exception. Running slot finally
  still releases. Specific exception-preservation concern, not a claim slot is
  leaked or normal cleanup is cancelled.
- Present byte-identically before `f1a90006`; located in earlier named-source
  history including `c59de796`. No evidence introduced by this split; earliest
  semantic introducer not established.
- Static conditional path is clear, but no storage-failure/cancel reproduction
  executed. Keep B2 pending coordinator reproduction rather than claiming a
  newly observed filesystem failure.
- Minimal experiment: composed runtime with a pending local tool and suspended
  handler; cancel resume while forcing completeToolCall/storage write failure.
  Assert original cancellation
  remains primary with cleanup error suppressed, slot stays claimed through
  cleanup, partial completed prefix is valid, later operation can claim slot.
  Existing slot tests do not establish this failure case.

### B2-2 — actual remote-compaction default conflicts with recorded policy

- [OpenAI checklist line 16](file:///home/stream/ACodeSpace/push/BuildKodex/checklist/openai-module-boundaries.md#L16)
  says at most **two** protocol/transport retries.
  [OpenAiClientConfig default](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/impl/client/src/commonMain/kotlin/io/github/stream29/kodex/openai/client/OpenAiClientConfig.kt#L13)
  is **20** remote retries.
- Actual chain: BackendServices default client factory → Agent State compact
  DTO → OpenAiClient.compaction copies config at line 263 →
  [retry budget](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/impl/client/src/commonMain/kotlin/io/github/stream29/kodex/openai/client/OpenAiClient.kt#L646).
  Not unused config or decorator-only count; default production remote operation
  really uses 20. Cancellation is separately rethrown; no shared wall-clock
  deadline found.
- Historical introducer `fa436ce885d6c2e23ef0f729aaab2dd7a9afae09` changes remote
  retries 2 → 20 and SSE socket timeout 300000 → 600000 before split.
  `f1a90006` renames config unchanged. **Not a new migration regression.**
- [Config test](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/impl/client/src/commonTest/kotlin/io/github/stream29/kodex/openai/client/OpenAiClientConfigTest.kt#L18)
  expects 20; [remote helper retry test](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/impl/client/src/commonTest/kotlin/io/github/stream29/kodex/openai/client/OpenAiRemoteCompactionV2Test.kt#L171)
  supplies explicit limit 2. Both can pass without reconciling default/checklist.
- Confirmed discrepancy; unresolved acceptance intent. No explicit approval
  superseding two-retry guideline found in reviewed task records. Coordinator
  should reconcile policy/history before unconditional acceptance; do not
  silently “fix” to either value in this audit.
- Minimal future check: actual default compaction operation with deterministic
  failing SSE, count submissions, verify cancellation during backoff; separately
  obtain policy resolution. No external endpoint needed.

### D-1 — inherited Agent State KDoc differs from actual Runtime semantics

- [injectHistory KDoc](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-state/contract/src/commonMain/kotlin/io/github/stream29/kodex/agentstate/contract/KodexAgentState.kt#L217)
  says user injection does not rotate persisted turn ID. Actual steer calls
  injectHistory; [implementation](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-state/impl/src/commonMain/kotlin/io/github/stream29/kodex/agentstate/impl/KodexAgentStateImpl.kt#L330)
  rotates after qualifying prior assistant message. Outer resume Job/logical
  operation identity must not be conflated with persisted turn ID.
- Existing history inference is recorded in
  [Runtime checklist](file:///home/stream/ACodeSpace/push/BuildKodex/checklist/agent-state-and-runtime.md#L39)
  and [late-steer test](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-runtime/impl/decorator/steer/src/commonTest/kotlin/io/github/stream29/kodex/agentruntime/decorator/steer/SteerRuntimeTest.kt#L117),
  whose line 162 asserts changed ID. `80e4d277e4483267f2e99a2b4bb58d994b64fb94`
  changed implementation/test before split. Do not “repair” by reverting approved
  history-based identity semantics.
- [requestResponseApi KDoc](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-state/contract/src/commonMain/kotlin/io/github/stream29/kodex/agentstate/contract/KodexAgentState.kt#L173)
  groups Failed and Incomplete as exceptional; actual Failed returns retryable
  disposition to compact decorator, while Incomplete throws. Current Runtime
  retry behavior/checklist agrees with code. Inherited cross-root documentation
  debt should be reconciled with backend reviewer.

### D-2 — old model comments conflict with serializers/parser

- [ResponsesApiRequest comment](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/spec/models/src/commonMain/kotlin/io/github/stream29/kodex/openai/ResponsesApiModels.kt#L20)
  says default reasoning omitted; property at lines 44–45 uses ALWAYS encoding,
  consistent with later reasoning documentation and actual transport.
  [ExplicitReasoningHttpTest](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/impl/client/src/jvmTest/kotlin/io/github/stream29/kodex/openai/client/ExplicitReasoningHttpTest.kt#L1)
  checks real request-body projection. Old comment/model moved unchanged;
  no evidence of new omission introduced by split.
- [RemoteCompactionV2Response comment](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/spec/models/src/commonMain/kotlin/io/github/stream29/kodex/openai/CompactionModels.kt#L97)
  allows no Completed; actual parser rejects that ending with spec-owned
  incomplete exception. Nullable representation alone is not proof client
  permits null on success. Comment lineage includes `97d6b45e`; parser behavior
  predates rename and tests record terminal enforcement.
- Recorded debt, not reason to weaken wire validation or create replacement
  DTOs. Clarify intended documentation in separately authorized repair.

### D-3 — known mixed roots prevent whole-repository closure claims

- Scoped Runtime specs inherit real, unrelocated Agent State contract; related
  context-window projection is outside split. The
  [original migration plan](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/executable/2026-09-28-plan-domain-spec-impl-boundaries.md#L28)
  records incremental scope. Parent explicitly excludes unmigrated roots as
  automatic failures.
- No independent runtime state/model was added to hide those dependencies.
  No direct scoped spec → impl edge found. This does not erase transitive
  Shell/Tool/MCP issues owned by other reviewers or certify fully pure repository
  dependency graph.

## Tests, ablation and remaining U

- Runtime compact tests cover spec-typed state identity/delegation, continuation,
  20 Agent retries/reset, pre/mid budget checks, hook order and pending-tool stop.
  Steer tests cover pending rejection, ordered claims and repeated concurrent
  provider claims. Tool/turn-hook tests cover owned/unowned/removed calls,
  hook blocks/contexts, host interactions and continuation persistence.
  These exercise actual decorator methods, not only policy helpers.
- [AgentRuntimeRunningTurnTest](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-session/impl/in-memory/src/commonTest/kotlin/io/github/stream29/kodex/agentsession/inmemory/AgentRuntimeRunningTurnTest.kt#L51)
  exercises real composed runtime caller identity, resume/compact rejection,
  failure reuse and cancellation cleanup with pending steer retained.
  [BackendSessionHostTest](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/server/src/commonTest/kotlin/io/github/stream29/kodex/rpc/server/BackendSessionHostTest.kt#L202)
  covers accepted-operation ownership/lease renewal and shutdown joining.
  Static inspection is stronger than isolated fake, but no new execution.
  Provider-level coroutine tests do not establish end-to-end UI/RPC/persistence
  races on multiple native threads.
- OpenAI tests inspected cover result/login decoding, metadata/reasoning/union
  serialization, retry helpers, remote compaction terminal/order validation,
  catalog longest-prefix/namespace/normalization/cancellation and Codex parsing/
  redaction. Real-loopback HTTP tests cover concrete reasoning/metadata/header
  projection and transport-only state absence; not run. They do not prove
  equivalence for every flattened optional argument or every stream event.
- Account-root tests mostly cover aggregate snapshots; stronger switch behavior
  is in [BackendAccountStateTest](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/server/src/commonTest/kotlin/io/github/stream29/kodex/rpc/server/BackendAccountStateTest.kt#L84).
  It confirms intended definitive-result isolation with mocks, not real bearer
  switching/retry behavior against live service.
- Mock metadata deliberately uses `"mock-turn-metadata"` and does not expose
  transport-only turn-state to request handlers. DTO mock tests cannot prove
  real header projection or SSE lifetime. One metadata model test named for
  transport-only state only asserts thread identity; real HTTP body absence
  assertion is more relevant evidence.
- No test found for builder mutation after build (B1-1), pending-cleanup storage
  failure masking cancellation (B2-1), or production-default policy reconciliation
  (B2-2). Borrowed-resource close failures/catalog-collision failure paths also
  lack a demonstrated full-composition test in this review.
- Ordered minimal ablation/experiments for later authorized phase:
  1. Confirm B1 builder snapshot repro; remove only now-dead constructor handler
     after making fallback independent.
  2. Inject B2 cancellation-cleanup write failure in real composition.
  3. Resolve retry policy, then test default-operation submission count.
  4. Add representative real-composition borrowed-resource/collision assertions;
     do not proliferate policy-only specs or forwarding adapters.
- Removing runtime decorator specs would break implementation/factory contract
  typing and production composition. Removing pure models/codec would break
  serializer/transport/storage consumers. No broader “delete wrappers” proposal
  is justified by this audit.
- **U / not certified:** live endpoint compatibility; current external
  Codex/Rust protocol/schema; Curl/native cancellation/resource timing;
  multiplatform build/test success; full UI callback-to-RPC race execution;
  fault injection above; exhaustive historical protocol-variant tests. Focused
  test bodies were read, not every unrelated integration-test body or upstream
  library implementation. These limits are not counted as passed.

**REVIEW READY — coordinator may independently recheck B1/B2 and merge this
report; source repairs and resumed migration remain unauthorized.**

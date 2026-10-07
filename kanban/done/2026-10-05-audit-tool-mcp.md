# Task Tree

- `Trace Tool and MCP contracts through actual registration and dispatch`()
- `Review payloads, ownership, tests and ablation candidates`()
- `Publish evidence and unreviewed coverage`()

# Details

- Parent: [Spec/impl reacceptance](2026-10-05-reaccept-spec-impl-refactor.md).
- Own review scope: complete `tool` and `mcp` roots.
- Report only in this file; source, other tasks and shared resources are read-only.
- Apply parent acceptance rules and pinned baseline; no independent fixes or builds.
- Status: **REVIEW READY**; independent static review complete.

## Review report

## Baseline, method and result

- Reviewed shared inner branch `refactor/spec`, HEAD
  `821b66275aae2f0a9541fa2e9e03023d875c30b5`; inner status was clean at
  opening and after source review.
- Static source/history review only. No Gradle, tests, CLI, network connection,
  process fixture, IDE, resource operation, source edit, commit, push or branch
  change. Only this outer child report was written with `apply_patch`.
- Loaded project AGENTS, change/checklist/kanban/planning/document skills,
  parent task, this child, relevant utility/tool/MCP records and checklists.
  Also loaded `KotlinMcpSdk/AGENTS.md` before reading SDK call-chain evidence.
- **Do not accept the complete migration as meaningful-boundary closure.**
  The new MCP transport factory contracts are not used by production.
- Distinct findings: **one migration-introduced B1**, **four inherited B1
  contract/operation discrepancies**, and **two B2 review risks** below.
  The inherited findings are not claims that the move broke previously working
  behavior. The coordinator should keep those two categories separate.
- Shell is a shared finding, counted **zero additional times here**.
  Topic coverage is not a runtime or all-platform pass.

## Migration and original-declaration evidence

- `f1a9000603083fed7c2cfbcf3a7a3a16ebe8118c` and
  `5d43da3805836c0cb6623c1c0ea58ca455d5f61e` supply the earlier Runtime/OpenAI
  spec extraction and downstream retargeting.
- `6606efe93bfbb70d63bd93424f2f91385a0e08b4` is the Tool/MCP relocation and
  extraction commit. Compared both its parent declarations and its resulting
  tree, then compared `6606efe9..821b6627`.
- `d05e9cdb9bff5eb42609a5db505d57e66990a290` later retargets
  `tool/spec/contract` to `agent-storage-spec-clean-models`, and plan/context
  tests to `agent-storage-impl-in-memory`. No Tool/MCP Kotlin body changes
  occur in that interval.
- History uses real moves: existing topic DTOs, schemas, MCP core declarations
  and attached KDoc show `R100` source moves. Apply Patch grammar/description
  and builder result declarations were extracted from their original files,
  not copied alongside another live declaration.
- Material additions in `6606efe9`: the two MCP factory interfaces/default
  objects and the second stdio startup body. These are the B1-M1 evidence,
  not just project-name changes.
- Original locations in the table below are at `6606efe9^`; at `6606efe9`
  they already have the current layout. Treating `6606efe9` itself as the
  pre-migration tree would miss the new unused surfaces.

## Complete leaf-project/topic coverage

- Inventory: **31 tracked Gradle leaf projects**: Tool 13 specs + 12 impls;
  MCP 3 specs + 3 impls. No extra private-helper leaf project was found.
- `tool/spec/contract` is intentionally spec-only. MCP
  `impl/composition` corresponds to `spec/contract`; a matching basename is
  not an acceptance requirement.
- Directory-derived inclusion is real:
  [settings.gradle.kts lines 34–63](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/settings.gradle.kts#L34).
  Tracked Gradle consumers no longer use the removed original leaf coordinates.
- Status legend: `R` traced path; `D` explicitly documented debt; `B1`
  confirmed discrepancy; `B2` concrete risk needing a targeted experiment;
  `U` not reviewed. `R` means no additional finding on that path, not purity
  of every transitive dependency.
- FQCN prefix for Tool declarations is `io.github.stream29.kodex.tool`;
  for MCP contracts `io.github.stream29.kodex.mcp.contract`.
  Consumer chains C1–C4 and test coverage follow this table.

| Topic / current project leaves | Original declaration location | Current contract → real implementation/registration → caller | Result |
| --- | --- | --- | --- |
| Tool `spec/contract` | `tool/contract`, `Tool.kt`, `ToolName.kt` | [Tool.handle(PendingToolEvent), typedTool][t-contract] and namespace-aware `ToolName.matches` → concrete tools/builder → C1 runtime routing and completion | R; Shell transitive impact |
| Tool `{spec,impl}/builder` | `tool/tool-builder/JsonToolBuilder.kt` | [JsonToolHandlerResult, success/failure helpers][t-builder-spec] → [jsonTool/textTool/functionOutputTool, FunctionOutputTool][t-builder-impl] → current-time/context and other adapters → C1 | R; real result algebra, not a marker |
| Tool `{spec,impl}/apply-patch` | `tool/apply-patch/ApplyPatchTools.kt` | [ApplyPatchGrammar/Description][t-patch-spec] → [FreeformTool registration, ApplyPatchTool][t-patch-impl] → parser/ApplyPatchToolClient → filesystem → C1/C2 | R |
| Tool `{spec,impl}/current-time` | `tool/current-time/CurrentTimeSchemas.kt` | [input/output schemas][t-time-spec] → [clock.curr_time registration and textTool][t-time-impl] → CurrentTimeToolClient/Clock → C1/C2 | R; normal output deliberately text |
| Tool `{spec,impl}/get-context-remaining` | `tool/get-context-remaining/GetContextRemainingSchemas.kt` | [input/nullable output schemas][t-context-spec] → GetContextRemainingTools.spec → [state-bound textTool][t-context-impl] → tokensUntilCompaction/current snapshot → C1/C2 | R; mixed AgentState domain is outside this migration |
| Tool `{spec,impl}/image-generation` | `tool/image-generation/{contract,impl}` | [ImageGenToolArguments/GeneratedImageOutput serializers][t-image-spec] → ImageGenParametersSchema → [typed pending handler][t-image-impl] → ImageGenerationToolClient/OpenAiClient/artifacts → C1/C2 | B1-I1 inherited: history edit member is advertised but unusable |
| Tool `{spec,impl}/multi-agent` | `tool/multi-agent/{contract,impl}` | [SuggestSubagentTaskArgs/response/SessionMeta][t-multi-spec] → [SuggestSubagentTaskTools registration][t-multi-impl] → C2 parsing → C3 host confirmation/create/complete | R; host-owned handler is valid |
| Tool `{spec,impl}/plan` | `tool/plan/PlanSchemas.kt` | [UpdatePlanParametersSchema][t-plan-spec] → PlanTools registration → [updatePlanTool][t-plan-impl] using original OpenAI UpdatePlanArgs → appendPlanUpdate → C1/C2 | R; no copied plan DTO |
| Tool `{spec,impl}/request-user-input` | `tool/request-user-input/{contract,impl}` | [RequestUserInputArgs/response serializers][t-input-spec] → [registration][t-input-impl] → C2 parsing → C3 answer drafts/submit/complete | B1-I2 inherited: timeout member has no execution consumer |
| Tool `{spec,impl}/tool-search` | `tool/tool-search/{contract,impl}` | [SearchToolCallParams, result/document/source models][t-search-spec] → [ToolSearchEngine][t-search-impl]/document conversion/ToolSearchTools schema → C2 visibility + C1 special dispatch | R; real DTO/result contracts; not ordinary Tool |
| Tool `{spec,impl}/unified-exec` | `tool/unified-exec/{spec,impl}` | [UnifiedExecClient/ProcessSession][t-exec-spec] and ExecCommand/WriteStdin/Output serializers → [UnifiedExecToolClient implements client][t-exec-impl] → createTools → ShellClient/start/drain → C1/C2 | D-Shell shared boundary; own client is genuinely implemented |
| Tool `{spec,impl}/view-image` | `tool/view-image/{contract,impl}` | [ViewImageToolArguments/Output/Detail][t-view-spec] → ViewImageSchemas/options → [typed handler][t-view-impl] → ViewImageToolClient/codec/filesystem → C1/C2 | R; optional unsupported environment/detail exposure is off by default |
| Tool `{spec,impl}/web-run` | `tool/web-run/WebRunSchemas.kt` | [WebRunParametersSchema][t-web-spec] → [web.run registration/typed handler][t-web-impl] → SearchCommands serializer/OpenAiClient.search → C1/C2 | R; schema-only spec is meaningful |
| MCP `spec/contract`, `impl/composition` | `mcp/{contract,impl}` | [McpSettings/config/secret models][m-settings], [McpManager and ports][m-manager-spec], [McpClient][m-client-spec], [McpService][m-service-spec], McpTool → [service][m-service-impl]/[manager][m-manager-impl]/[owner/client][m-client-impl]/[tool projection][m-tool-impl]/OAuth → C4/C1/C2 | Real core contracts; B1-I3/I4 inherited projection/protocol issues; B2-1/2 below |
| MCP `{spec,impl}/stdio` | `mcp/stdio/McpStdioTransport.kt` | [McpStdioTransportFactory][m-stdio-spec] → default implements it, but [legacy ProcessClient extension][m-stdio-impl] remains the production entry → ProcessOwnedTransport/raw SDK adapters → C4 | B1-M1 new unused boundary + duplicated body |
| MCP `{spec,impl}/streamable-http` | `mcp/streamable-http/McpStreamableHttp.kt` | [McpStreamableHttpClientFactory][m-http-spec] → default implements it, but [legacy scope/client extensions][m-http-impl] construct clients/transports/auth → C4 | B1-M1 same root cause; do not count twice |

### Actual production chains

- **C1 — construction/dispatch/ownership.**
  [KodexAgentTools lines 39–97, 100–127](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-runtime/impl/composition/src/commonMain/kotlin/io/github/stream29/kodex/agentruntime/impl/KodexAgentTools.kt#L39)
  creates every fixed local handler, projects MCP client tools, and builds
  ToolSearchEngine. [Runtime composition lines 74–101](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-runtime/impl/composition/src/commonMain/kotlin/io/github/stream29/kodex/agentruntime/impl/KodexAgentRuntimeComposition.kt#L74)
  lends them to the runtime and closes fixed tools on owner completion.
  [Runtime lines 53–132 and 187–195](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-runtime/impl/decorator/tool/src/commonMain/kotlin/io/github/stream29/kodex/agentruntime/decorator/tool/KodexToolRuntimeImpl.kt#L53)
  index routes, dispatch actual Tool.handle or ToolSearchEngine.search, run
  hooks, and avoid a second completion after state-bound plan persistence.
- **C2 — request schema → actual decode → durable result.**
  [visibleToolSpecs lines 24–75](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-state/tool/src/commonMain/kotlin/io/github/stream29/kodex/agentstate/tool/KodexVisibleToolSpecs.kt#L24)
  uses real registrations, question-mode visibility and local/MCP deferred
  documents. [PendingToolEventProjection lines 60–195](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-state/tool/src/commonMain/kotlin/io/github/stream29/kodex/agentstate/tool/PendingToolEventProjection.kt#L60)
  decodes the actual serializers: ExecCommandArguments, WriteStdinArguments,
  SearchCommands, ImageGenToolArguments, ViewImageToolArguments,
  RequestUserInputArgs, SuggestSubagentTaskArgs, UpdatePlanArgs and
  SearchToolCallParams. Apply Patch uses parsePatch; MCP uses parsed JSON.
  Clean pending/stable events carry these same models, not alternate payloads.
- **C3 — host-owned tool completion.**
  [RequestUserInputViewModel lines 33–41, 107–180](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/request-user-input/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/cli/agent/RequestUserInputViewModel.kt#L33)
  and [SuggestSubagentTaskViewModel lines 116–177](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/suggest-subagent-task/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/cli/agent/SuggestSubagentTaskViewModel.kt#L116)
  consume the real typed pending events and complete typed stable events through
  their dependencies. Suggestion rejection never calls createSessions;
  acceptance does. A separate Tool implementation for these topics is not
  required.
- **C4 — application-wide MCP.**
  [BackendServices lines 56–66](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/server/src/commonMain/kotlin/io/github/stream29/kodex/rpc/server/BackendServices.kt#L56)
  creates OAuth and gives shared MCP service to Agent dependencies.
  [BackendGlobalState lines 45–94, 111–135](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/server/src/commonMain/kotlin/io/github/stream29/kodex/rpc/server/BackendGlobalState.kt#L45)
  constructs the real service/manager, shares the file update port with token
  refresh/CAS, and closes backend resources. Service.createOwner → owner
  openConnection/client.connect/readCatalog → publishCatalog/client.listTools
  → C1/C2 → McpToolImpl.handle → owner.call → SDK callTool. Backend
  reconnect calls manager/client; observations export sanitized manager state,
  not behavioral McpClient instances.

## Confirmed findings

### B1-M1 — new MCP transport contracts do not control production

- **Introduced in `6606efe9`; one root cause, two transport topics.**
- New stdio spec declares factory.open with owned-process close semantics;
  new HTTP spec declares createClient/openTransport. Both default objects
  nominally implement those interfaces.
- Production [McpServiceImpl lines 48–49, 185–213][m-service-impl]
  creates `scope.McpStreamableHttpClient()` and concrete `ProcessClient`,
  adds `withMcpAuthorization`, then calls
  `HttpClient.openMcpStreamableHttpTransport` or
  `ProcessClient.openMcpStdioTransport`. The public composition boundary
  neither accepts nor invokes either new factory contract.
- [DefaultMcpStdioTransportFactory lines 49–80][m-stdio-impl] invokes a
  separate private ProcessClientSpec startup body. The still-used public
  ProcessClient body at lines 22–46 performs the same command construction,
  SDK pipe adaptation, ownership wrapping and failure cleanup.
- [DefaultMcpStreamableHttpClientFactory lines 83–90][m-http-impl] is only
  a forwarding object. No production or test Kotlin caller references either
  default object or either factory type outside its declaration/implementation.
  This was checked across the repository, not inferred from names.
- Tests still use old extensions or real service → old extension. In
  particular [HTTP test lines 18–23, 47–50](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/impl/streamable-http/src/commonTest/kotlin/io/github/stream29/kodex/mcp/streamablehttp/McpStreamableHttpTest.kt#L18)
  and [stdio service test lines 35–54](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/impl/stdio/src/jvmTest/kotlin/io/github/stream29/kodex/mcp/stdio/McpStdioTransportIoTest.kt#L35)
  cannot establish new-contract takeover.
- The legacy mechanisms are not all useless: ProcessOwnedTransport and pipe
  adapters really add ownership/framing adaptation. The defect is the
  disconnected new boundary and extra stdio body, not their SDK delegation.
- Minimal hypothetical ablation: remove new factory types/default objects,
  duplicate private stdio body and new spec dependencies. Existing production
  extension call chains would remain semantically unchanged. **Not executed.**
  Conversely, a later acceptance test should inject a sentinel factory at
  real service composition and prove startup/close crosses that boundary.

### B1-I1 — advertised image-history edit path always fails

- **Inherited; not a migration regression and not an unused whole DTO.**
- [ImageGenToolArguments lines 6–18][t-image-spec] retains
  `numLastImagesToInclude`; registration tells the model to use it for
  recent conversation images ([ImageGenerationTools lines 30–36][t-image-impl]).
- C2 decodes it unchanged; C1 constructs ImageGenerationToolClient without a
  conversation-history provider; typed handler passes `pending.arguments`
  directly to `client.run` at lines 65–68.
- [Client lines 82–85](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/image-generation/src/commonMain/kotlin/io/github/stream29/kodex/tool/imagegeneration/ImageGenerationToolClient.kt#L82)
  validates any requested count then unconditionally throws “requires
  conversation history from the agent loop.” No production adapter fills that
  history or converts this member to paths.
- [Existing test lines 45–55](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/image-generation/src/commonTest/kotlin/io/github/stream29/kodex/tool/imagegeneration/ImageGenerationToolClientTest.kt#L45)
  asserts failure, not successful history selection. Generate and local-path
  edit paths remain real.
- Minimal acceptance experiment: route a pending history-edit call through
  real composition with synthetic retained images and a fake image API.
  Assert image selection/API request, not merely expected local failure.
  A change to hide/remove the advertised member instead needs separate
  contract/wire approval; do not silently alter it during migration repair.

### B1-I2 — request_user_input timeout is recorded but never operates

- **Inherited; real request/response DTOs and manual interaction do work.**
- [Spec lines 10–16][t-input-spec] gives null a wait-for-user meaning;
  registration/schema advertise a non-blocking auto-resolution window.
- C2 serializer preserves the field into the pending event. C3 synchronize
  merely publishes the arguments; submit builds an Answered result and resumes
  only on explicit submission. There is no timer, timeout admission or
  automatic completion in the actual handler path.
- Repository-wide production `autoResolutionMs` references are the model,
  schema/description, retention-size estimation and
  [history display line 995](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/history/CleanEventView.kt#L995).
  None consumes the value to schedule or finish a call.
- [Tool test lines 44–70](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/request-user-input/src/commonTest/kotlin/io/github/stream29/kodex/tool/requestuserinput/RequestUserInputToolsTest.kt#L44)
  checks omission when absent; it does not validate an elapsed deadline.
- Minimal acceptance experiment: use a fake clock with
  `autoResolutionMs=60000`, no user submission and real pending/host binding;
  inspect completion/resume after the window. Removing the timeout member
  would require separate wire review. Component reviewer should deduplicate
  this inherited cross-domain finding if independently reported.

### B1-I3 — valid distinct MCP names can collapse to duplicate routes

- **Inherited confirmed projection/dispatch defect; not a naming heuristic.**
- Manager/draft validation permits distinct nonblank `a-b` and `a_b`;
  [configuration validation lines 43–46](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/impl/composition/src/commonMain/kotlin/io/github/stream29/kodex/mcp/impl/McpConfigurationValidation.kt#L43)
  checks only trimmed uniqueness. Both servers can legally publish `echo`.
- [McpToolImpl lines 38–47, 108–118][m-tool-impl] maps both namespaces to
  `mcp__a_b`; remote names `x-y` and `x_y` likewise both become `x_y`.
  [Client lines 52–59][m-client-impl] deduplicates original SDK names,
  not normalized routes.
- Service publishes both raw-name clients → C1 flatMaps both tool lists →
  [ToolRouting.index lines 18–28](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-runtime/impl/decorator/tool/src/commonMain/kotlin/io/github/stream29/kodex/agentruntime/decorator/tool/ToolRouting.kt#L18)
  sees identical ToolName routes and throws `IllegalArgumentException`
  before tool handling. The runtime's duplicate guard is correct; projection
  has supplied an unusable catalog.
- No separate collision-disambiguation catalog stage was found in this chain.
  Do not rely on the historical foundation task's SHA-1/collision claim as
  current evidence.
- Minimal pure acceptance fixture: two catalog owners `a-b`/`a_b`, both with
  `echo`, plus one server with `x-y`/`x_y`; pass their real projections into
  dynamic routing and assert unique, executable routes. No live server needed.
  Overlong names should be tested separately; no provider rejection was
  observed or claimed in this review.

### B1-I4 — “fixed 2025-11-25/no negotiation” contradicts the actual SDK path

- **Inherited code/checklist discrepancy; not proof of a newly broken
  connection and not a request to add protocol features.**
- [MCP management line 7](file:///home/stream/ACodeSpace/push/BuildKodex/checklist/mcp-management.md#L7)
  and MCP client checklist describe fixed `2025-11-25`, with no old/new
  protocol negotiation.
- C4 owner constructs ordinary SDK Client and calls connect, without a
  stricter protocol gate. The pinned SDK submodule is
  `53f717661712d618cd4822d1760392e903989ead`.
- [SDK Client lines 202–225](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/KotlinMcpSdk/kotlin-sdk-client/src/commonMain/kotlin/io/modelcontextprotocol/kotlin/sdk/client/Client.kt#L202)
  proposes latest but accepts any SUPPORTED_PROTOCOL_VERSIONS result.
  [SDK constants lines 12–23](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/KotlinMcpSdk/kotlin-sdk-core/src/commonMain/kotlin/io/modelcontextprotocol/kotlin/sdk/types/common.kt#L12)
  include `2025-06-18`, `2025-03-26`, and `2024-11-05`.
- The actual [HTTP fixture line 571](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/impl/composition/src/jvmTest/kotlin/io/github/stream29/kodex/mcp/impl/McpServiceImplIoTest.kt#L571)
  and [stdio fixture line 19](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/impl/stdio/src/jvmTest/kotlin/io/github/stream29/kodex/mcp/stdio/McpStdioServerFixture.kt#L19)
  return `2025-03-26`. These tests are not evidence of enforcing a fixed version.
- Minimal acceptance experiment: feed initialization replies with latest,
  one supported older version and an unsupported version through a fake
  transport. First resolve whether “fixed” means latest proposal or strict
  acceptance; any stricter runtime change is separate behavioral approval.

## B2 risks, KDoc and contract completeness

- **B2-1 — overlapping refresh/reconnect can publish an old catalog last.**
  [Owner.refresh lines 155–175][m-client-impl] reads under reader lock,
  releases it, then calls publishCatalog. Reconnect independently uses writer
  lock/reconnectMutex and publishes its new catalog at lines 112–149.
  [Service publication lines 292–299][m-service-impl] rejects retired owners
  but carries no connection/catalog revision for an unchanged owner.
  Concrete schedule: refresh finishes reading old connection → pauses before
  publication → reconnect publishes new catalog → old refresh publishes.
  The required ordering for overlapping successful operations is not fully
  specified. Use deterministic barriers around publication/read completion,
  not live servers, to determine intended/current behavior.
- **B2-2 — throwing and authorization semantics are incomplete in specs.**
  [McpDraftConfiguration lines 4–32, 64–79](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/spec/contract/src/commonMain/kotlin/io/github/stream29/kodex/mcp/contract/McpDraftConfiguration.kt#L4)
  publicly throws validation errors without `@throws` conditions.
  Manager spec operations at lines 265–278 lack operation-level failure
  conditions for errors shown in actual manager commands.
  HTTP spec only covers scope cancellation and isolated client/transport;
  the refresh/one-401-retry mechanism is documented on the old impl extension,
  not represented by the extracted factory contract.
  [Boundary checklist lines 5–8](file:///home/stream/ACodeSpace/push/BuildKodex/checklist/spec-impl-module-boundaries.md#L5)
  requires those normative semantics in spec. This is a documentation/
  replaceability acceptance risk, not evidence that every possible exception
  must be turned into a new result type.
- Unified Exec is stronger: spec documents argument/local-process exceptions,
  completed-but-unread sessions and termination/final-read semantics; real
  implementation implements these operations and the observable session
  interface. Normal raw/typed adapters legitimately perform decoding,
  completion or ownership work.
- No empty marker Tool spec, copied alternate Tool payload, second writable
  MCP configuration model or dummy private-helper project was found.
  Private catalog/identity/snapshot/result classes serve immutable metadata,
  reconciliation, transport results or sanitized observation; they are not
  parallel public configuration authorities.

## Shared debt and deliberate non-findings

- **D-Shell / shared B1 impact, zero new root-cause count.**
  [Unified Exec spec dependency line 9](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/spec/unified-exec/build.gradle.kts#L9)
  directly consumes `utils-shell-client-impl`; its model imports the existing
  host-bearing `Shell`. UnifiedExecToolClient actually uses ShellClient, not
  the separate unused Shell spec surface reported by the utility reviewer.
- The dependency propagates through
  [clean-model spec lines 12–17](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-storage/spec/clean-models/build.gradle.kts#L12)
  → Tool contract → specs for apply-patch, builder, current-time,
  get-context-remaining and plan → MCP core and both transport specs.
  Image-generation, multi-agent, request-user-input, tool-search, view-image
  and web-run own specs have no direct Shell edge; their Tool implementations
  still consume the shared Tool contract. Do not claim full spec closure.
- The utility migration record explicitly documents the direct Unified Exec
  mixed edge; the four-component record also documents its transitive closure.
  This report does not weaken the separate confirmed Shell blocker to a pass.
- Current-time/context `outputSchema` does **not** require JSON object output
  on ordinary Responses calls: [ResponsesApiTool lines 67–82](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/spec/models/src/commonMain/kotlin/io/github/stream29/kodex/openai/ToolSpecModels.kt#L67)
  explicitly makes it transient code-mode metadata; ordinary returned text
  agrees with the utility display checklist and tests.
- ViewImage defaults do not advertise `environment_id` or original-detail
  capability. The client rejects unresolved environment and downgrades detail
  unless enabled. Merely retaining those DTO fields is not a fake whole spec.
- Tool typedTool/builder no-op close is valid for resource-free adapters.
  Unified Exec's CloseableUnifiedExecTool adds real shared-client ownership;
  McpTool no-op close correctly leaves application-wide owner to McpService.
  ProcessOwnedTransport closes the process once and delegates SDK close.
- Core clients have fixed listTools generations and shared connection owner;
  nonhealthy owner.call returns Unavailable without remote invocation.
  Successful catalog publication replaces client view; loss does not erase
  catalog. Blocked client's retained tools reference the original owner, not a
  second remote connection. After retirement completes, that closed owner
  prevents calls. [Service lines 146–175][m-service-impl] publish the blocked
  view before closing retired owners; atomic prevention during that interval
  is not established by this static review.
- Service identity excludes rotating token values. Per-server authorizer reads
  latest persisted credentials, refreshes under its mutex and writes via the
  same store. Manager/runtime state is derived, not a new settings truth.
- RPC value-only import intentionally does not replay old Replace/invalidate
  intent. Legacy manager preview/apply/effects remain but are not the current
  frontend import/OAuth boundary. Backend's concrete
  commitConfigurationChange/loginWithPreparedCallback/PreparedMcpOAuthLogin
  additions are real RPC composition behavior, not forwarding-only specs.
  Their core-contract exposure remains a documented architectural limitation,
  not an invented requirement to change frozen RPC APIs.
- Current `tool-handler-decisions.md` still quotes an old raw ResponseItem
  signature; the utility task explicitly identifies it as superseded by clean
  events. Likewise the older MCP checklist direct-extension/no-factory wording
  conflicts with the later authorized transport split. These stale instructions
  do not justify claiming that unused new factories were successfully cut over.

## Test evidence and limits

- Read test source and assertions; **ran none**. Prior task “passed” statements
  were context only. Existing generated build directories/artifacts were not
  used as new verification evidence and were not removed from the shared tree.

| Topic | Existing test source and what it can establish statically | Important limit |
| --- | --- | --- |
| contract / builder | ToolContractTest; JsonToolBuilderTest: namespaced match, JSON/text/native completion, wrong custom payload, resource-free close | Isolated adapters, not complete production composition |
| apply-patch | ApplyPatchToolsTest: real freeform grammar registration | Topic test only asserts schema; filesystem semantics belong to patch utility tests |
| current-time / context | CurrentTimeToolsTest; GetContextRemainingToolsTest and BehaviorTest: UTC/text/nullable metadata/current compaction budget | Fixed clock/mock Agent dependencies; no new provider/runtime pass |
| image-generation | ToolsTest, ClientTest, GeneratedImageArtifactsTest: DTO inputs, generated image content/artifact hint, path sanitization; history branch asserts failure | No successful conversation-history edit; synthetic image API |
| multi-agent | SuggestSubagentTaskToolsTest: content-only input, accepted/rejected serializer shapes; component ViewModel tests exist | Tool tests do not prove actual backend Session creation/receipt semantics |
| plan | PlanToolsTest, UpdatePlanToolTest: schema and state-bound matching completion | Does not by itself prove generic dispatcher avoids duplicate persistence; inspected C1 separately |
| request-user-input | RequestUserInputToolsTest and component ViewModel tests: schema/wire/keyed manual responses, revision-bound submission | No deadline/automatic-resolution execution test identified |
| tool-search | ToolSearchToolsTest: invalid values, standalone/namespaced results, source description, deferred loadables | SearchIndex algorithm/platform correctness owned by utility reviewer; no new model/provider pass |
| unified-exec | UnifiedExecToolsTest + target test-shell actuals: metadata, login/settings shell, yield/poll, PTY, cancellation, final read | Real-I/O tests require execution; existing historical Linux cwd failure not retested |
| view-image / web-run | ViewImageClient/Tools tests: native image result/failure/capability; WebRunTools/RequestControls: reserved schema, existing medium default | Codec/provider/real endpoint outcomes unvalidated |
| MCP contract / manager | McpServerConfigurationTest: serializer/secret rendering; ManagerImplTest: edit/rename/secrets/import/login/reconnect | Manager tests use fake ports; old import behavior differs intentionally from RPC import |
| MCP client/service/OAuth | McpSchemaProjectionTest; ServiceImplIoTest: generations/loss/blocked/token-refresh; OAuthIoTest: challenge/discovery/registration/PKCE/token resource | Loopback fixtures not run; older protocol replies; no normalized-route collision or overlapping publication test identified |
| MCP transports | StreamableHttpTest: cancellation/retry/same-endpoint isolation; stdio JVM/JS fixtures: env/cwd/raw protocol/service lifecycle | No new factory caller; tests cover legacy path. JS fixture presence is not CLI Node acceptance |

- SDK call-chain spot checks: listTools capability gate/pagination is real;
  Protocol request cancellation removes pending handlers and sends cancellation
  ([SDK Protocol lines 830–895](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/KotlinMcpSdk/kotlin-sdk-core/src/commonMain/kotlin/io/modelcontextprotocol/kotlin/sdk/shared/Protocol.kt#L830)).
  Stdio uses SDK JSON-RPC read buffer/write framing, with raw process pipes,
  not Shell text buffers. HTTP transport close cancels its transport scope,
  not the borrowed HttpClient; owner release/scope cancellation handles client
  lifetime. These are source traces, not conformance claims.

## Minimal follow-up acceptance gates and unreviewed bits

- First resolve B1-M1 with one real transport boundary and sentinel composition
  tests. The suggested ablation is diagnostic, not permission to remove APIs
  or edit shared code.
- Keep inherited B1-I1/I2/I3/I4 visible separately; decide approved contract
  intent before any repair, especially payload removal or protocol tightening.
- B2-1 needs controlled overlapping-operation barriers; B2-2 needs normative
  exception/auth documentation review, not broad speculative refactoring.
- `U`: exhaustive platform process-tree termination/PTY/cwd behavior,
  image codecs, Lucene internals, Ktor redirect/security behavior, complete SDK
  conformance, OAuth crypto arithmetic, real providers/servers, live CLI/UI and
  every failure/cancellation interleaving, including blocked-view publication
  versus retired-owner closure. Those are outside this static review's
  validated claims; utility/component owners cover their own implementation
  internals.
- Every Tool/MCP leaf topic has a contract/real caller/test trace above;
  **no leaf topic is silently left U or counted as a runtime pass**.
  Cross-root consumers were read to establish invocation, not independently
  accepted as full Runtime/RPC/component implementations.
- Ready for coordinator's asynchronous evidence check. **REVIEW READY**.

[t-contract]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/spec/contract/src/commonMain/kotlin/io/github/stream29/kodex/tool/contract/Tool.kt#L13
[t-builder-spec]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/spec/builder/src/commonMain/kotlin/io/github/stream29/kodex/tool/builder/JsonToolBuilderSpec.kt#L10
[t-builder-impl]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/builder/src/commonMain/kotlin/io/github/stream29/kodex/tool/builder/JsonToolBuilder.kt#L22
[t-patch-spec]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/spec/apply-patch/src/commonMain/kotlin/io/github/stream29/kodex/tool/applypatch/ApplyPatchToolSpec.kt#L3
[t-patch-impl]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/apply-patch/src/commonMain/kotlin/io/github/stream29/kodex/tool/applypatch/ApplyPatchTools.kt#L18
[t-time-spec]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/spec/current-time/src/commonMain/kotlin/io/github/stream29/kodex/tool/currenttime/CurrentTimeSchemas.kt#L6
[t-time-impl]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/current-time/src/commonMain/kotlin/io/github/stream29/kodex/tool/currenttime/CurrentTimeTools.kt#L18
[t-context-spec]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/spec/get-context-remaining/src/commonMain/kotlin/io/github/stream29/kodex/tool/getcontextremaining/GetContextRemainingSchemas.kt#L9
[t-context-impl]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/get-context-remaining/src/commonMain/kotlin/io/github/stream29/kodex/tool/getcontextremaining/GetContextRemainingTool.kt#L12
[t-image-spec]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/spec/image-generation/src/commonMain/kotlin/io/github/stream29/kodex/tool/imagegeneration/ImageGenerationToolModels.kt#L6
[t-image-impl]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/image-generation/src/commonMain/kotlin/io/github/stream29/kodex/tool/imagegeneration/ImageGenerationTools.kt#L30
[t-multi-spec]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/spec/multi-agent/src/commonMain/kotlin/io/github/stream29/kodex/tool/multiagent/SuggestSubagentTaskModels.kt#L8
[t-multi-impl]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/multi-agent/src/commonMain/kotlin/io/github/stream29/kodex/tool/multiagent/SuggestSubagentTaskTools.kt#L12
[t-plan-spec]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/spec/plan/src/commonMain/kotlin/io/github/stream29/kodex/tool/plan/PlanSchemas.kt#L6
[t-plan-impl]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/plan/src/commonMain/kotlin/io/github/stream29/kodex/tool/plan/UpdatePlanTool.kt#L20
[t-input-spec]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/spec/request-user-input/src/commonMain/kotlin/io/github/stream29/kodex/tool/requestuserinput/RequestUserInputModels.kt#L8
[t-input-impl]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/request-user-input/src/commonMain/kotlin/io/github/stream29/kodex/tool/requestuserinput/RequestUserInputTools.kt#L9
[t-search-spec]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/spec/tool-search/src/commonMain/kotlin/io/github/stream29/kodex/tool/toolsearch/ToolSearchModels.kt#L18
[t-search-impl]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/tool-search/src/commonMain/kotlin/io/github/stream29/kodex/tool/toolsearch/ToolSearchEngine.kt#L8
[t-exec-spec]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/spec/unified-exec/src/commonMain/kotlin/io/github/stream29/kodex/tool/unifiedexec/UnifiedExecClient.kt#L15
[t-exec-impl]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/unified-exec/src/commonMain/kotlin/io/github/stream29/kodex/tool/unifiedexec/UnifiedExecToolClient.kt#L35
[t-view-spec]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/spec/view-image/src/commonMain/kotlin/io/github/stream29/kodex/tool/viewimage/ViewImageModels.kt#L12
[t-view-impl]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/view-image/src/commonMain/kotlin/io/github/stream29/kodex/tool/viewimage/ViewImageTools.kt#L27
[t-web-spec]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/spec/web-run/src/commonMain/kotlin/io/github/stream29/kodex/tool/webrun/WebRunSchemas.kt#L152
[t-web-impl]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/web-run/src/commonMain/kotlin/io/github/stream29/kodex/tool/webrun/WebRunTools.kt#L127
[m-settings]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/spec/contract/src/commonMain/kotlin/io/github/stream29/kodex/mcp/contract/McpSettings.kt#L9
[m-manager-spec]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/spec/contract/src/commonMain/kotlin/io/github/stream29/kodex/mcp/contract/McpManager.kt#L175
[m-client-spec]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/spec/contract/src/commonMain/kotlin/io/github/stream29/kodex/mcp/contract/McpClient.kt#L6
[m-service-spec]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/spec/contract/src/commonMain/kotlin/io/github/stream29/kodex/mcp/contract/McpService.kt#L5
[m-service-impl]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/impl/composition/src/commonMain/kotlin/io/github/stream29/kodex/mcp/impl/McpServiceImpl.kt#L41
[m-manager-impl]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/impl/composition/src/commonMain/kotlin/io/github/stream29/kodex/mcp/impl/McpManagerImpl.kt#L55
[m-client-impl]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/impl/composition/src/commonMain/kotlin/io/github/stream29/kodex/mcp/impl/McpClientImpl.kt#L45
[m-tool-impl]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/impl/composition/src/commonMain/kotlin/io/github/stream29/kodex/mcp/impl/McpTool.kt#L31
[m-stdio-spec]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/spec/stdio/src/commonMain/kotlin/io/github/stream29/kodex/mcp/stdio/McpStdioTransportSpec.kt#L8
[m-stdio-impl]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/impl/stdio/src/commonMain/kotlin/io/github/stream29/kodex/mcp/stdio/McpStdioTransport.kt#L22
[m-http-spec]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/spec/streamable-http/src/commonMain/kotlin/io/github/stream29/kodex/mcp/streamablehttp/McpStreamableHttpSpec.kt#L8
[m-http-impl]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/impl/streamable-http/src/commonMain/kotlin/io/github/stream29/kodex/mcp/streamablehttp/McpStreamableHttp.kt#L26

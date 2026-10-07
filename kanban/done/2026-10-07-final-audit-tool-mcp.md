REVIEW READY

# Task Tree

- `Verify the pinned source baseline`()
- `Trace all tool schemas, dispatch and MCP assembly paths`()
- `Review transport ownership, public contracts and tests`()
- `Publish the independently evidenced report`()

# Details

- Status: **REVIEW READY**; coordinator must independently verify findings.
- Parent: [final repository reacceptance](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-07-final-reaccept-spec-impl-repository.md#L1).
- Pin: inner `refactor/spec`, HEAD `6b7129fa0e719dde8f88a0034b9d572679984a03`.
  Initial, checkpoint and pre-publication checks matched; inner porcelain output was empty.
- Read-only source/history/test-assertion audit. Only this outer child report was written.
  No build/test execution, Gradle, IDE, network, live MCP, device/resource operation or Git mutation.
- Loaded outer AGENTS, scoped SDK AGENTS, change/checklist/kanban/planning skills,
  workspace/IDE/Gradle guidance and document/ask-user guidance.
- Consulted [module boundaries][boundaries], tool-handler/search and relevant
  apply-patch/command/utility/image/view/web/external/display checklists, MCP client,
  management, RPC and coroutine lifecycle checklists. Display redesign remains planning,
  not an invented current requirement.
- Loaded Draft, parent, closure planning and prior Tool/MCP audit. Prior verdicts/test
  logs were inputs, not inherited passes. Findings below come from current source and history.

## Verdict and checkpoint

- **Physical/production takeover:** reviewed all 29 actual projects, not virtual roots.
  Meaningful DTO/schema specs and concrete function specs are consumed.
  No empty spec, self-referencing fake interface or unconsumed transport factory remains.
- **Repaired migration blocker:** `eff7b10e` removes both unused MCP transport factories,
  their default wrappers and the duplicate private stdio startup. The service now calls
  the sole real Process-backed stdio function and real HTTP/auth functions.
  Unified-exec now depends on the genuine Shell spec, not its former impl edge.
- **Not an unconditional behavioral pass:** B1-I1/I2/I3 below are inherited but real
  contract/behavior gaps. No confirmed *new migration-introduced* behavior blocker was
  found in these roots. That does not waive inherited blockers.
- B2 publication interleavings and incomplete failure KDoc are explicitly unproven or
  incompletely specified; strict protocol acceptance and model capability fallback are
  baseline policy debts, not silently repaired or declared new drift.
- Checkpoint was published here as `IN PROGRESS` before this final consolidation.
  All tests below were **read, not run**. Static review cannot certify live protocol,
  security, process termination or all-platform behavior.

## 1. Complete project coverage

- Counted actual `build.gradle.kts`: **25 Tool + 4 MCP = 29**.
  [settings project-name derivation][settings] uses path names; removed MCP impl
  transport leaves are not current projects.
- Grade legend follows parent: R = reviewed key path/no listed finding; D = explicit
  exception/debt with gate; B1 = confirmed chain discrepancy; B2 = specific suspicion
  or contract-evidence gap; U = unreviewed. R is not runtime certification.
- Primary grades: **21 R, 2 D, 5 B1, 1 B2, 0 U projects**.
  Shared findings are not five independent defects: image/input each affect two rows.
- Trace keys T0–T12 and M0–M2 below include source, production caller and test evidence.
- Read all 29 build files. Spec main has no direct impl-project edge or UI framework
  leakage in these roots. [Unified spec build:9][execbuild] uses Shell spec;
  [stdio build:8–21][stdiobuild] uses Process spec in main and composition only in
  JVM tests, not a main cycle. [HTTP build:8–35][httpbuild] exposes Ktor/SDK and binds
  actual platform engines: intentionally concrete transport spec, not engine-independent port.
  [Composition build:1–34][mcpbuild] binds Process impl and both real transport specs.
- Tool projects and MCP specs use `kodex.kmp-host` (host targets plus JS/node);
  MCP composition uses `kodex.kmp-cli` (host targets, no JS). Unified-exec adds posix
  hierarchy and JVM/JS/posix/mingw shell-description actuals. No owned project adds
  resource generation or a second serializer owner; serialization plugins/typed models
  are real. Direct graph inspection is not resolved Gradle-variant validation.

| Actual project | Grade | Declaration / trace / remaining gate |
| --- | --- | --- |
| `:tool-spec-contract` | R | [Tool / typedTool][toolcontract], [ToolName][toolname]; T0 |
| `:tool-spec-builder` | R | [JsonToolHandlerResult][builderspec]; T1, consumed result algebra |
| `:tool-impl-builder` | R | [textTool / FunctionOutputTool][builderimpl]; T1; optional API ablation A1 |
| `:tool-spec-apply-patch` | R | [description / grammar][patchspec]; T2 |
| `:tool-impl-apply-patch` | R | [ApplyPatchTools][patchimpl]; T2, actual filesystem caller |
| `:tool-spec-current-time` | R | [schemas][timespec]; T3; output metadata is transient, not JSON wire |
| `:tool-impl-current-time` | R | [CurrentTimeTools][timeimpl]; T3 |
| `:tool-spec-get-context-remaining` | R | [nullable-budget schema][contextspec]; T4 |
| `:tool-impl-get-context-remaining` | R | [getContextRemainingTool][contextimpl]; T4 |
| `:tool-spec-image-generation` | B1 | [ImageGenToolArguments][imagespec]; T5 / B1-I1, inherited history promise |
| `:tool-impl-image-generation` | B1 | [ImageGenerationTools][imageimpl]; T5 / B1-I1 |
| `:tool-spec-multi-agent` | R | [arguments / response serializers][multispec]; T6, host-owned behavior |
| `:tool-impl-multi-agent` | R | [SuggestSubagentTaskTools][multiimpl]; T6 |
| `:tool-spec-plan` | R | [UpdatePlanParametersSchema][planspec]; T7 |
| `:tool-impl-plan` | R | [updatePlanTool][planimpl]; T7, no second completion |
| `:tool-spec-request-user-input` | B1 | [RequestUserInputArgs][inputspec]; T8 / B1-I2, inherited timeout promise |
| `:tool-impl-request-user-input` | B1 | [registration][inputimpl]; T8 / B1-I2 |
| `:tool-spec-tool-search` | D | [document / result / argument models][searchspec]; T9, D-A1 enum/API gate |
| `:tool-impl-tool-search` | D | [ToolSearchEngine][searchengine]; T9, D-CAP policy gate |
| `:tool-spec-unified-exec` | R | [UnifiedExecClient][execspec]; T10; real Shell spec edge |
| `:tool-impl-unified-exec` | R | [UnifiedExecToolClient][execimpl]; T10; disabled-I/O limitation |
| `:tool-spec-view-image` | R | [ViewImageToolArguments / output][viewspec]; T11 |
| `:tool-impl-view-image` | R | [ViewImageTools][viewimpl]; T11; production capabilities agree |
| `:tool-spec-web-run` | R | [WebRunParametersSchema][webspec]; T12 |
| `:tool-impl-web-run` | R | [WebRunTools][webimpl]; T12, actual API invocation |
| `:mcp-spec-contract` | B2 | [McpManager][managercontract], client/service/settings; M0, B2-KDOC; D-PROTOCOL |
| `:mcp-spec-stdio` | R | [ProcessClient.openMcpStdioTransport][stdio]; M1, sole real startup |
| `:mcp-spec-streamable-http` | R | [HTTP/auth/transport functions][http]; M2, actual borrowing ownership |
| `:mcp-impl-composition` | B1 | [McpServiceImpl][service]; M0–M2 / B1-I3; B2-R1/R2 |

## 2. Original → current specification

- Independently inspected `6606efe93bfbb70d63bd93424f2f91385a0e08b4` and its parent:
  most declarations are R100 moves, not new semantics. Original paths at
  `6606efe9^` are listed below; current URIs are in the coverage/trace links.
- Reproduction for old-source evidence is a read-only
  `git -C Kodex show 6606efe9^:<original-path>`. Old locations are historical,
  not misleading current-file links.

| Trace | Original location under inner repository | Current split / semantic check |
| --- | --- | --- |
| T0 | `tool/contract` | `tool/spec/contract`; clean-event contract and names retained |
| T1 | `tool/tool-builder` (`src/commonMain/kotlin/io/github/stream29/kodex/tool/builder/JsonToolBuilder.kt`) | result algebra extracted to spec; actual decode/completion remains impl |
| T2 | `tool/apply-patch` | grammar/description spec; parser/client/tool impl |
| T3 | `tool/current-time` | schemas spec; clock/text completion impl |
| T4 | `tool/get-context-remaining` | schemas spec; real state budget caller impl |
| T5 | `tool/image-generation/{contract,impl}` | models spec; public schema/API/artifacts impl; failing history branch unchanged |
| T6 | `tool/multi-agent/{contract,impl}` | serializers spec; registration impl; host acceptance path retained |
| T7 | `tool/plan` | schema spec; state persistence impl |
| T8 | `tool/request-user-input/{contract,impl}` | serializer/KDoc spec; registration impl; absent timer is inherited |
| T9 | `tool/tool-search/{contract,impl}` | models spec; conversion/index/registration impl |
| T10 | `tool/unified-exec/{spec,impl}` | client/session/DTO spec, real Shell caller impl |
| T11 | `tool/view-image/{contract,impl}` | models spec; schema/options/codec caller impl |
| T12 | `tool/web-run` | schema spec; API caller/registration impl |
| M0 | `mcp/{contract,impl}` | settings/ports spec; owner/manager/OAuth/tool composition impl |
| M1/M2 | `mcp/{stdio,streamable-http}` | actual function implementations now in spec-only projects |

- Later `eff7b10e` is the transport/Process repair, not merely dependency renaming:
  deleted unused factory interfaces/default objects, deleted duplicated stdio body,
  moved actual transport functions/tests, and changed production assembly to consume them.
- `19bad603` retargets get-context/plan to actual AgentState spec. `d05e9cdb` retargets
  storage. The examined `821b6627..HEAD` Tool/MCP diff leaves core tool/manager/OAuth/
  MCP-tool bodies intact; service moves real default resources into constructor seams.
- Frozen DTO names/serializers and KDoc were compared with pre-split bodies.
  Historical green logs do not prove current execution or fix the inherited gaps.

## 3. Shared production and dispatcher chains

### C1 — construction, registration and resource ownership

- `io.github.stream29.kodex.agentruntime.impl.fixedTools`:
  [KodexAgentTools.kt:39–97][fixed] reads current settings/cwd, constructs real exec
  client (51–54), patch (59–63), time (65), context (66), plan (67), exec (68),
  web (70–75), view (79–82), image (86–92). This is not an isolated test factory.
- Same file [MCP/search assembly:100–127][catalogassembly] maps real MCP clients'
  `listTools()`, combines deferred local tools, rebuilds search index and flattens tools.
- `io.github.stream29.kodex.agentruntime.impl.masterRuntimeLayer`:
  [composition:68–99][runtimeassembly] creates Tool runtime (85–90); owner completion
  closes tools (92–94). `closeAll` at 206–214 closes in reverse, continuing after errors;
  construction failure closes accumulated resources at 216–230.
- `io.github.stream29.kodex.agentruntime.decorator.tool.KodexToolRuntimeImpl`:
  [routing/handling:33–113][dispatcher] merges current catalogs at 49, handles search
  specially at 63–69, dispatches actual `Tool.handle` at 105, and only completes a call
  still pending at 107–113. Host-owned calls remain pending at 86–87.
  Runtime factory at 152–164 borrows catalogs; it does not manufacture or own transports.
- `io.github.stream29.kodex.agentruntime.decorator.tool.ToolRouting`:
  [index/merge/routingNames:18–65][routing] extracts concrete names/namespace members,
  rejects duplicate executable routes at 26–28 and cross-catalog collisions at 33–38.

### C2 — model request, serializer and stable completion

- `io.github.stream29.kodex.agentstate.tool.visibleToolSpecs`:
  [visibility:24–75][visibility] direct specs are patch/time/context/exec/write/web;
  image/view deferred; plan/AskUser/search added from mode/catalog. These are actual
  request tools, not just metadata exposed by unused factories.
- `io.github.stream29.kodex.agentstate.impl.KodexAgentStateImpl`:
  [createResponse:195–203][requestcaller] passes tools at 201;
  remote compaction also passes tool specs at 316–321.
- `io.github.stream29.kodex.agentstate.tool` pending projection:
  [typed decode:45–153][decode] invokes real argument serializers for exec (62),
  write (71), web (80), image (89), view (98), input (107), subagent (116), plan (125);
  MCP namespace becomes `PendingMcpToolEvent` (133–142).
  Patch parsing is 157–175; search serializer is 187–201; invalid input becomes an
  invalid pending event at 235–244, not an alternate tool model.
- Stable-event history is the canonical completed-tool→provider projection.
  [StableToolHistory:29–98][stablehistory] serializes typed arguments/results, including
  JSON-string function outputs and success flags. Topic stable events retain image native
  content, input answers, suggestion decisions, MCP envelopes and native search output.
  Tool handlers are not permitted a second parallel protocol/state owner.

### C3 — host-owned interaction, not missing Tool implementations

- `io.github.stream29.kodex.cli.agent.RequestUserInputViewModelImpl`:
  [factory:33–46; submit:107–170; synchronize:172–180][inputhost].
  Submission persists `completeToolCall` at 145–152 and resumes at 155; close at
  183–190 releases observation. This is the production owner for AskUser completion.
- `io.github.stream29.kodex.cli.agent.SuggestSubagentTaskViewModelImpl`:
  [factory:34–48; submit:116–166][multihost].
  Rejection at 136–137 creates no sessions. Acceptance calls `createSessions` at
  139–142, persists response at 144–151 and resumes at 154.
- Therefore input/multi-agent's payload-only specs are meaningful contracts. Missing
  a local `Tool` implementor is not itself a defect. The input timer gap is a different,
  evidenced behavioral issue.

### C4 — app-wide MCP owner and persisted settings

- `io.github.stream29.kodex.rpc.server.BackendServices`:
  [actual defaults:57–68][backendservices] constructs `DefaultMcpOAuthClient`,
  supplies `BackendMcpOAuthBridge(mcpOAuth::prepare)` and shares global MCP service.
- `io.github.stream29.kodex.rpc.server.withBackendGlobalState`:
  [factory:36–79][backendglobal] uses actual `McpServiceImpl` at 47–49, actual settings
  store at 69 and `McpManagerImpl` at 75–79. Shutdown at 88–94 releases manager/service
  then joins the owner. Full-settings CAS at 111–116 calls
  `McpManagerImpl.commitConfigurationChange` before validated atomic persistence.
- `io.github.stream29.kodex.mcp.impl.McpServiceImpl`:
  [defaults/owner factory:41–68,318–335][service] owns a supervisor child scope,
  actual HTTP client (46) and actual Process client (47), reconciles enabled settings,
  constructs `McpClientOwner` at 180–203, publishes clients at 167.
- `io.github.stream29.kodex.mcp.impl.McpClientOwner`:
  [connect/catalog:221–259,295–310][clientowner] builds SDK `Client` at 228,
  connects at 236, reads capability-gated paginated tools at 301–310.
  `McpClientImpl` at 45–71 exposes generation-fixed projected tools; loss preserves
  catalog while state gates call execution.
- `io.github.stream29.kodex.mcp.impl.McpToolImpl`:
  [metadata/call/result:31–105][mcptool] projects SDK input schema (45), wraps
  structured output schema (46), calls original remote name (71) through owner
  admission (67), converts `CallToolResult` preserving fields (79,140–157).
  Per-tool close is intentionally no-op at 92: app-wide owner owns the transport.
- `io.github.stream29.kodex.mcp.impl.McpManagerImpl`:
  [state/commands/login:56–110,113–280][manager] derives sanitized state from settings,
  service and auth; store transformations are authoritative, not a second settings store.
  Prepared login persists before callback, awaits completion, identity-checks commit;
  finally closes attempt in `NonCancellable` at 273.

## 4. Per-topic schema → registration → actual caller → tests

FQCN notation: every `tool.*` below expands to `io.github.stream29.kodex.tool.*`;
every `mcp.*` expands to `io.github.stream29.kodex.mcp.*`. C1–C4 give full production
factory/caller chains. Tests in section 8 are source assertions only.

| Trace | Actual declaration and implementation | Production and assertion coverage |
| --- | --- | --- |
| T0 | `tool.contract.Tool` / `typedTool` [13–47][toolcontract]; `ToolName` [13–35][toolname] | C1 routes exact specs; C2 canonical pending/stable events. typedTool rejects wrong pending type; resource-free adapter close is intentional. ToolContractTest checks names/matching. |
| T1 | `tool.builder.JsonToolHandlerResult` [10–28][builderspec]; `textTool` [89–126][builderimpl], `FunctionOutputTool` 153–195 | Real T3/T4 text handlers use algebra; decode failure emits canonical failure; cancellation is not fabricated success. Builder tests cover adapters. `jsonTool`/`functionOutputTool` currently only test callers; see A1. |
| T2 | `tool.applypatch.ApplyPatchGrammar` [8–29][patchspec]; `ApplyPatchTools` [18–78][patchimpl]; `ApplyPatchToolClient.apply` [11–19][patchclient] | C1 factory→typed pending patch→actual patch client at Tools:63. C2 uses grammar/parser. Topic test only proves metadata; actual patch utility belongs to other-utils review. |
| T3 | `tool.currenttime.CurrentTimeParametersSchema` / output [6–11][timespec]; `CurrentTimeTools` [18–39][timeimpl]; `CurrentTimeToolClient` [10–34][timeclient] | C1 registers `clock` backed text handler; fixed-clock tests cover sentence/UTC. Output schema is code-mode metadata, not promised ordinary JSON result. |
| T4 | `tool.getcontextremaining.GetContextRemainingParametersSchema` / output [9–25][contextspec]; `getContextRemainingTool` [12–27][contextimpl] | C1 binds actual AgentState; calls `tokensUntilCompaction` at 19 then textTool. Real state/in-memory behavior test asserts 40 remaining from threshold800/current760. No live model usage measured. |
| T5 | `tool.imagegeneration.ImageGenToolArguments` / `GeneratedImageOutput` [12–32][imagespec]; `ImageGenerationTools.createTool` [59–103][imageimpl]; client [20–118][imageclient] | C1→C2→client real generate/edit API; artifact saves Base64 bytes through [GeneratedImageArtifacts:10–31][artifacts]. Save failure keeps generated content; cancellation rethrows. History-edit B1-I1. |
| T6 | `tool.multiagent.SuggestSubagentTaskArgs` / response [8–60][multispec]; `SuggestSubagentTaskTools` [12–19][multiimpl] | C2 typed event→C3 host confirmation→real createSessions. Exact accepted/rejected/null-feedback serializers tested; no invented extra host interface. |
| T7 | `tool.plan.UpdatePlanParametersSchema` [6][planspec]; `updatePlanTool` [20–44][planimpl] | `appendPlanUpdate` at 31 already completes stable event; C1 dispatcher guard prevents duplicate completion. Schema + actual State persistence test, not full renderer acceptance. |
| T8 | `tool.requestuserinput.RequestUserInputArgs` [8–51][inputspec]; `RequestUserInputTools` [9–19][inputimpl] | C2→C3 manual host answer→keyed response serialization. Optional timeout still advertised but not consumed: B1-I2. |
| T9 | `tool.toolsearch.ToolSearchDocument`, `SearchToolCallParams`, `ToolSearchResult` [18–78][searchspec]; `ToolSearchEngine.search` [8–53][searchengine] | C1 builds generic search index, C2 exposes native search; dispatcher emits native stable event. [Conversion:17–51][searchconversion] retains source/deferred specs, coalesces namespace results, ignores unsupported hosted/freeform shapes. Tests cover invalid query/limit, catalog/coalescing/loadability; D-CAP and A1. |
| T10 | `tool.unifiedexec.UnifiedExecClient` / session [15–78][execspec]; argument/output DTOs [31–93][execmodels]; `UnifiedExecToolClient` [35–218][execimpl] implements real client | C1→createTools→exec/write; factory191–199 calls genuine `scope.ShellClient()`. exec51 starts command, registry59, read79; final read117–131 removes completed session. Wrapper [109–121][exectools] adds shared-client close responsibility. Tests limitations below. |
| T11 | `tool.viewimage.ViewImageToolArguments` / `ViewImageToolOutput` [13–36][viewspec]; `ViewImageTools` [18–46][viewimpl]; `ViewImageToolClient.view` [17–45][viewclient] | C1/C2 default options agree; regular image file becomes native image completion. [ViewImageToolOptions/schema:6–37][viewschema] remain public impl metadata, not an empty spec. Default hides unsupported environment/original capabilities; opt-in combos need separate validation. |
| T12 | `tool.webrun.WebRunParametersSchema` [153–221][webspec]; `WebRunTools` [127–158][webimpl]; `WebRunToolClient.run` [20–31][webclient] | C1 settings/model/session→pending SearchCommands→real `OpenAiClient.search` at 22; canonical stable output. Schema and medium-default SearchRequest tests; endpoint untested here. |

## 5. MCP real transport, authentication, cancellation and close

### M0 — meaningful contracts and ownership

- `mcp.contract.McpClient` [16–39][mcpclientcontract] documents healthy-only remote calls,
  generation-fixed catalog, reconnect replacement after success and retention on failure.
  State variants/reason enum at 47–70 retain stable serializable shapes.
- `mcp.contract.McpService` [12–35][mcpservicecontract] owns clients and sanitized auth;
  explicit invalidate differs from successful refresh. Settings specs contain real
  serialized HTTP/stdio/OAuth models; `McpSecret` [23–26][mcpsettings] serializes its
  string but redacts `toString`, and stdio working_directory retains Path serializer.
- Service identity [378–396][service] intentionally excludes rotating tokens; runtime
  credential refresh does not manufacture another configuration owner.
  `authorizationToken` at 216–282 reads latest by-name settings, serializes refresh via
  per-server mutex, conditionally persists only matching OAuth generation, rethrows cancel.
- Per-server derived HTTP client at 185–190 obtains token through the owner; release at
  202 belongs to that server. Whole-service cleanup at 302–314 runs `NonCancellable`,
  closes owners then shared Process client; shared HTTP client follows owner completion.

### M1 — sole Process-backed stdio startup

- FQCN extension `io.github.stream29.kodex.mcp.stdio.openMcpStdioTransport`:
  [McpStdioTransport.kt:19–54][stdio] is an **actual function in spec**, not an unused
  abstract factory. It calls `ProcessClient.start` at 33 with exact executable,
  argument list, cwd and raw environment values at 34–38.
- Production chain:
  C4 `McpServiceImpl` factory318→constructor default
  `io.github.stream29.kodex.utils.processclient.ProcessClient` actual factory→
  service `createOwner`198→`openTransport`206–214→stdio branch213→extension30→
  Process spec `start`→platform process. [Real Process factory:24][processfactory].
- **One startup**, no second shell interpretation, no `ProcessClientSpec`,
  `McpStdioTransportFactory` or old private duplicate body in current production.
  Shell is used separately for unified-exec: [real Shell factory:20][shellfactory].
  MCP stdio must not take a Shell text-buffer path.
- SDK transport receives raw stdout source/stdin sink/stderr at 43–46.
  Private adapters56–80 bridge real SDK raw IO types; removing them without equivalent
  plumbing breaks framing. Construction failure at 50–52 closes acquired process.
- `ProcessOwnedTransport` at 84–124 closes Process once via atomic gate121–124,
  also on remote transport close106–110; explicit close98–104 closes both.
  KDoc19–29 now specifies ownership/termination, Process failure and cancellation.
- Contract test [30–102][stdiotest] proves mapping including literal `$(not-a-shell)`,
  raw newline frame, close twice→one process termination/one callback, start failure
  identity and construction-failure cleanup. Fake source/sink is not a child process.
- Composition sentinel [33–85][transporttest] reaches this function through actual
  service constructor and captures `ProcessCommand`; injected startup failure does
  not establish successful live handshake. JVM service fixture and JS direct-SDK
  fixture exist but were not run; JS fixture is not CLI MCP-service assembly.

### M2 — real HTTP functions, borrowing and one-retry auth

- FQCN functions `io.github.stream29.kodex.mcp.streamablehttp.McpStreamableHttpClient`,
  `withMcpAuthorization`, `openMcpStreamableHttpTransport`:
  [McpStreamableHttp.kt:20–87][http].
- C4 service default46→actual scope-bound `McpStreamableHttpClient`28 (requires Job29,
  installs SSE33, closes HTTP on owner completion35–36).
  Owner's derived view185→`withMcpAuthorization`49→openTransport HTTP branch211→
  `openMcpStreamableHttpTransport`78→SDK transport borrowing the passed HTTP client.
- Auth gets current token with `forceRefresh=false`54; on 401 does one forced refresh62,
  cancels first response63–65 and retries once68. Static configured headers remain
  passed at85. KDoc42–47 now describes real behavior, not hidden only behind an impl.
- KDoc72–76 says transport close does not close borrowed HTTP client.
  SDK [StreamableHttpClientTransport.closeResources:242–246][sdkhttpclose] cancels SSE
  and transport scope, not borrowed client. Service owner later releases derived client.
- HTTP MockEngine tests [22–121][httptest] check borrowed-client survival, owner
  cancellation, one retry `[false,true]`, token values and isolated same-endpoint headers.
  Composition sentinel [87][transporttesthttp] captures URL/headers via real service.
  This is assembly evidence, not network/SSE/server interoperability certification.

### Cancel/close evidence and limits

- Owner call [182–205][clientowner] uses healthy-state admission/read lease and
  cancellable SDK request. Reconnect writer119 closes previous connection; owner
  close207–219 sets Closed and closes SDK client then releases server resource.
- SDK pin `53f717661712d618cd4822d1760392e903989ead`:
  [Protocol:830–900][sdkcancel] removes pending/progress state, remembers canceled IDs,
  sends peer cancel where applicable, performs cancellation cleanup in `NonCancellable`
  at887, does not replace original cancellation with send failure, rethrows895.
  Initialization has special no-peer-cancel handling; SDK default request timeout exists.
  The application having no added timeout must not be read as “no SDK timers.”
- SDK stdio [closeResources:271–277][sdkstdioclose] stops/closes read sources/joins under
  `NonCancellable`; writer cleanup also closes output. Those source paths are real,
  but actual process-tree termination on every target remains platform-line/runtime work.
- OAuth `mcp.impl.DefaultMcpOAuthClient` [62–145][oauth] implements login-attempt factory
  and token refresher, performs real discovery/PKCE/code exchange/refresh.
  Separate scope HTTP client at398–401 is OAuth work, not a second MCP transport startup.
  Prepared attempt414–431 closes callback in finally; callback473–541 validates state,
  owns loopback server and stops it on completion. No live OAuth/security certification.

## 6. Confirmed findings: inherited, not migration drift

### B1-I1 — advertised history-image edit always fails

- Contract FQCN `io.github.stream29.kodex.tool.imagegeneration.ImageGenToolArguments`:
  [KDoc/field:9–19][imagespec] offers `num_last_images_to_include`.
- Registration FQCN `io.github.stream29.kodex.tool.imagegeneration.ImageGenerationTools`:
  [description:23–38][imageimpl] advertises previous-conversation images at33–34;
  schema includes the property. C2 decodes this serializer at89.
- Actual factory/caller chain:
  C1 [fixedTools:86–92][imagefactory] constructs `ImageGenerationToolClient` with API/cwd,
  **no history provider**→`ImageGenerationTools.createTool`59→handle68→
  `ImageGenerationToolClient.run`27→`requestFor`82–85.
  [History branch][imagehistory] validates count then unconditionally throws
  `ImageGenerationToolException("... requires conversation history from the agent loop")`.
- Impact: a valid advertised history-edit request cannot succeed in production.
  File-reference edits and fresh generation remain real; the whole payload spec is not fake.
- History: at `6606efe9^`,
  `tool/image-generation/impl/src/commonMain/kotlin/io/github/stream29/kodex/tool/imagegeneration/ImageGenerationToolClient.kt:82–85`
  has the same branch. No migration introduced this behavior.
- Test [ImageGenerationToolClientTest:45–55][imagehistorytest] **expects the failure**.
  It characterizes the limitation, not fulfillment of the advertised history capability.
- Minimal acceptance experiment: retained synthetic conversation images + fake image API,
  routed through actual composition/client, assert selected history reaches edit request.
  Need a real history binding or separately authorized advertising/policy change;
  do not silently delete frozen wire field or add a new tool framework.

### B1-I2 — optional auto-resolution window has no timer consumer

- Contract FQCN `io.github.stream29.kodex.tool.requestuserinput.RequestUserInputArgs`:
  [KDoc/field:10–16][inputspec] distinguishes null/manual wait from an auto-resolution
  window. Impl [schema:48–52][inputschema] advertises number `autoResolutionMs`,
  60000–240000, and [description:9–11][inputimpl] promises the window.
- Actual chain: C2 serializer at107 preserves value→C3 factory33→pending synchronization
  [172–180][inputhost]→only manual `submit`107–170→complete145→resume155.
  No deadline/timer reads this property. Repository reference search found DTO/schema,
  history retention/display and serializer tests, not an auto-resolution consumer.
- Impact: setting a valid window does not cause timeout completion/resumption through
  the production host path. Null/manual waiting is functional.
- History: `6606efe9^` file
  `tool/request-user-input/contract/src/commonMain/kotlin/io/github/stream29/kodex/tool/requestuserinput/RequestUserInputModels.kt:10–16`
  has the same promise/property; current host rewrite did not install a timer.
  This is inherited, not new schema drift.
- Tests [44–73][inputtest] cover omitted timeout/serializer/keyed answers, not elapsed time.
- Minimal experiment: fake clock, bind pending call with60000, no submit, advance to
  deadline and assert persisted completion/resume exactly once. Desired generated-answer
  semantics still need confirmation; do not invent a new protocol timeout policy,
  clamp rules or remove `autoResolutionMs`.

### B1-I3 — legal raw MCP names collapse to duplicate executable routes

- Contract/config path accepts distinct raw server names `a-b` and `a_b`.
  [McpServerDraft.validatedName:4][draft] rejects blank, not this collision;
  [validateMcpConfigurationUpdate / validateNames:11,43–46][validation] checks
  trimmed uniqueness, not model-name uniqueness.
- FQCN `io.github.stream29.kodex.mcp.impl.McpToolImpl`:
  [namespace/tool projection:38–49][mcptool] uses `mcp__` plus normalized server name
  and normalized tool name; [toModelToolName:108–118][normalize] maps punctuation to `_`.
  Thus both servers' `echo` produce **`mcp__a_b.echo`**. Within one server,
  `x-y` and `x_y` also collapse.
- Factory→caller evidence:
  C4 service creates owners→SDK catalog→`McpClientImpl` projection52–60
  [raw-name distinctBy:53][clientprojection] does not prevent normalized collisions→
  C1 actual `mcpToolsState/listTools`100–127→C1 Tool runtime at49→
  [ToolRouting.index:26–28][routing] throws on duplicate executable route.
- Impact: catalog routing fails before processing a pending tool, potentially even a
  local pending tool. This is a deterministic source-level collision/throw chain,
  not a claim that this audit connected to and crashed a real server.
- History: `6606efe9^` file
  `mcp/impl/src/commonMain/kotlin/io/github/stream29/kodex/mcp/impl/McpTool.kt:108–118`
  has the same normalizer; projection/raw-name catalog behavior predates split.
  Core McpTool body moved intact. No new provider naming restrictions assumed.
- Existing fixtures use ordinary distinct names; no collision regression assertion found.
- Minimal offline fixture: two projected clients with `a-b`/`a_b` + `echo`, and one client
  with `x-y`/`x_y`; invoke actual ToolRouting index and assert an agreed unique routable
  representation/admission outcome. Resolve disambiguation at existing projection/
  validation boundary; no additional registry/dispatcher framework.

## 7. Policy debts, suspected races and public surface

### D-PROTOCOL — fixed-version wording vs supported-version acceptance

- [MCP client checklist][mcpchecklist] specifies fixed `2025-11-25` / no negotiation.
  C4 uses ordinary SDK `Client.connect`.
- At pinned SDK [Client:202–218][sdkversion], proposed version is latest but response
  is accepted if in supported versions; [constants:18–23][sdkversions] include older
  versions. No app strict-equality gate exists in reviewed connect path.
- [Service fixture initialize response:571][oldversionfixture] and stdio fixture return
  `2025-03-26`. They do not test rejection of a supported older reply.
- Inherited policy discrepancy, explicitly separated by [closure planning][planning]
  from migration closure. Do not label older acceptance a new transport regression.
- Gate: confirm whether “fixed” means proposed version or mandatory accepted version.
  Minimal fake-transport initialize matrix latest/supported-old/unsupported through
  existing owner. No protocol-policy repair authorized here.

### D-CAP — unsupported-model deferred fallback has no capability contract

- [Tool-search checklist][searchchecklist] describes promoting deferred tools when
  model/provider lacks search support.
- Actual [ModelInfo:39–62][modelinfo] has no capability field. C2 visibility adds
  native tool search/deferred local tools without such branch, and actual request caller
  uses that list. Default tool-search behavior is inherited.
- This proves missing capability discrimination, **not** a particular provider's current
  failure. Closure planning explicitly defers capability/wire policy.
- Gate: agree capability input and compatibility policy, then recorded unsupported-model
  fixture through actual visibility/request projection. Do not invent a capability field
  or use live provider behavior as this audit's evidence.

### B2-R1 — refresh may publish old catalog after successful reconnect

- `mcp.impl.McpClientOwner.refresh` [155–180][clientowner] reads under reader lease,
  releases it172, then publishes175. Reconnect112–149 closes/replaces under writer,
  releases writer before catalog publication142; refresh does not hold reconnect mutex.
- Service [publishCatalog:292–300][catalogpublish] checks owner identity297, not connection
  revision. Same owner can therefore receive both publications.
- Suspected ordering: old refresh finishes reading, stalls before publish; reconnect
  publishes new catalog; old refresh resumes and publishes old catalog last.
  No controlled interleaving was run, so not B1. Behavior is inherited.
- Minimal reproducer: existing internal owner + fake transport + publish callback barriers;
  return distinguishable old/new catalogs, release old publication last, inspect final
  generation after reconnect completes. Clarify desired concurrent ordering first.
  If confirmed, constrain existing publication/lease or generation check, not a new layer.

### B2-R2 — auth-blocked view temporarily retains a still-healthy tool owner

- Service [reconcile:146–175][blockedreconcile] constructs blocked client146–150 with
  prior tools, publishes it167, then closes retired owners174–175.
- `mcp.impl.McpAuthenticationBlockedClient` [74–85][clientowner] has AuthenticationBlocked state,
  but retained `McpToolImpl` still calls original owner's healthy-state gate182–205.
  Owner becomes Closed only when retirement reaches close210.
- Suspected gap: observer sees blocked view and invokes retained tool before retirement
  acquires writer; old remote call may be admitted or report a different unavailable reason.
  Existing blocked test observes settled state, not this gap. Not B1 without controlled
  evidence/clarified admission-cutover semantics. Inherited ordering.
- Minimal fake-owner/service test: barrier at blocked publication and retiring writer,
  invoke retained tool, record remote request count/result, then retire. Keep one owner;
  do not add another auth facade/parallel tool state.

### B2-KDOC — public failure/lifecycle contract remains incomplete

- Normative transport KDoc is substantially repaired: stdio Process/cancel/ownership
  and HTTP auth/borrowing now live beside actual spec functions.
  Unified client [34–49][execspec] has explicit failure docs.
- `mcp.contract.validatedName` / `McpServerDraft.toConfiguration`
  [4–79][draft] have real blank-value/secret-retention failures but no operation-level
  normative `@throws`. `McpManager` [265–280][managercontract] lists commands without
  per-operation failures although impl add/edit/reconnect have validation/missing-name
  failures. Factory/refresher ports at234/241 omit detailed attempt/failure lifecycle.
- This is an observed documentation omission, **not** proof of a new runtime fault.
  Exact intended replaceability/failure matrix still needs review, so B2 not a blanket
  behavioral B1. The omissions predate migration.
- Gate: compare add/edit/reconnect/login/import invalid inputs and cleanup against
  current implementations/tests; document stable failure category and ownership where
  intended. Do not freeze incidental exception strings as new wire API.

### Public impl surface not entirely represented in spec

- Concrete tool clients, registration objects, image/view/schema metadata and builders
  remain public impl declarations. They are actually consumed through Tool/DTO specs;
  no universal “every public impl class needs an interface” rule is valid.
- MCP concrete APIs outside `McpManager` spec include
  `mcp.impl.McpManagerImpl.commitConfigurationChange` [329][managercas],
  `loginWithPreparedCallback` [187][managerprepared],
  `validateMcpConfigurationUpdate` [11][validation],
  `DefaultMcpOAuthClient.prepare` [77][oauth] and
  `PreparedMcpOAuthLogin` [405–412][prepared].
- These are **not unused factories**:
  BackendGlobalState CAS111–116 calls concrete commit API; [BackendOAuth:102–108][backendoauth]
  calls concrete prepared login; [BackendMcpOAuthBridge:36–49][backendbridge] consumes
  PreparedMcpOAuthLogin from actual Default client.
- Record this composition-specific surface as replaceability limitation, consistent with
  [RPC-MCP/full-config ownership checklist][rpcchecklist], not duplicate settings authority
  or a reason to add an empty port. Backend integration reviewer should verify frozen RPC
  lifecycle independently. Legacy import preview/apply is still a real manager path;
  it is not the current full-settings RPC CAS path.

### D-A1 / minimal ablation review

- `tool.toolsearch.ToolSearchExecution` [57–63][searchspec] has no consumers; actual
  OpenAI search wire discriminators are owned by OpenAI serializers, not this enum.
  Inherited unused public enum is an API-removal debt, not a fake entire spec.
  Gate: public compatibility decision, then remove just enum if allowed.
- Builder `jsonTool`22–73 and `functionOutputTool`140–151 currently have only test callers;
  production textTool really uses FunctionOutputTool. Consider shrinking test-only public
  entrypoints if API policy permits; retain actual shared decode/completion mechanism.
- Tool-search schema construction duplicates default schema/helper; custom metadata
  `defaultLimit` does not configure engine default8. Current production uses8 consistently.
  A configurable public default needs an explicit fixture before promising configuration.
  Small consolidation can preserve schema/default wording without an engine framework.
- Service and Manager each map persisted OAuth configuration to sanitized auth state
  ([Service:404–411][authmapping], [Manager:604–612][managerauthmapping]).
  This is small inherited pure mapping duplication, not two writable auth authorities.
  A shared pure helper, if justified, is enough.
- Do **not** ablate DTO/serializer-only specs, raw SDK IO bridges, supervisor owners,
  identity/catalog snapshots or exec close wrapper: they have concrete responsibilities.
  Removing fake factory wrappers was appropriate; replacing them with another unused
  registry/interface would recreate the prior blocker.

## 8. Frozen wire, serializers and test limits

### Wire/behavior comparison

- Preserved exec field names (`cmd`, `yield_time_ms`, `session_id`, `chunk_id`, etc.);
  image reference/history fields and `output_hint`; view path/detail/environment and
  `image_url`; input camelcase `autoResolutionMs` and keyed answers; plan statuses.
- Suggestion response uses explicit accepted/rejected serializers and ALWAYS-encoded
  feedback/decision, including null feedback. It does not invent a parallel sealed
  type discriminator in the native tool response.
- MCP `streamable_http`/`stdio`, OAuth union, secret string serialization and Path
  working_directory shape remain real persisted contracts, not class-name documentation.
- MCP wrapped output preserves `content`, `structuredContent`, `isError`, `_meta`;
  [McpTool:126–185][mcpoutput] schema/envelope/failure handling retains stable unavailable
  categories. Details are logged rather than becoming a new serializable state string.
- Time/context produce human text in normal function completion. OpenAI
  [ToolSpec.outputSchema:62–82][outputschema] is transient code-mode metadata;
  expecting JSON from these tools would be a false schema-drift finding.
- Stable completion is the canonical output owner; typed reserialization is not a promise
  of arbitrary unknown JSON byte preservation. Image persistence paths are stable artifact
  metadata, not newly added provider response fields. No frozen wire repair made here.

### Test assertions actually inspected (none executed)

| Domain | Source evidence | What it establishes / limitation |
| --- | --- | --- |
| Contract/builder | [ToolContractTest][tcontract]; [JsonToolBuilderTest][tbuilder] | Names, typed JSON/custom/raw text completion and failure adapters. No resource/platform execution proof. |
| Patch | [ApplyPatchToolsTest:13][tpatch] | Exact freeform metadata; not real fs application or backend rendering. |
| Time | [CurrentTimeToolsTest:17–37][ttime] | Fixed clock, metadata and human text. |
| Context | [behavior test:38–72][tcontext] | Actual State/in-memory threshold calculation with fake provider/catalog; no provider token measurement. |
| Image | [client:45–55][imagehistorytest]; [artifact tests][tartifacts]; [tool tests][timage] | Fake API/filesystem/native completion/artifacts; expected history failure remains B1, not green capability coverage. |
| Subagents | [SuggestSubagentTaskToolsTest:12–47][tmulti]; C3 | Schema/decision/null-feedback wire plus separately read real host calls; not end-to-end backend session creation. |
| Plan | [UpdatePlanToolTest:35][tplan] | Actual State single persisted completion with fake API; C1 guard read separately, not renderer acceptance. |
| Input | [RequestUserInputToolsTest:44–73][inputtest]; C3 | Optional field/keyed response, manual production host path; no deadline/elapsed-time test found. |
| Search | [ToolSearchToolsTest:26–161][tsearch] | Sources/coalescing/loadable results/invalid args; no unsupported-model fallback. Generic index utility/Lucene internals outside ownership. |
| Unified exec | [realIoTestConfig:53–55][texecdisabled]; [session-control suite:576–612][texecsession] | **Many real exec/poll/burst/PTY/cancel tests use `isEnabled=false`.** Separate RealTime final-read/closed-input suite exists without that disabled config; none run here. Owner-cancel test with no live process cannot prove process termination. |
| View | [ViewImageToolClientTest:56–91][tview]; [schema tests][tviewschema] | File/image fixtures, missing file and optional capability conditions; no real environment resolution or all codec actuals. |
| Web | [WebRunToolsTest:25][tweb]; [RequestControls:12][twebcontrols] | Exact schema and default medium SearchRequest; no endpoint request/interoperability. |
| MCP config | [McpServerConfigurationTest:10–42][tmcpconfig] | HTTP/stdio roundtrip and initialized-secret redaction; not full file/YAML/CAS runtime. |
| MCP manager | [McpManagerImplTest][tmanager] | Fake store/client/login/import/reconnect paths; does not replace real backend CAS/remote frontend test. |
| MCP schemas | [McpSchemaProjectionTest][tmcpschema] | Rich input schema and result wrapper/envelope preservation; no provider schema acceptance. |
| MCP composition | [McpTransportCompositionTest:33,87][transporttest] | Real service defaults/seams reach Process command / HTTP URL/header; injected errors and MockEngine are not successful live protocol. |
| stdio | [McpStdioContractTest:30–102][stdiotest]; [JVM Io fixture:32][tstdiojvm]; [JS direct fixture:26][tstdiojs] | Raw fake framing/close/startup failures; JVM service fixture exists, JS is SDK direct. No fixture run/all-target pass. |
| HTTP | [McpStreamableHttpTest:22–121][httptest] | Borrow/owner cancellation/one401 retry/per-server auth isolation with MockEngine. Not live SSE/session/delete/redirect interoperability. |
| MCP service | [McpServiceImplIoTest:64–353][tservice] | Source includes loopback generations, connection loss, invalidation, blocked state and token refresh. No collision or controlled B2 barrier tests found; suite not run. Older handshake fixture not strict-version rejection evidence. |
| OAuth | [McpOAuthIoTest:48,115,173][toauth] | Code/refresh resource, metadata/register and rejection without PKCE source assertions. No live login or exhaustive security/cancellation failure coverage. |

## 9. U subpaths and handoff

- No assigned build project omitted. U **subpaths**, not hidden project passes:
  actual all-platform process-tree/PTY/cwd/cancel handoff, image codecs, full Lucene/SDK
  internals, all Ktor redirect/credential-leak cases, OAuth cryptographic arithmetic and
  exhaustive failure/cancel interleavings, live provider/server protocol, CLI rendering.
- External boundaries checked at actual pins: KotlinMcpSdk `53f71766`, LuceneKmp
  `549f8afc`; their whole codebases are not certified by these 29 project rows.
- Main must independently reproduce B1 source chains, decide inherited-policy disposition,
  and schedule minimal offline B2 fixtures before choosing repair. Cross-line root causes
  (host input, dispatcher, backend settings) should be de-duplicated, not tallied twice.
- Final source pin remains `6b7129fa0e719dde8f88a0034b9d572679984a03`,
  branch `refactor/spec`, inner worktree clean. Only this child report was modified by
  this audit. No source fixes, test runs or policy changes are implied by REVIEW READY.

## Evidence URI index

[boundaries]: file:///home/stream/ACodeSpace/push/BuildKodex/checklist/spec-impl-module-boundaries.md#L1
[planning]: file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-07-plan-spec-impl-closure-and-consistency.md#L1
[settings]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/settings.gradle.kts#L34
[toolcontract]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/spec/contract/src/commonMain/kotlin/io/github/stream29/kodex/tool/contract/Tool.kt#L13
[toolname]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/spec/contract/src/commonMain/kotlin/io/github/stream29/kodex/tool/contract/ToolName.kt#L13
[builderspec]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/spec/builder/src/commonMain/kotlin/io/github/stream29/kodex/tool/builder/JsonToolBuilderSpec.kt#L10
[builderimpl]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/builder/src/commonMain/kotlin/io/github/stream29/kodex/tool/builder/JsonToolBuilder.kt#L89
[patchspec]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/spec/apply-patch/src/commonMain/kotlin/io/github/stream29/kodex/tool/applypatch/ApplyPatchToolSpec.kt#L8
[patchimpl]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/apply-patch/src/commonMain/kotlin/io/github/stream29/kodex/tool/applypatch/ApplyPatchTools.kt#L18
[patchclient]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/apply-patch/src/commonMain/kotlin/io/github/stream29/kodex/tool/applypatch/ApplyPatchToolClient.kt#L11
[timespec]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/spec/current-time/src/commonMain/kotlin/io/github/stream29/kodex/tool/currenttime/CurrentTimeSchemas.kt#L6
[timeimpl]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/current-time/src/commonMain/kotlin/io/github/stream29/kodex/tool/currenttime/CurrentTimeTools.kt#L18
[timeclient]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/current-time/src/commonMain/kotlin/io/github/stream29/kodex/tool/currenttime/CurrentTimeToolClient.kt#L10
[contextspec]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/spec/get-context-remaining/src/commonMain/kotlin/io/github/stream29/kodex/tool/getcontextremaining/GetContextRemainingSchemas.kt#L9
[contextimpl]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/get-context-remaining/src/commonMain/kotlin/io/github/stream29/kodex/tool/getcontextremaining/GetContextRemainingTool.kt#L12
[imagespec]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/spec/image-generation/src/commonMain/kotlin/io/github/stream29/kodex/tool/imagegeneration/ImageGenerationToolModels.kt#L9
[imageimpl]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/image-generation/src/commonMain/kotlin/io/github/stream29/kodex/tool/imagegeneration/ImageGenerationTools.kt#L23
[imageclient]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/image-generation/src/commonMain/kotlin/io/github/stream29/kodex/tool/imagegeneration/ImageGenerationToolClient.kt#L20
[imagehistory]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/image-generation/src/commonMain/kotlin/io/github/stream29/kodex/tool/imagegeneration/ImageGenerationToolClient.kt#L82
[imagefactory]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-runtime/impl/composition/src/commonMain/kotlin/io/github/stream29/kodex/agentruntime/impl/KodexAgentTools.kt#L86
[artifacts]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/image-generation/src/commonMain/kotlin/io/github/stream29/kodex/tool/imagegeneration/GeneratedImageArtifacts.kt#L10
[multispec]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/spec/multi-agent/src/commonMain/kotlin/io/github/stream29/kodex/tool/multiagent/SuggestSubagentTaskModels.kt#L8
[multiimpl]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/multi-agent/src/commonMain/kotlin/io/github/stream29/kodex/tool/multiagent/SuggestSubagentTaskTools.kt#L12
[planspec]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/spec/plan/src/commonMain/kotlin/io/github/stream29/kodex/tool/plan/PlanSchemas.kt#L6
[planimpl]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/plan/src/commonMain/kotlin/io/github/stream29/kodex/tool/plan/UpdatePlanTool.kt#L20
[inputspec]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/spec/request-user-input/src/commonMain/kotlin/io/github/stream29/kodex/tool/requestuserinput/RequestUserInputModels.kt#L8
[inputimpl]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/request-user-input/src/commonMain/kotlin/io/github/stream29/kodex/tool/requestuserinput/RequestUserInputTools.kt#L9
[inputschema]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/request-user-input/src/commonMain/kotlin/io/github/stream29/kodex/tool/requestuserinput/RequestUserInputSchemas.kt#L48
[searchspec]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/spec/tool-search/src/commonMain/kotlin/io/github/stream29/kodex/tool/toolsearch/ToolSearchModels.kt#L18
[searchengine]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/tool-search/src/commonMain/kotlin/io/github/stream29/kodex/tool/toolsearch/ToolSearchEngine.kt#L8
[searchconversion]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/tool-search/src/commonMain/kotlin/io/github/stream29/kodex/tool/toolsearch/ToolSearchDocumentConversion.kt#L17
[execspec]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/spec/unified-exec/src/commonMain/kotlin/io/github/stream29/kodex/tool/unifiedexec/UnifiedExecClient.kt#L15
[execmodels]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/spec/unified-exec/src/commonMain/kotlin/io/github/stream29/kodex/tool/unifiedexec/UnifiedExecModels.kt#L31
[execimpl]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/unified-exec/src/commonMain/kotlin/io/github/stream29/kodex/tool/unifiedexec/UnifiedExecToolClient.kt#L35
[exectools]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/unified-exec/src/commonMain/kotlin/io/github/stream29/kodex/tool/unifiedexec/UnifiedExecTools.kt#L109
[viewspec]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/spec/view-image/src/commonMain/kotlin/io/github/stream29/kodex/tool/viewimage/ViewImageModels.kt#L12
[viewimpl]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/view-image/src/commonMain/kotlin/io/github/stream29/kodex/tool/viewimage/ViewImageTools.kt#L18
[viewclient]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/view-image/src/commonMain/kotlin/io/github/stream29/kodex/tool/viewimage/ViewImageToolClient.kt#L17
[viewschema]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/view-image/src/commonMain/kotlin/io/github/stream29/kodex/tool/viewimage/ViewImageSchemas.kt#L6
[webspec]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/spec/web-run/src/commonMain/kotlin/io/github/stream29/kodex/tool/webrun/WebRunSchemas.kt#L153
[webimpl]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/web-run/src/commonMain/kotlin/io/github/stream29/kodex/tool/webrun/WebRunTools.kt#L127
[webclient]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/web-run/src/commonMain/kotlin/io/github/stream29/kodex/tool/webrun/WebRunToolClient.kt#L20
[fixed]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-runtime/impl/composition/src/commonMain/kotlin/io/github/stream29/kodex/agentruntime/impl/KodexAgentTools.kt#L39
[catalogassembly]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-runtime/impl/composition/src/commonMain/kotlin/io/github/stream29/kodex/agentruntime/impl/KodexAgentTools.kt#L100
[runtimeassembly]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-runtime/impl/composition/src/commonMain/kotlin/io/github/stream29/kodex/agentruntime/impl/KodexAgentRuntimeComposition.kt#L68
[dispatcher]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-runtime/impl/decorator/tool/src/commonMain/kotlin/io/github/stream29/kodex/agentruntime/decorator/tool/KodexToolRuntimeImpl.kt#L33
[routing]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-runtime/impl/decorator/tool/src/commonMain/kotlin/io/github/stream29/kodex/agentruntime/decorator/tool/ToolRouting.kt#L18
[visibility]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-state/impl/state/src/commonMain/kotlin/io/github/stream29/kodex/agentstate/tool/KodexVisibleToolSpecs.kt#L24
[decode]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-state/impl/state/src/commonMain/kotlin/io/github/stream29/kodex/agentstate/tool/PendingToolEventProjection.kt#L45
[requestcaller]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-state/impl/state/src/commonMain/kotlin/io/github/stream29/kodex/agentstate/impl/KodexAgentStateImpl.kt#L195
[stablehistory]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-storage/spec/clean-models/src/commonMain/kotlin/io/github/stream29/kodex/agentstorage/cleanmodels/stable/StableToolHistory.kt#L29
[inputhost]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/request-user-input/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/cli/agent/RequestUserInputViewModel.kt#L107
[multihost]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/suggest-subagent-task/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/cli/agent/SuggestSubagentTaskViewModel.kt#L116
[backendservices]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/server/src/commonMain/kotlin/io/github/stream29/kodex/rpc/server/BackendServices.kt#L57
[backendglobal]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/server/src/commonMain/kotlin/io/github/stream29/kodex/rpc/server/BackendGlobalState.kt#L36
[backendoauth]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/server/src/commonMain/kotlin/io/github/stream29/kodex/rpc/server/BackendOAuth.kt#L102
[backendbridge]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/server/src/commonMain/kotlin/io/github/stream29/kodex/rpc/server/BackendOAuth.kt#L36
[managercontract]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/spec/contract/src/commonMain/kotlin/io/github/stream29/kodex/mcp/contract/McpManager.kt#L261
[mcpclientcontract]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/spec/contract/src/commonMain/kotlin/io/github/stream29/kodex/mcp/contract/McpClient.kt#L16
[mcpservicecontract]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/spec/contract/src/commonMain/kotlin/io/github/stream29/kodex/mcp/contract/McpService.kt#L12
[mcpsettings]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/spec/contract/src/commonMain/kotlin/io/github/stream29/kodex/mcp/contract/McpSettings.kt#L23
[draft]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/spec/contract/src/commonMain/kotlin/io/github/stream29/kodex/mcp/contract/McpDraftConfiguration.kt#L4
[validation]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/impl/composition/src/commonMain/kotlin/io/github/stream29/kodex/mcp/impl/McpConfigurationValidation.kt#L11
[service]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/impl/composition/src/commonMain/kotlin/io/github/stream29/kodex/mcp/impl/McpServiceImpl.kt#L41
[catalogpublish]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/impl/composition/src/commonMain/kotlin/io/github/stream29/kodex/mcp/impl/McpServiceImpl.kt#L292
[blockedreconcile]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/impl/composition/src/commonMain/kotlin/io/github/stream29/kodex/mcp/impl/McpServiceImpl.kt#L146
[authmapping]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/impl/composition/src/commonMain/kotlin/io/github/stream29/kodex/mcp/impl/McpServiceImpl.kt#L404
[clientowner]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/impl/composition/src/commonMain/kotlin/io/github/stream29/kodex/mcp/impl/McpClientImpl.kt#L87
[clientprojection]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/impl/composition/src/commonMain/kotlin/io/github/stream29/kodex/mcp/impl/McpClientImpl.kt#L52
[mcptool]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/impl/composition/src/commonMain/kotlin/io/github/stream29/kodex/mcp/impl/McpTool.kt#L31
[normalize]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/impl/composition/src/commonMain/kotlin/io/github/stream29/kodex/mcp/impl/McpTool.kt#L108
[mcpoutput]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/impl/composition/src/commonMain/kotlin/io/github/stream29/kodex/mcp/impl/McpTool.kt#L126
[manager]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/impl/composition/src/commonMain/kotlin/io/github/stream29/kodex/mcp/impl/McpManagerImpl.kt#L56
[managerprepared]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/impl/composition/src/commonMain/kotlin/io/github/stream29/kodex/mcp/impl/McpManagerImpl.kt#L187
[managercas]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/impl/composition/src/commonMain/kotlin/io/github/stream29/kodex/mcp/impl/McpManagerImpl.kt#L329
[managerauthmapping]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/impl/composition/src/commonMain/kotlin/io/github/stream29/kodex/mcp/impl/McpManagerImpl.kt#L604
[oauth]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/impl/composition/src/commonMain/kotlin/io/github/stream29/kodex/mcp/impl/McpOAuth.kt#L62
[prepared]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/impl/composition/src/commonMain/kotlin/io/github/stream29/kodex/mcp/impl/McpOAuth.kt#L405
[stdio]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/spec/stdio/src/commonMain/kotlin/io/github/stream29/kodex/mcp/stdio/McpStdioTransport.kt#L19
[http]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/spec/streamable-http/src/commonMain/kotlin/io/github/stream29/kodex/mcp/streamablehttp/McpStreamableHttp.kt#L20
[processfactory]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/process-client/impl/src/commonMain/kotlin/io/github/stream29/kodex/utils/processclient/ProcessClient.kt#L24
[shellfactory]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/shell-client/impl/src/commonMain/kotlin/io/github/stream29/kodex/utils/shellclient/ShellClient.kt#L20
[modelinfo]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/spec/models/src/commonMain/kotlin/io/github/stream29/kodex/openai/ModelCatalogModels.kt#L39
[outputschema]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/spec/models/src/commonMain/kotlin/io/github/stream29/kodex/openai/ToolSpecModels.kt#L62
[sdkcancel]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/KotlinMcpSdk/kotlin-sdk-core/src/commonMain/kotlin/io/modelcontextprotocol/kotlin/sdk/shared/Protocol.kt#L830
[sdkversion]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/KotlinMcpSdk/kotlin-sdk-client/src/commonMain/kotlin/io/modelcontextprotocol/kotlin/sdk/client/Client.kt#L202
[sdkversions]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/KotlinMcpSdk/kotlin-sdk-core/src/commonMain/kotlin/io/modelcontextprotocol/kotlin/sdk/types/common.kt#L18
[sdkstdioclose]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/KotlinMcpSdk/kotlin-sdk-client/src/commonMain/kotlin/io/modelcontextprotocol/kotlin/sdk/client/StdioClientTransport.kt#L271
[sdkhttpclose]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/KotlinMcpSdk/kotlin-sdk-client/src/commonMain/kotlin/io/modelcontextprotocol/kotlin/sdk/client/StreamableHttpClientTransport.kt#L242
[mcpchecklist]: file:///home/stream/ACodeSpace/push/BuildKodex/checklist/mcp-client.md#L1
[searchchecklist]: file:///home/stream/ACodeSpace/push/BuildKodex/checklist/tool-search.md#L1
[rpcchecklist]: file:///home/stream/ACodeSpace/push/BuildKodex/checklist/rpc-mcp.md#L1
[tcontract]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/spec/contract/src/commonTest/kotlin/io/github/stream29/kodex/tool/contract/ToolContractTest.kt#L12
[tbuilder]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/builder/src/commonTest/kotlin/io/github/stream29/kodex/tool/builder/JsonToolBuilderTest.kt#L30
[tpatch]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/apply-patch/src/commonTest/kotlin/io/github/stream29/kodex/tool/applypatch/ApplyPatchToolsTest.kt#L13
[ttime]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/current-time/src/commonTest/kotlin/io/github/stream29/kodex/tool/currenttime/CurrentTimeToolsTest.kt#L17
[tcontext]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/get-context-remaining/src/commonTest/kotlin/io/github/stream29/kodex/tool/getcontextremaining/GetContextRemainingToolBehaviorTest.kt#L38
[imagehistorytest]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/image-generation/src/commonTest/kotlin/io/github/stream29/kodex/tool/imagegeneration/ImageGenerationToolClientTest.kt#L45
[timage]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/image-generation/src/commonTest/kotlin/io/github/stream29/kodex/tool/imagegeneration/ImageGenerationToolsTest.kt#L25
[tartifacts]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/image-generation/src/commonTest/kotlin/io/github/stream29/kodex/tool/imagegeneration/GeneratedImageArtifactsTest.kt#L19
[tmulti]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/multi-agent/src/commonTest/kotlin/io/github/stream29/kodex/tool/multiagent/SuggestSubagentTaskToolsTest.kt#L12
[tplan]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/plan/src/commonTest/kotlin/io/github/stream29/kodex/tool/plan/UpdatePlanToolTest.kt#L35
[inputtest]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/request-user-input/src/commonTest/kotlin/io/github/stream29/kodex/tool/requestuserinput/RequestUserInputToolsTest.kt#L44
[tsearch]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/tool-search/src/commonTest/kotlin/io/github/stream29/kodex/tool/toolsearch/ToolSearchToolsTest.kt#L26
[texecdisabled]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/unified-exec/src/commonTest/kotlin/io/github/stream29/kodex/tool/unifiedexec/UnifiedExecToolsTest.kt#L53
[texecsession]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/unified-exec/src/commonTest/kotlin/io/github/stream29/kodex/tool/unifiedexec/UnifiedExecToolsTest.kt#L576
[tview]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/view-image/src/commonTest/kotlin/io/github/stream29/kodex/tool/viewimage/ViewImageToolClientTest.kt#L56
[tviewschema]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/view-image/src/commonTest/kotlin/io/github/stream29/kodex/tool/viewimage/ViewImageToolsTest.kt#L29
[tweb]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/web-run/src/commonTest/kotlin/io/github/stream29/kodex/tool/webrun/WebRunToolsTest.kt#L25
[twebcontrols]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/web-run/src/commonTest/kotlin/io/github/stream29/kodex/tool/webrun/WebRunRequestControlsTest.kt#L12
[tmcpconfig]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/spec/contract/src/commonTest/kotlin/io/github/stream29/kodex/mcp/contract/McpServerConfigurationTest.kt#L10
[tmanager]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/impl/composition/src/commonTest/kotlin/io/github/stream29/kodex/mcp/impl/McpManagerImplTest.kt#L130
[tmcpschema]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/impl/composition/src/commonTest/kotlin/io/github/stream29/kodex/mcp/impl/McpSchemaProjectionTest.kt#L24
[transporttest]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/impl/composition/src/commonTest/kotlin/io/github/stream29/kodex/mcp/impl/McpTransportCompositionTest.kt#L33
[transporttesthttp]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/impl/composition/src/commonTest/kotlin/io/github/stream29/kodex/mcp/impl/McpTransportCompositionTest.kt#L87
[stdiotest]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/spec/stdio/src/commonTest/kotlin/io/github/stream29/kodex/mcp/stdio/McpStdioContractTest.kt#L30
[tstdiojvm]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/spec/stdio/src/jvmTest/kotlin/io/github/stream29/kodex/mcp/stdio/McpStdioTransportIoTest.kt#L32
[tstdiojs]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/spec/stdio/src/jsTest/kotlin/io/github/stream29/kodex/mcp/stdio/McpStdioTransportIoTest.js.kt#L26
[httptest]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/spec/streamable-http/src/commonTest/kotlin/io/github/stream29/kodex/mcp/streamablehttp/McpStreamableHttpTest.kt#L22
[tservice]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/impl/composition/src/jvmTest/kotlin/io/github/stream29/kodex/mcp/impl/McpServiceImplIoTest.kt#L64
[oldversionfixture]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/impl/composition/src/jvmTest/kotlin/io/github/stream29/kodex/mcp/impl/McpServiceImplIoTest.kt#L571
[toauth]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/impl/composition/src/jvmTest/kotlin/io/github/stream29/kodex/mcp/impl/McpOAuthIoTest.kt#L48
[execbuild]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/spec/unified-exec/build.gradle.kts#L9
[stdiobuild]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/spec/stdio/build.gradle.kts#L8
[httpbuild]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/spec/streamable-http/build.gradle.kts#L8
[mcpbuild]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/impl/composition/build.gradle.kts#L1

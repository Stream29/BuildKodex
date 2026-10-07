REVIEW READY — independent read-only audit of `6b7129fa0e719dde8f88a0034b9d572679984a03`; 33/33 projects statically covered; qualified verdict, NOT backend acceptance.

# Task Tree

- `Verify the pinned source baseline`()
- `Trace State, Context, Session, Storage, RPC and notification ownership`()
- `Review persistence, backend auth/naming and real fixtures`()
- `Publish the independently evidenced report`()

# Details

- Parent: [final reacceptance](2026-10-07-final-reaccept-spec-impl-repository.md).
- Source: inner `Kodex/refactor/spec`, exact HEAD
  `6b7129fa0e719dde8f88a0034b9d572679984a03`; clean inner worktree verified.
- Only this outer child report changed. No source/docs outside this child,
  build, IDE, live fixture, network, resource acquisition, commit, push or switch.
- Skills loaded: buildkodex-change, checklist-workflow, kanban-workflow,
  programmatic-planning, shared-context-workflow; workspace/document/IDE-collaboration/
  ask-user guidance. No IDE operation or user-resource interaction followed.
- Read canonical module-boundary, State abstraction/mutation/runtime, clean-model,
  Codex storage, coroutine resource, Hook, Home, model-budget/token-count,
  CLI Session, RPC architecture/state/settings/timeline/runtime/session/authentication/
  account-usage/MCP checklists; relevant planning, done histories and shared context.
- Classification follows parent: **R** critical static trace reviewed, not execution
  certification; **D** documented exception/debt; **B1** source-established behavior
  error; **B2** concrete suspicion requiring stated additional evidence; **U**
  unresolved project coverage.
- Result: physical hard-cutover and real backend ownership are substantially present.
  Three B1 findings below prevent an unqualified behavior verdict. B2 experiments
  remain gated. The initial checkpoint was published in this child before completion.

## Project coverage

- Actual `build.gradle.kts` inventory: State **4**, Context **11**, Storage **6**,
  Session **4**, RPC **6**, Hook **2** = **33**, including two genuine test fixtures.
- Paths below are relative to inner repository. For compact exact identities,
  `K.` expands to **`io.github.stream29.kodex.`**; extension identities include the
  receiver, not an invented class/interface. Linked source references give URI/line.
- History column is **tracked declaration introduction → current project move**,
  obtained with `git log --follow --diff-filter=A` and rename/source comparisons.
  It is not a claim that every current method existed at declaration introduction.
- Tests column names actual current suites. Pure specs use consuming implementation
  tests where they have no own suite; absence is stated rather than fabricated.

| # | Original project → actual project | Grade | Actual spec / implementation and production caller | Real tests / coverage limitation | Introduction → move |
| --- | --- | --- | --- | --- | --- |
| 1 | `agent-state/contract` → `agent-state/spec/contract` | R | [K.agentstate.contract.KodexAgentState][sc], `KodexAgentStateValue`, `RequestFinish`; sole [State factory/implementation][si] consumed by [file Session][sf]. Exception retains `K.agentstate.impl.KodexAgentStateInvalidTransitionException` identity but lives in spec, not a second implementation. | `SessionSettingsCasTest`, [AgentStateHardCutoverTest][st]; rich transition/cancel KDoc. Known compaction loss explicitly documented, not resolved. | `9c506941` → `19bad603` |
| 2 | `agent-state/context-window` → `agent-state/spec/context-window` | R | [K.agentstate.contextwindow.KodexAgentState.contextWindowTokenStatus / tokensUntilCompaction][sb]; pure extension derives from actual storage/model catalog. [Runtime auto-compaction][autobudget] and [GetContextRemainingTool][toolbudget] both consume it. No empty “budget service” needed. | `ContextWindowTokenBudgetTest`; model fallback/unknown/synthetic-zero paths; no live catalog certification. | `c099ef12` → `19bad603` |
| 3 | `agent-state/impl` + `agent-state/tool` → `agent-state/impl/state` | B1 | [K.agentstate.impl.CoroutineScope.KodexAgentState → private KodexAgentStateImpl][si] implements row 1. [K.agentstate.tool.McpService.visibleToolSpecs][tools] and `K.agentstate.tool.ResponseItem.ToolCall.toPendingToolEvent` retained original packages in same project. File/InMemory Session runtime owns returned State; [runtime RPC][rr] reaches it. | State implementation, context/request/hosted-search, tool projection, CAS suites; [known-loss test][st] proves B1-1. Tool-support merger is real, not forwarding scaffolding. | State `9c506941` → `19bad603`; tool support merged in that move |
| 4 | `agent-state/test` → unchanged `agent-state/test` | D | [K.agentstate.test.TestMcpService / TestAgentContextSettings][sfixture]; actual mock clients/context values, no production main consumer. Fixture exception, not a missing spec/impl pair. | Consumed by State/Session/runtime test source sets; no independent filesystem/provider coverage. | `6a35030e` → unchanged |
| 5 | `agent-context/contract` → `agent-context/spec/contract` | R | [K.agentcontext.contract.AgentContextSettings / AgentContextSourceSettings / AgentContextSourcePlan][cc]; actual private [K.rpc.server.ContextSnapshot at 143–150][bg] implements read-only context/Shell settings. [prefix resolver][cp] consumes one snapshot. | `AgentContextPrefixResolverTest`, settings model/default tests; no separate contract suite. | `1f585015` → `a6959173` |
| 6 | `agent-context/prompt-dsl` → `agent-context/spec/prompt-dsl` | R | [K.agentcontext.promptdsl.PromptXmlBuilder / promptXml][xml]; pure escaping DSL used by [prefix renderer][cr] and skill renderer. Public helpers are the actual shared semantics. | `PromptXmlTest`; escaping/indentation, not provider behavior. | `5594e1aa` → `a6959173` |
| 7 | `agent-context/prefix/agents-md/contract` → `agent-context/spec/agents-md` | R | [K.agentcontext.prefix.agentsmd.contract.AgentsMdInstructions / AgentsMdSnapshot / AgentsMdWarning][amd]; filesystem loader → prefix resolver → State request. Pure input/result models do not need a fake implementation. | [FileSystemAgentsMdTest][amdt]; consuming loader coverage; no own spec suite. | `1f585015` → `a6959173` |
| 8 | `agent-context/prefix/agents-md/filesystem` → `agent-context/impl/agents-md-filesystem` | R | [K.agentcontext.prefix.agentsmd.filesystem.loadAgentsMd][amdf]; genuine filesystem work, scoped reads, capped project input, cancellation preserved; [resolver][cp] calls configured-plan overload. | [FileSystemAgentsMdTest][amdt]: discovery/budget/read warnings; not all platform permission faults executed here. | `1f585015` → `a6959173` |
| 9 | `agent-context/prefix/skill/contract` → `agent-context/spec/available-skill` | R | [K.agentcontext.prefix.skill.contract.AvailableSkill / SkillSource / SkillScope / SkillWarning][avail]; actual discovery → prefix render → State. Metadata only, not a duplicate executable skill authority. | `FileSystemSkillsResolverTest`, `AvailableSkillRendererTest`; no own spec suite. | `1f585015` → `a6959173` |
| 10 | `agent-context/prefix/contract` → `agent-context/spec/prefix` | R | [K.agentcontext.prefix.contract.AgentContextPrefix / AgentSessionMeta][prefix]; resolver constructs real value; renderer consumes it on State request. Stable metadata identity is not a parallel runtime. | `AgentContextPrefixResolverTest`, `AgentContextPrefixRendererTest`; no own spec suite. | `6a35030e` → `a6959173` |
| 11 | `agent-context/prefix/impl` → `agent-context/impl/prefix` | R | [K.agentcontext.prefix.impl.AgentContextPrefixResolver][cp]; actual configured source plan, discovery and deduplication. State owns this helper at [State line 138][si]; no invented coordinator interface. | [AgentContextPrefixResolverTest][cpt]; configurable sources, paths/project scope. | `1f585015` → `a6959173` |
| 12 | `agent-context/prefix/render` → `agent-context/impl/prefix-render` | R | [K.agentcontext.prefix.render.AgentContextPrefix.render][cr], `List<AvailableSkill>.render`, `renderPlanningInstructions`; actual request history assembly at [State 190–193][si]. Rendering policy stays implementation; shared DSL remains spec. | [AgentContextPrefixRendererTest][crt], available-skill/planning renderer tests; provider acceptance not tested. | renderer `5594e1aa` → `a6959173` |
| 13 | `agent-context/skill/contract` → `agent-context/spec/skill` | D | [K.agentcontext.skill.contract.SkillsResolver / ResolvedSkills][skills], `SkillDocument`, `SkillResourceResult`; real [filesystem implementation][skillsfs]. Production uses discovery; body/resource operations have no production caller found. | `FileSystemSkillsResolverTest`; throws/read-authority/lifetime contract debt and dormant body API, see D-2. | `1f585015` → `a6959173` |
| 14 | `agent-context/skill/filesystem` → `agent-context/impl/skill-filesystem` | D | [K.agentcontext.skill.filesystem.FileSystemSkillsResolver : SkillsResolver][skillsfs]; private `FileSystemResolvedSkills : ResolvedSkills` at 256. Resolver calls plan overload; real refresh/cache/traversal behavior, but body/resource half dormant. | [FileSystemSkillsResolverTest][skillst]; refresh/add/remove/path rejection; legacy overload fixtures do not establish all configured-plan permutations. | `1f585015` → `a6959173` |
| 15 | `agent-context/skill/render` → `agent-context/impl/skill-render` | D | [K.agentcontext.skill.render.SkillDocument.render][skillrender]; substantive renderer with **no production caller or incoming production project dependency** found. Do not claim CLI skill-body injection is connected. | `SkillDocumentRendererTest` only; meaningful retirement/connection decision remains, D-2. | `1f585015` → `a6959173` |
| 16 | `agent-storage/clean-models` → `agent-storage/spec/clean-models` | R | [K.agentstorage.cleanmodels.stable.CleanIndexEntry][clean], `StableCleanEvent`, `StableWorkEvent`, and independent `unstable.UnstableCleanEvent`/pending hierarchy. [filesystem codecs][fs] and State complete/project paths use these exact hierarchies. | [StableTimelineGoldenTest][gold], stable/specialized/patch/pending hierarchy serialization and projection suites; golden is synthetic, not user-data capture. | `CleanIndexEntry` `24036a7e` → `d05e9cdb` |
| 17 | `agent-storage/contract` → `agent-storage/spec/contract` | D | [K.agentstorage.contract.KodexAgentStorage / MutableKodexAgentStorage][store], [IndexVersioned / MutableIndexVersioned][index], observable/cached interfaces, TokenCountSnapshot. Both storage impls and Session caches implement these exact contracts. | Filesystem/InMemory/cache suites exercise sparse semantics. Operation-level throws/invariants remain underdocumented, D-1; no fake contract suite claimed. | `d896b0d2` → `d05e9cdb` |
| 18 | `agent-storage/contract-ext` → `agent-storage/spec/contract-ext` | R | [K.agentstorage.contract.ext.MutableKodexAgentStorage.appendCompaction][compact], `initialize`, `KodexAgentStorage.activeMessageWindowAt`, `buildCompactionPrefix`; actual State and Session initialization consume these shared algorithms. Caller-supplied captured settings participate in B1-1; no duplicate owner. | [AgentStorageContractExtTest][extt], State context/compaction tests; retention tradeoff is whole-item approximate budget, not hidden truncation. | `CompactionStorage` `ca0616fc` → `d05e9cdb` |
| 19 | `agent-storage/filesystem` → `agent-storage/impl/filesystem` | B2 | [K.agentstorage.filesystem.FileSystemAgentStorage : MutableKodexAgentStorage][fs], [FileSystemIndexVersioned : MutableIndexVersioned][fi]; six real codecs/dirs → file Session-owned cache → State/RPC. Actual `StorageUri` platform implementations retained. | Index/storage/URI/legacy-settings suites, [FileSystemWriteCleanupTest][writet]. Prior primary-failure fix is real; B2-1/2/3 remain separate. | `bb82aa3a` → `d05e9cdb`; cleanup `88d26812` |
| 20 | `agent-storage/filesystem-layout` → `agent-storage/impl/filesystem-layout` | D | [K.agentstorage.filesystemlayout.requireStorageLayout / storedRecordIndexes / readRecord / writeRecord][layout]; concrete raw filesystem support used by frozen Home migrations, no current business model/codec. Explicit checklist exception; no dummy storage interface appropriate. | `FileSystemLayoutTest` real raw I/O/canonical numeric parsing; historical migration suites owned by UI/Home reviewer. | `81d353af` → `d05e9cdb` |
| 21 | `agent-storage/in-memory` → `agent-storage/impl/in-memory` | R | [K.agentstorage.inmemory.InMemoryKodexAgentStorage : MutableKodexAgentStorage][ims]; private `InMemoryIndexVersioned : MutableIndexVersioned`; real transient [InMemory Session][imse] initializes/owns it. Not default CLI disk storage and not a duplicate DTO authority. | `InMemoryKodexAgentStorageTest`, State tests; does not establish filesystem fault semantics. | `ba9caa3d` → `d05e9cdb` |
| 22 | `agent-session/contract` → `agent-session/spec/contract` | D | [K.agentsession.contract.KodexAgentSession / KodexSessionRepository / KodexRootSessionRepository][session], [KodexAgentDependencies][deps]. Both repositories expose one full State-bearing runtime; backend host consumes root repository. | File/InMemory Session, runtime-running-turn suites. Seven-vs-six KDoc and throws/borrow wording debt, D-1. | `e8623918` → `b4cb23e2` |
| 23 | `agent-session/filesystem` → `agent-session/impl/filesystem` | B1 | [K.agentsession.filesystem.FileSystemKodexSessionRepository : KodexRootSessionRepository][repo]; [CoroutineScope.FileSystemKodexAgentSession][sf] owns lease/cache/State/runtime. [CachedIndexVersionedImpl][cache] implements observable cached contract. [backend assembly][services] owns one repository. | Repository/catalog/cache suites, [FileSystemSessionCleanupTest][sessiont]; B1-2 eager root creation, B2-3 cache publication. Cleanup repair is not absence of every possible acquisition fault. | `e8623918` → `b4cb23e2`; cleanup `88d26812` |
| 24 | `agent-session/in-memory` → `agent-session/impl/in-memory` | R | [K.agentsession.inmemory.InMemoryKodexSessionRepository : KodexRootSessionRepository][imse]; same active root identity, real State/runtime and owner-scoped `SessionAgentStorage`, no second Session-state model. | `InMemoryKodexSessionRepositoryTest`, [AgentRuntimeRunningTurnTest][turnt]; no disk/lease certification inferred. | `e8623918` → `b4cb23e2` |
| 25 | `agent-session/test` → unchanged `agent-session/test` | D | [K.agentsession.test.testKodexAgentDependencies][sefixture]; real borrowed mock client/catalog/context/MCP wiring, no production main consumer. Old `NoOpHookManager` is gone, not another active Hook authority. | Used by actual repository/runtime suites; fixture-only exception, not a missing implementation. | `87ca62ef` → unchanged |
| 26 | `rpc/models` → `rpc/spec/models` | R | [K.rpc.models.BackendSettings][models], `CliFrontendSettings`, `AgentStateValue`, `Notification`, OAuth target/authorization/shell/suggested-session values. Real [global RPC][globalrpc], [runtime RPC][rr] and frontend proxies serialize/project them; reuse component specs, not old shared impl. | Settings/agent-state/notification/OAuth/shell/suggested-session model suites. NotificationHook moved to row 32, not duplicated. | `6b5e7b36` → `a6959173`; shared hard-cutover `206266c8` |
| 27 | `rpc/contract` → `rpc/spec/contract` | D | [K.rpc.contract.GlobalRpc][global], `AgentRuntimeRpc`, six concrete `@Rpc` timeline contracts; generic TimelineRpc is supporting API. [K.cli.sessiontitle.SessionTitleGenerator][titleport] moved from `app/shared/session-title`, real generator row 29. All eight services actually registered. | RPC boundary/value suites; frontend implementations come from kRPC generation. Stale “no implementation” KDoc, D-1; title port does have cancel/exception KDoc. | RPC `6b5e7b36` → `a6959173`; original title declaration `eb3aace0` → handoff `206266c8` |
| 28 | `rpc/client` → `rpc/impl/client` | R | [K.rpc.client.RestoringRpcClient][restore] implements upstream RpcClient; [CoroutineScope.rpcCachedIndexVersioned][rpcache] implements storage cached contract; `rpcStateIn`/SuspendMutableStateFlow helpers implement existing boundaries. [Application][application] wraps raw client before [RpcServices][proxies]; [RpcSessionBinding][binding] consumes all six caches. | Restoring/delegation, RpcStateIn, mutable flow, cached timeline/token-count suites; real generated RPC plus separate probe tests, no live transport certification. | `4897a08d` → `a6959173` |
| 29 | `rpc/server` → `rpc/impl/server` | B1 | [K.rpc.server.withBackendServices / BackendServices][services] → [BackendSessionHost][host], `BackendGlobalRpc : GlobalRpc`, `BackendAgentRuntimeRpc : AgentRuntimeRpc`, [six Backend*TimelineRpc][timelines]. Actual `K.cli.auth.BackendFileSystemAuthStore : OpenAiAuthStore`, OAuth and `OpenAiSessionTitleGenerator : SessionTitleGenerator` moved into real owner. | Services/host/runtime/global/account/OAuth/auth/title/management/notification suites, [BackendTitleRpcTest][titlet]. B1-3 auth cleanup masking; production exposure of B1-1/2. Integration is not fake delegation. | assembly `8397a155` → `a6959173`; auth/title `206266c8` |
| 30 | `rpc/in-memory` → `rpc/impl/in-memory` | R | [K.rpc.inmemory.withInMemoryRpc][transport], `ChannelRpcTransport : kotlinx.rpc.krpc.KrpcTransport`; scoped JSON RpcClient/RpcServer and channels, actual [Application][application] transport. Upstream interfaces are the genuine contracts; no invented local pair. | `InMemoryRpcTest`, `ChannelRpcTransportTest`, contract/client/server end-to-end fixtures; no socket/network transport claimed. | `f5a78934` → `a6959173` |
| 31 | `rpc/krpc-utils-patch` → `rpc/impl/krpc-utils-patch` | D | [kotlinx.rpc.internal.utils.map.SynchronizedHashMap : RpcInternalConcurrentHashMap][patch]; Native actual uses snapshot map, JVM uses concurrent map. [root substitution][rootbuild] replaces upstream utils in real kRPC transitive graph. Pinned API/KLIB exception, not domain spec. | `SynchronizedHashMapTest` checks detached collections. No new Native build/cancellation execution; removing replacement requires upstream-version/Native validation. | `ffc45f1e` → `a6959173` |
| 32 | `rpc/models/NotificationHook` → `hook/spec/notification` | R | [K.rpc.models.NotificationHook / NotificationHookType][hookmodel]; retained FQCN/serialized identity, frontend settings consume same model and row 33 executes it. No old control-Hook spec remains. | `NotificationHookSpecTest`, model compatibility tests; validation/serial defaults checked in source, not disk re-save. | `6b5e7b36` → `206266c8` |
| 33 | `app/shared/notification` → `hook/impl/notification` | R | [K.cli.notification.collectNotificationHooks][hooks]; sole [Application consumer 145–149][application], frontend-store config, startup cwd and frontend shell policy. Shell commands consume notification JSON; cannot veto/control State. | [NotificationHooksTest][hookt], [NotificationHooksRpcTest][hookrpct]: real JSON RPC and isolated local commands in historical suite, not executed here. | `8397a155` → `206266c8` |

### Physical/source-set and historical closure

- Read each of the 33 current build files, not just `settings.gradle.kts` names.
  Main dependencies now refer to real spec/component specs. State/Session fixture
  dependencies are test support, not production owners.
- Host convention retains configured JVM/Native targets plus JS node library/test;
  CLI convention retains configured host targets without that JS addition.
  Filesystem `StorageUri` still has JVM/JS/mingw/posix actuals; posix group includes
  Linux x64/arm64 and macOS arm64. No target removal was inferred from a commonMain move.
- Read-only rename/blob comparison of Kotlin declarations/tests:
  `a6959173` mapped 137 old paths (none survive at old paths), six current blobs
  changed after later handoffs, 26 mapped old-control-Hook paths subsequently retired;
  `d05e9cdb` mapped 73, one current changed filesystem-index blob;
  `b4cb23e2` mapped ten, three changed cleanup/runtime-fixture blobs;
  `19bad603` mapped 17, five changed contract/ownership-doc blobs.
  These are migration-comparison counts, not an assertion that every file is identical.
- Latest State change really merges former tool helpers into `impl/state`;
  budget extensions remain in `spec/context-window`. The exception relocation
  intentionally preserves original FQCN. No copied parallel State declaration found.
- `206266c8` really removes control Hook contract/implementation/tool-utils and Runtime
  turn-hook sources. Frozen historical parsers and strings such as “Stopped by hook”
  are old-data compatibility, not executable turn-control authority.
- `filesystem-layout` and kRPC patch are meaningful support exceptions.
  Contract-ext and prompt DSL are actual shared semantics; adding dummy interfaces
  would obscure, not improve, their boundary.
- Direct spec `project(...)` edges do not prove an implementation-free *resolved*
  graph: [root substitution][rootbuild] replaces external kRPC utils globally,
  including Native transitive use. This is explicit infrastructure support debt,
  not an invented clean direct-edge claim.

## Production ownership and operation traces

### State, Context and durable request behavior

- [State factory][si] 76–119 creates a child scope, loads existing durable state,
  returns the actual spec and cleans up factory failure. Private implementation
  at 129 is the sole executable State. Injected OpenAI/catalog/MCP resources are
  borrowed; the Session/backend closes them at its own level.
- Spec at [139][sc] documents coroutine lifetime, mutation admission,
  invalid transitions, cancellation and request/compaction failure semantics.
  State mutation mutex at [139][si] is a real algorithm boundary, not an extra DTO.
  `modify` 147–155 and mutation recovery 691–716 update durable state/projections.
- Request 157–285 snapshots context/settings, reads durable active message window,
  resolves configured input, selects actual visible tools, and writes terminal items
  into storage before publishing State. Response headers patch the **latest**
  settings with turn state at 545–559. This is different from the stale compaction
  snapshot bug; the two paths must not be conflated.
- [KodexRequestProjection][requestprojection] keeps provider mapping and stable thread
  cache identity separate from persistence DTOs. Compaction transport has null prompt
  cache key; ordinary request uses original stable identity. No legacy execution/
  frozen-prefix/provider/AgentMode authority was restored.
- [Visible tool support][tools] derives the effective catalog from actual MCP clients
  and fixed tool definitions, preserves conditional AskUser exposure and always Plan.
  Pending tool projection retains typed clean pending events, including invalid-call
  fallback, not a second wire hierarchy.
- Complete 395–429 writes stable results, removes completed unstable entries and
  timestamps them. This is sequential six-timeline persistence, **not** proof of a
  cross-timeline transaction. Full streaming output replay at 593–605 is intentionally
  retained; no bounded-memory certification follows from this review.
- Settings full CAS 436–443 compares actual latest settings under State mutex;
  same-value CAS returns true without append. Settings append 447–454 writes settings
  then timestamp then publishes latest index. Title and frontend settings use this
  actual State operation. Compaction B1-1 still bypasses latest-value preservation.
- [Context source settings][cc] retain default-enabled sources and empty custom list;
  custom path must be nonblank. [Resolver][cp] captures a snapshot at 32, constructs
  configured global/project plan and canonical-path deduplication at 47–78, and reads
  both instruction and skill discovery without creating input directories.
- [AGENTS loader][amdf] uses global/project budget with scoped source reads;
  project 32 KiB bound, truncation/UTF-8/read warnings remain actual result data.
  Cancellation is rethrown through `ensureActive`, not converted into a read warning.
- [Skill filesystem][skillsfs] supports real configured-plan discovery at 64,
  last-name precedence, rescanning and bounded metadata caching. Resolved resource
  access at 267 rejects absolute/traversal/resolved escapes. Production only uses
  discovered metadata; do not extrapolate dormant `loadSkill/readResource` into
  an implemented UI workflow.
- [Budget extension][sb] reads actual storage tail/settings/token snapshot and shared
  model budget. Runtime compaction and the context remaining tool consume the same
  arithmetic; successful compaction synthetic zero and missing ordinary usage keep
  their different semantics.

### Storage, models and Session/runtime ownership

- [FileSystemAgentStorage][fs] directly implements spec, with codecs at 26–64 for
  `index → CleanIndexEntry`, `work → StableWorkEvent`, settings, timestamp,
  structured token snapshot and list of unstable events. Constants 85–99 use exactly
  `index/work/settings/timestamp/token-count/unstable`. Empty creation 109–126
  initializes six pointers to `-1`; raw fork 132–146 copies only these timelines.
- Stable index/work and independent unstable/pending hierarchies remain in one
  clean-model spec; stable projected tool events are not a duplicate persistence
  hierarchy. [Golden tests][gold] check JSON serial names/shapes against synthetic
  pre-migration fixtures. They do not prove decoding all user histories or every
  event variant.
- Settings compatibility fixture reads old unknown `tools` and preserves current
  defaults (`cwd "."`, absent work/unstable indexes `-1`) without read-time rewrite.
  Token snapshots keep optional usage/diagnostics as null and serial kinds
  legacy/init/compact/response. Integer conversion belongs to frozen Home migration,
  not an alternate ordinary-runtime decoder.
- [Storage contract][store] exposes six timelines and derives latest across them.
  [Projection/retention][compact] are shared operations; active window uses durable
  compaction lineage and bounded retained stable events, not unstable output.
  Retention stops at a whole-item approximate budget; no covert truncation/second model.
- [File Session factory][sf] 27–66 owns child scope, renewable entry lease, cached
  storage, State and composed full runtime. Parent completion closes lease; factory
  failure cancels/joins child and closes lease. This proves ownership, not infallible
  cleanup under every platform exception.
- Repository [open 199–219][repo] returns the same live root object for repeated open.
  InMemory counterpart uses the same identity and State-bearing composition.
  [runtime composition][composition] 41/78–93 layers compact/steer/tool over the
  **same** State and pending-steer flow; runtime delegates State at 142 and owns
  `runningTurnSlot` at 143. The slot remains owned through cancellation cleanup,
  released in `finally` 182. No generic coordinator or second running flag is needed.
- [Session cache][cache] has real owner-lifetime guards, metadata/nonce and value
  cache; appending does not invalidate history nonce, destructive revert does.
  Observable storage is the same Session-owned cache consumed by State and RPC.
  This is legitimate owner adaptation, not copied domain authority.
- Catalog [391–411][repo] uses valid pointers without leasing/scanning; invalid
  pointers try a temporary entry lease. Busy writer permits scan-only read, no
  pointer repair; acquired lease permits repair. `useAndRelease` 433–440 explicitly
  joins the **lease Job**, not `NonCancellable`. Whole Session open reconciles
  timelines; this does not recover interrupted staging directories (B2-1).
- [BackendServices][services] borrows shared client/catalog/context/MCP into the
  dependency bundle, rather than instantiating per-Session infrastructure.
  Ordinary Session close does not call bundle `close()` to destroy shared clients.
  Actual backend resource teardown closes consumers before client/login owners.

### RPC, settings/authentication/title and notification integration

- [Application][application] 71–85 builds one backend, one scoped in-memory
  connection, registers actual services and wraps client error restoration **before**
  generating the eight [RpcServices][proxies] proxies. Frontend creates its own
  settings store and receives RPC, not live repository/State/auth handles.
  Frontend/Home/UI ownership itself remains other reviewers' scope.
- [BackendSessionHost][host] owns accepted operations, active Session bindings and
  full-TTL renewals. `keepAlive` 94–108 opens actual repository Session; commands
  121–137 are launched in binding operations scope, so caller waiting cancellation
  does not cancel already accepted work. Session-specific getter is not implicit load.
- Host renewal 201–219 stays while exact operation remains active, including cleanup;
  monotonic expiry 220–236 uses TTL, not shell/pending heuristics. Deactivation
  244–256 cancels/joins accepted operations before closing Session. Binding flows
  signal SessionNotActive, not silently reassigned roots.
- [Runtime RPC][rr] uses exact captured Session for admitted commands, actual
  runtime running-turn exclusion, actual State modifications and actual shell/
  pending-steer identities. Revert 138–161 validates nonce inside State write,
  rejects running, invalidates title history and clears pending tools.
- [Timelines][timelines] bind six actual observable storage timelines. Settings CAS
  69–70 invokes runtime/State full CAS; it is not a read-and-write wrapper with a
  separate settings lock. Cached frontend [binding 97–102][binding] consumes all six.
- [Global state][bg] projects current backend settings for actual context/shell;
  backend settings full CAS 111–116 uses original settings-store/MCP-manager
  coordination. MCP configuration store updates the same store at 135.
  Models/auth/account/MCP are real shared sources, not fabricated “healthy” DTOs.
- [BackendGlobalRpc][globalrpc] delegates actual GlobalState/account/OAuth/Session
  management; consume-usage-reset passes exact credit id to actual account store.
  Settings/account-usage component owners and supplier algorithms are reviewed by
  their allocated lines; this audit verifies backend integration, not every supplier.
- [BackendFileSystemAuthStore][auth] owns two source states and serialized writes.
  Fixed Codex `auth.json` and Kodex `auth.yml` paths at 233–235 are distinct.
  Selected source changes projection; maintenance refreshes **both** sources,
  including unselected, on the backend rather than requiring Settings UI.
- Auth login generation 88–107 fences stale attempts; successful write at 104
  precedes published in-memory auth. Remove targets exact source and advances
  generation. Refresh 129–163 rereads source, refreshes when due, rechecks external
  latest tokens before commit and preserves Codex metadata. Failed save does not
  retain unsaved new credentials. Cross-process compare+write is **not** atomic CAS:
  canonical policy promises complete last atomic replacement, no extra source lock.
- [OAuth][oauth] owns attempt identities, source/target, verifier/state and one
  pending completion; bad callback parsing precedes consumption. Cancellation/
  removal operate exact attempt/target; callback URL is parsed as data, not fetched.
  Accepted completion uses backend ownership. MCP bridge calls real OAuth manager.
- [SessionTitleGenerator spec][titleport] is a genuine moved backend-local port;
  [OpenAiSessionTitleGenerator][titlegenerator] performs one tool-free structured
  model request. [AgentTitleGeneration][title] consumes first eligible nonblank
  text once, includes disabled/nondefault-title cases, and is Session-runtime owned.
- Title commit 139–156 rereads latest settings and full-CASes only threadName,
  preserving unrelated changes and manual rename; conflict loops reuse generated
  result, do not issue more model calls. History replacement fences obsolete
  non-cooperative results and can reopen eligibility. Closing Settings/source tab
  does not own/cancel title work. Runtime RPC append launches it only **after**
  durable user append; title failure does not undo the message.
- [BackendTitleRpcTest][titlet] actually uses file-backed backend services/State and
  JSON RPC, closes originating frontend views, checks another projection/catalog,
  delayed manual rename and non-cancellable old-history result. It is stronger than
  an isolated naming fake, weaker than a live provider/renderer acceptance.
- [NotificationHook][hookmodel] retains original serialization/package identity.
  [BackendNotifications][notifications] emits actual accepted stopped/pending events;
  replay-zero/drop-oldest delivery is not a durable notification queue.
- [Hook executor][hooks] snapshots configured matching hooks in order, transfers
  notification JSON through stdin, uses frontend shell/default startup cwd, and
  waits for process exit during non-cancellable teardown with suppression.
  [Application][application] 145–149 is the **single** collection point. No turn hook,
  control reply, or duplicate per-tab subscription was restored.
- [RPC cache][rpcache] is actual Get-before-Flow metadata initialization, sparse
  exact-value cache and nonce validation. Remote SessionNotActive invalidates binding;
  no polling, hidden activation/retry or connection-close ownership.
  [Restoring client][restore] restores RPC errors on unary/stream boundaries,
  never resubmits business commands. Streaming restoration is upstream catch,
  not interception of downstream consumer exceptions.
- [InMemory RPC][transport] has bounded channels, actual JSON client/server,
  unexpected-completion monitors and non-cancellable bounded teardown with original
  failure suppression. It does not establish network/platform transport guarantees.

## B1 findings — source-established blockers

### B1-1 — CONFIRMED UNFIXED baseline: compaction overwrites accepted settings

- Evidence: [State compact 287–333][si] captures settings at start; remote wait permits
  `compareAndSetSettings`. Commit 632–647 passes captured `previousSettings` into
  [appendCompaction 13–32][compact], whose `previousSettings.copy` becomes the next
  settings record. No merge against current accepted settings occurs.
- Production: `SettingsTimelineRpc.compareAndSet` → [BackendSettingsTimelineRpc
  67–70][timelines] → same Session runtime/State; force/auto-compaction reaches that
  State through [BackendAgentRuntimeRpc][rr]/runtime composition.
- [Current characterization 112–149][st] **accepts** title `"during"`, observes it at
  139, then asserts checkpoint title `"before"` at 143. Comment 110 explicitly calls
  this a known baseline defect. A green test proves loss, **not** correctness.
- Impact: accepted manual title/settings changes during provider compaction wait
  can disappear in next durable snapshot, including backend one-shot title output.
  Mutex protects each commit but not preservation across that wait.
- History: inherited State/compaction algorithm; `034b778c` added characterization;
  `19bad603` moved/clarified contracts, did not repair executable policy.
  Spec [228–230][sc] honestly describes limitation; KDoc cannot absolve lost acceptance.
- Minimum authorized next work: preserve latest unrelated settings at the actual
  State compaction commit, applying only compaction-owned fields. Turn existing
  characterization into regression expecting `"during"` and add unrelated settings
  cases. Do not introduce a generic transaction coordinator. **No fix in this audit.**

### B1-2 — backend startup eagerly creates/scans Session root

- Evidence: [Application 71–74][application] eagerly calls `withBackendServices`.
  [Services 69–70][services] calls `withBackendSessionHost` factory; [host 53–59][host]
  creates repository before providing services. [File repository 472–481][repo]
  immediately creates Home/`sessions/` and scans entries.
- Canonical [Home on-demand rule][homepolicy] says constructing Application for
  initial New Session must not create `sessions/`; creation/open/catalog-refresh are
  the first allowed Session uses. Registration/startup is none of these operations.
- Impact: initial local draft startup mutates Session layout and may fail on Session
  layout/permission errors before user opens/creates/refreshes a Session.
- History: repository factory behavior inherited (`e8623918`); actual eager
  application-wide backend assembly introduced `8397a155`, moved `a6959173`.
  Latest Home tests/source do not remove this backend call chain.
- Tests: current direct repository factory tests assume initializing root; backend
  services fixtures use isolated existing/prepared homes. No inspected test proves
  Application-with-initial-draft leaves `sessions/` absent.
- Minimum boundary change: retain **one** backend repository owner but defer its
  filesystem acquisition/layout scan until first real create/open/catalog access.
  Do not recreate old per-tab repositories. Coordinate once with Home/frontend
  audit; this is a shared root cause, not an additional count per consumer.

### B1-3 — auth temporary cleanup can replace primary error/cancellation

- Evidence: actual backend [auth write 205–231][auth] does write/move in `try`,
  then directly deletes temp at 229 in non-cancellable `finally`. If write/move
  throws and delete also throws, Kotlin `finally` propagates delete failure instead
  of the original I/O failure or cancellation. There is no primary capture/
  suppression in this function.
- Production: [BackendOAuth][oauth] completion → `commitLogin` → auth write;
  background refresh uses same function for both Codex and Kodex sources.
  Caller mapping cannot recover a primary throwable already replaced here.
- Impact: lost causal failure/cancellation semantics and unusable diagnostics;
  cleanup failure also leaves uncertainty about durable write outcome.
  This is conditional fault behavior established by source, **not** a claim that
  current normal login or every cancellation loses credentials.
- History: exact `finally` lines blame to `8397a155`; byte-identical backend auth
  moved in `206266c8`. Storage repair `88d26812` does not cover this separate owner.
- Tests: [BackendFileSystemAuthStoreTest][autht] verifies failed save, cancel cleanup,
  external-refresh winner and binding; its move fault fixture does not combine
  primary write/move/cancel with temp-delete failure.
- Minimum authorized repair/experiment: actual auth owner primary capture, attempt
  cleanup in NonCancellable, suppress secondary failure onto original, keep successful
  cleanup failure observable per policy. Inject both failures for both sources,
  assert identity/suppression and no unsaved published credentials. No generic
  filesystem interface or frontend auth-maintenance facade needed.

## B2 findings — additional gated evidence required

### B2-1 — interruption residue has no actual write/revert recovery on open

- [Timeline][fi] writes `.kodex-write-*` and stages revert suffixes in
  `.kodex-revert-pending-*`/committed directories. Revert catches ordinary failure
  and rolls back at 147; post-commit cleanup is best-effort. Process interruption
  is expressly not repaired on open in KDoc 18–28.
- Actual [Session cache open 109–110][cache] enumerates numbered entries and
  reconciles pointers. It never classifies/restores/cleans owned staging directories.
  Valid-pointer/catalog repair **does exist**; this is not “no disk repair at all.”
- Canonical [Home Session timeline rule][homepolicy] calls for owned interruption
  recovery/cleanup. Residue is inherited staged-write/revert behavior (`d9e27f77`
  and earlier), not a migration-copy regression.
- Missing evidence: deterministic staged crash snapshots through actual repository
  reopen, including pre/post revert commit boundaries and unknown-path preservation.
  Inspect authoritative record outcome before declaring durable history corruption.
- Separate same-policy check: [auth startup/read/write][auth] also writes uniquely
  named sibling temporaries but has no inspected startup/read scan of its own
  leftover namespace. Global-file interruption cleanup is required by Home policy.
  Include interrupted-auth-write fixtures for both source roots; settings-store
  namespace recovery belongs the Settings reviewer, not an assumed backend fix.
- Minimum work: isolated fixture snapshots for owner namespaces, real reopen, retry
  and record/pointer assertions. Decide recoverable vs fail-closed ambiguous states
  in timeline owner; do not build a global cleaner or delete generic `.tmp` files.

### B2-2 — runtime numeric parser differs from canonical migration layout parser

- [Runtime index parser 348–353][fi] accepts digit names like `01.json` as index 1;
  subsequent read/reconcile uses canonical `1.json`.
  [Layout parser 140–144][layout] accepts only canonical nonnegative decimal names.
- Normal writers produce canonical names, so no normal-writer failure is claimed.
  Existing externally introduced/noncanonical layout can be enumerated differently
  by runtime and frozen migration helpers; desired failure policy is not established
  by a serializer-green suite.
- History: runtime parser inherited (`bb82aa3a`), canonical layout declaration
  `81d353af`; relocations preserve both.
- Minimum evidence: real isolated reopen with `01.json`, both names, malformed and
  huge indexes; compare scan/get/pointer/catalog/migration admission. Agree whether
  reject/report or support, never silently merge/remove ambiguous history.

### B2-3 — successful disk publication + cleanup error can bypass live cache publication

- Precursor is **explicitly verified** by [FileSystemWriteCleanupTest 97–117][writet]:
  record/pointer remain durably published while successful operation throws cleanup
  failure. This approved observability policy is not itself labeled a bug.
- Cross-owner evidence: [FileSystemIndexVersioned.setUnsafe 234–251][fi] commits
  pointer/record then may throw final-delete error. [CachedIndexVersionedImpl.set
  256–280][cache] publishes cached indexes/values/latest **only after normal delegate
  return**. No catch/reconciliation or owner invalidation handles a committed-but-
  throwing write. State reads the same cached storage, not a fresh raw directory.
- Concrete suspected consequence: live cached tail stays old; State retry computes
  previous tail + 1, cached guard accepts it, `delegate.setUnsafe` bypasses append
  check and atomically replaces that same numbered record. Accepted earlier durable
  history can therefore be hidden/replaced in this conditional sequence.
- Production chain is real: [Services 69–70][services] → [file Session cache 45][sf]
  → [State settings append 447–454][si] → [Settings timeline RPC 69–70][timelines].
  No invented generic transaction manager is involved.
- History: cache delegate-first publication originates in `e8623918` (metadata
  additions later); write finalization inherited `d9e27f77`; `88d26812` preserves
  successful-cleanup-failure propagation deliberately. Do not blame the repair
  merely because it added a test exposing this precursor.
- Missing experiment: **real Session cache + State**, one-shot temp-delete fault
  after entry move, compare raw records/pointer, cached metadata and published State,
  then explicit subsequent write; assert prior record is not overwritten.
  Existing direct-storage suite does not cover this owner/cache integration.
- Minimal boundary decision: reconcile/invalidate the actual Session cache after
  uncertain/committed delegate outcome before allowing another append. Preserve
  failure visibility; do not fix by swallowing cleanup or automatically retrying RPC.

### B2-4 — settings-write success then timestamp failure can leave State tail stale

- [State appendSettings 447–454][si] writes actual settings, then timestamp, then
  State `latestIndex`. `updateSettings`/CAS at 431–443 have no recovery `finally`.
  This differs from ordinary `mutate` at 691–716, which resynchronizes projections
  in NonCancellable even when its body fails.
- Specific suspicion: settings/cache append succeeds but timestamp throws or is
  cancelled; durable settings/cache tail advances while State/RPC latest index does
  not. Subsequent observation/title lookup at State's old index may miss that record.
  This is a **failed/uncertain** operation, not another accepted-CAS compaction loss.
- History: shared appendSettings/CAS extraction `f622b9c0`, preserved by `19bad603`.
  Current CAS and cleanup suites do not establish this settings-then-timestamp fault
  through the real Session-owned cache and backend projections.
- Minimum experiment: inject timestamp-only failure after successful settings append;
  compare disk, timeline metadata, State latest/settings view and next explicit CAS.
  Define failed-write observation/recovery at State owner without promising a new
  six-timeline transaction or silently replaying the command.

## Prior cleanup findings — narrowly repaired, not erased or re-reported

- [Repository create 142–162 / fork 164–187][repo] now catches failure after returned
  reservation, deletes exact reserved target in NonCancellable, suppresses cleanup
  error and rethrows primary. Source/unknown/legacy data are not blanket-cleaned.
- [FileSystemSessionCleanupTest][sessiont] contains 66 declared fault cases:
  initialization/cancellation across reservation-following storage/timeline steps,
  disk re-scan, same-owner retry, fork source preservation and cleanup failure.
  At 93–113 a deliberate cleanup failure leaves target present and is suppressed.
  Therefore do **not** promise failed creation can never leave a directory.
- The repair covers faults after owned reservation return; it does not prove every
  acquisition/handoff boundary before that return, or every failed cleanup path.
  Such claims would require further injected cases, not “tests green” inference.
- [FileSystemWriteCleanupTest][writet] contains 20 declared cases: entry/pointer
  write/move I/O or cancellation plus failed delete, original identity and suppressed
  cleanup, nested pointer+entry cleanup, successful-publication cleanup failure.
  [Current source 243–253 / 323–341][fi] actually implements this suppression.
- `88d26812` is a real source-and-test repair of those exact prior findings.
  Historical done record reports 20/20 + 66/66 (storage/session totals 36/103).
  These are **historical reported executions, not runs performed by this audit**.
- Do not reopen the repaired primary-error findings unchanged. Auth B1-3 is another
  owner; B2-1 is process interruption; B2-3 is live cache integration after committed
  failure. Neither those concerns nor successful fixtures certify absence of all
  filesystem, lease, cancellation or cleanup failures.

## D findings, supporting exceptions and meaningful ablation

### D-1 — contract/document consistency

- [Session spec 13][session] still says **seven** timelines while current spec/code/
  paths have six. Repository open/delete and sparse index ops lack enough throws/
  invalid-boundary/lifetime documentation for real lease/layout/closed-owner errors.
  This is documentation debt, not a fabricated absent implementation.
- [KodexAgentDependencies][deps] describes owning dependencies and implements close,
  but Session assembly **borrows** application-wide dependencies. Current production
  teardown is correctly centralized; clarify bundle caller obligations to prevent a
  future Session caller closing shared catalog/client/MCP.
- [GlobalRpc KDoc 28][global] still says review contract has no client/server
  implementation, contradicted by eight registered/generated real services.
  Remove stale sentence, not actual implementation.
- Current State KDoc meaningfully covers transitions/lifetime/cancellation and even
  states the known compaction limitation. Do not downgrade it to an empty API; do not
  turn honest documentation into behavior acceptance.
- Shared-context abstraction note still mentions old markNewTurn/provider/frozen-
  prefix/AgentMode/control-Hook assumptions. Canonical current checklist/source take
  precedence. Reconcile the reusable note separately, do not restore obsolete APIs.

### D-2 — dormant skill-body and title-helper surfaces

- Repository-wide source/dependency search found no production use of
  `ResolvedSkills.loadSkill/readResource` or `SkillDocument.render`.
  The checklist's explicit selected-skill-body injection account is not substantiated
  by current CLI calls. Discovery metadata **is** connected and must be preserved.
- Spec skill operations only tersely describe methods; authority, changing files,
  returned failures vs thrown cancellation and lifetime obligations need documented
  intent. Real implementation traversal safeguards cannot substitute for contract.
- Meaningful ablation: either explicitly retain body/resource rendering as supported
  dormant API with intended consumer/tests, or remove unused renderer project/body
  surface after proving no external support commitment. Do not add an empty
  `SkillCoordinator` to create a cosmetic spec/impl pair.
- `AgentTitleGeneration.suppress/renameThread/updateSettings` have test/direct
  helper uses, not production runtime-RPC callers. Production manual rename is
  SettingsTimeline CAS. Candidate simplification must retain actual gate/history
  fencing/CAS commit, not replace backend naming with a frontend coordinator.
- State extensions `updateThreadName` and `appendPlanUpdate` document separate
  read/write operations. Do not use them as evidence of atomic full-settings CAS;
  current backend title does not call the former. Plan tool legitimately calls the
  latter; whole operation atomicity is not established by its convenience name.

### D-3 — legitimate non-paired modules, fixtures and kRPC replacement

- `contract-ext` pure storage algorithms, prompt DSL and clean/protocol models are
  meaningful shared semantics. Their lack of `implements` is expected.
- `filesystem-layout` is concrete raw historical-migration support by explicit Home
  checklist decision, not a domain storage spec or current codec layer.
- kRPC patch uses upstream internal interfaces/API names and exact Native KLIB
  unique name `org.jetbrains.kotlinx:utils`; [patch build][patchbuild] states pinned
  0.10.3 reason. Snapshot collection getters at [54–61][patch] fix actual mutation/
  cancellation traversal behavior. A forwarding empty pair would do nothing.
- Removing patch requires compatible upstream replacement and all relevant Native
  cancellation/binary-link checks, not just JVM unit green. No such execution here.
- Two fixtures have real test behavior, no ordinary production consumer. Do not add
  production “test contract” abstractions merely to balance directory counts.
- Keep necessary owner/cache/nonce bindings, restoring transport client and settings
  projection. Their ablation would lose lifetime checks, sparse/history invalidation,
  wire-error restoration or read-only live source projection; these are not generic
  empty wrappers.

## Test limitations, unreviewed paths and coordinator handoff

- **All 33 project rows assigned; no U project inventory remainder.**
  That is critical-path static coverage, not exhaustive execution of every branch.
- No new Gradle/build/IDE/fixture/provider/shell/RPC run. Source/test bodies and Git
  histories are evidence; historical green records are explicitly attributed.
- No new Native/JS/Windows execution, generated kRPC compilation or platform
  `StorageUri` permission/cancellation test. No resolved configuration invocation.
- External kRPC/Mosaic/LuceneKmp/KotlinMcpSdk internals were not exhaustively reviewed.
  Actual dependency/substitution/consumer boundaries were inspected.
- Not exhaustive live provider refresh/usage-reset/MCP OAuth; not all malformed
  clean-event variants or actual user-disk histories; not every acquisition-before-
  return failure. B2 experiments above specify what missing evidence matters.
- Frontend reviewer owns application/tab/popup/composer/History rendering and Home
  reviewer owns prepare/migration lease behavior. Relevant frontend calls were read
  to validate backend ownership only. Direct model/RPC tests are not renderer input
  acceptance; stopped Hook historical tests are not live process tests here.
- Session lease mechanism/platform cleanup algorithms belong other allocated lines.
  This audit inspected actual creation/close/join consumers, not every lease internal.
- Coordinator should deduplicate B1-1/B1-2 across Runtime/frontend/Home reports,
  verify B1-3 actual owner, and gate B2 fixtures centrally. No parallel reviewer should
  compete for build/fixture resources or implement any suggestion under this audit.
- Final verdict: **real physical/spec/owner closure verified within listed scope;
  behavior acceptance blocked by confirmed unrepaired compaction loss, eager Session
  startup mutation and auth primary-failure masking.** Separate debt/experiments
  remain explicit. There is no basis for “all six roots pass.”

## URI/line evidence index

- References below resolve against pinned source. A starting line identifies the
  cited declaration; ranges in text identify the particular operation.

[sc]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-state/spec/contract/src/commonMain/kotlin/io/github/stream29/kodex/agentstate/contract/KodexAgentState.kt#L124
[si]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-state/impl/state/src/commonMain/kotlin/io/github/stream29/kodex/agentstate/impl/KodexAgentStateImpl.kt#L76
[st]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-state/impl/state/src/commonTest/kotlin/io/github/stream29/kodex/agentstate/impl/AgentStateHardCutoverTest.kt#L112
[sb]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-state/spec/context-window/src/commonMain/kotlin/io/github/stream29/kodex/agentstate/contextwindow/ContextWindowTokenBudget.kt#L20
[tools]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-state/impl/state/src/commonMain/kotlin/io/github/stream29/kodex/agentstate/tool/KodexVisibleToolSpecs.kt#L43
[requestprojection]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-state/impl/state/src/commonMain/kotlin/io/github/stream29/kodex/agentstate/impl/KodexRequestProjection.kt#L12
[sfixture]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-state/test/src/commonMain/kotlin/io/github/stream29/kodex/agentstate/test/TestAgentStateDependencies.kt#L29
[cc]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-context/spec/contract/src/commonMain/kotlin/io/github/stream29/kodex/agentcontext/contract/AgentContextSettings.kt#L18
[xml]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-context/spec/prompt-dsl/src/commonMain/kotlin/io/github/stream29/kodex/agentcontext/promptdsl/PromptXml.kt#L15
[amd]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-context/spec/agents-md/src/commonMain/kotlin/io/github/stream29/kodex/agentcontext/prefix/agentsmd/contract/AgentsMdModels.kt#L11
[amdf]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-context/impl/agents-md-filesystem/src/commonMain/kotlin/io/github/stream29/kodex/agentcontext/prefix/agentsmd/filesystem/FileSystemAgentsMd.kt#L43
[amdt]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-context/impl/agents-md-filesystem/src/commonTest/kotlin/io/github/stream29/kodex/agentcontext/prefix/agentsmd/filesystem/FileSystemAgentsMdTest.kt#L13
[avail]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-context/spec/available-skill/src/commonMain/kotlin/io/github/stream29/kodex/agentcontext/prefix/skill/contract/AvailableSkillModels.kt#L37
[prefix]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-context/spec/prefix/src/commonMain/kotlin/io/github/stream29/kodex/agentcontext/prefix/contract/AgentContextPrefix.kt#L23
[cp]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-context/impl/prefix/src/commonMain/kotlin/io/github/stream29/kodex/agentcontext/prefix/impl/AgentContextPrefixResolver.kt#L23
[cpt]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-context/impl/prefix/src/commonTest/kotlin/io/github/stream29/kodex/agentcontext/prefix/impl/AgentContextPrefixResolverTest.kt#L19
[cr]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-context/impl/prefix-render/src/commonMain/kotlin/io/github/stream29/kodex/agentcontext/prefix/render/AgentContextPrefixRenderer.kt#L22
[crt]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-context/impl/prefix-render/src/commonTest/kotlin/io/github/stream29/kodex/agentcontext/prefix/render/AgentContextPrefixRendererTest.kt#L22
[skills]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-context/spec/skill/src/commonMain/kotlin/io/github/stream29/kodex/agentcontext/skill/contract/SkillsResolver.kt#L8
[skillsfs]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-context/impl/skill-filesystem/src/commonMain/kotlin/io/github/stream29/kodex/agentcontext/skill/filesystem/FileSystemSkillsResolver.kt#L33
[skillst]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-context/impl/skill-filesystem/src/commonTest/kotlin/io/github/stream29/kodex/agentcontext/skill/filesystem/FileSystemSkillsResolverTest.kt#L23
[skillrender]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-context/impl/skill-render/src/commonMain/kotlin/io/github/stream29/kodex/agentcontext/skill/render/SkillDocumentRenderer.kt#L7
[clean]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-storage/spec/clean-models/src/commonMain/kotlin/io/github/stream29/kodex/agentstorage/cleanmodels/stable/CleanIndexEntry.kt#L12
[gold]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-storage/spec/clean-models/src/commonTest/kotlin/io/github/stream29/kodex/agentstorage/cleanmodels/stable/StableTimelineGoldenTest.kt#L7
[store]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-storage/spec/contract/src/commonMain/kotlin/io/github/stream29/kodex/agentstorage/contract/KodexAgentStorage.kt#L16
[index]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-storage/spec/contract/src/commonMain/kotlin/io/github/stream29/kodex/agentstorage/contract/IndexVersioning.kt#L21
[compact]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-storage/spec/contract-ext/src/commonMain/kotlin/io/github/stream29/kodex/agentstorage/contract/ext/CompactionStorage.kt#L13
[extt]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-storage/spec/contract-ext/src/commonTest/kotlin/io/github/stream29/kodex/agentstorage/contract/ext/AgentStorageContractExtTest.kt#L88
[fs]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-storage/impl/filesystem/src/commonMain/kotlin/io/github/stream29/kodex/agentstorage/filesystem/FileSystemAgentStorage.kt#L19
[fi]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-storage/impl/filesystem/src/commonMain/kotlin/io/github/stream29/kodex/agentstorage/filesystem/FileSystemIndexVersioned.kt#L18
[writet]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-storage/impl/filesystem/src/commonTest/kotlin/io/github/stream29/kodex/agentstorage/filesystem/FileSystemWriteCleanupTest.kt#L24
[layout]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-storage/impl/filesystem-layout/src/commonMain/kotlin/io/github/stream29/kodex/agentstorage/filesystemlayout/FileSystemLayout.kt#L10
[ims]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-storage/impl/in-memory/src/commonMain/kotlin/io/github/stream29/kodex/agentstorage/inmemory/InMemoryKodexAgentStorage.kt#L26
[session]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-session/spec/contract/src/commonMain/kotlin/io/github/stream29/kodex/agentsession/contract/KodexSession.kt#L13
[deps]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-session/spec/contract/src/commonMain/kotlin/io/github/stream29/kodex/agentsession/contract/KodexAgentDependencies.kt#L17
[repo]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-session/impl/filesystem/src/commonMain/kotlin/io/github/stream29/kodex/agentsession/filesystem/FileSystemKodexSessionRepository.kt#L39
[sf]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-session/impl/filesystem/src/commonMain/kotlin/io/github/stream29/kodex/agentsession/filesystem/FileSystemKodexAgentSession.kt#L27
[cache]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-session/impl/filesystem/src/commonMain/kotlin/io/github/stream29/kodex/agentsession/filesystem/CachedAgentStorage.kt#L53
[sessiont]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-session/impl/filesystem/src/commonTest/kotlin/io/github/stream29/kodex/agentsession/filesystem/FileSystemSessionCleanupTest.kt#L33
[imse]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-session/impl/in-memory/src/commonMain/kotlin/io/github/stream29/kodex/agentsession/inmemory/InMemoryKodexSessionRepository.kt#L31
[turnt]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-session/impl/in-memory/src/commonTest/kotlin/io/github/stream29/kodex/agentsession/inmemory/AgentRuntimeRunningTurnTest.kt#L58
[sefixture]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-session/test/src/commonMain/kotlin/io/github/stream29/kodex/agentsession/test/TestKodexAgentDependencies.kt#L15
[composition]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-runtime/impl/composition/src/commonMain/kotlin/io/github/stream29/kodex/agentruntime/impl/KodexAgentRuntimeComposition.kt#L41
[models]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/spec/models/src/commonMain/kotlin/io/github/stream29/kodex/rpc/models/BackendSettings.kt#L30
[global]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/spec/contract/src/commonMain/kotlin/io/github/stream29/kodex/rpc/contract/GlobalRpc.kt#L21
[titleport]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/spec/contract/src/commonMain/kotlin/io/github/stream29/kodex/cli/sessiontitle/SessionTitleGenerator.kt#L14
[restore]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/client/src/commonMain/kotlin/io/github/stream29/kodex/rpc/client/RestoringRpcClient.kt#L21
[rpcache]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/client/src/commonMain/kotlin/io/github/stream29/kodex/rpc/client/RpcCachedIndexVersioned.kt#L40
[services]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/server/src/commonMain/kotlin/io/github/stream29/kodex/rpc/server/BackendServices.kt#L36
[host]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/server/src/commonMain/kotlin/io/github/stream29/kodex/rpc/server/BackendSessionHost.kt#L43
[rr]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/server/src/commonMain/kotlin/io/github/stream29/kodex/rpc/server/BackendAgentRuntimeRpc.kt#L38
[timelines]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/server/src/commonMain/kotlin/io/github/stream29/kodex/rpc/server/BackendTimelines.kt#L18
[bg]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/server/src/commonMain/kotlin/io/github/stream29/kodex/rpc/server/BackendGlobalState.kt#L36
[globalrpc]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/server/src/commonMain/kotlin/io/github/stream29/kodex/rpc/server/BackendGlobalRpc.kt#L18
[auth]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/server/src/commonMain/kotlin/io/github/stream29/kodex/cli/auth/BackendFileSystemAuthStore.kt#L44
[autht]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/server/src/commonTest/kotlin/io/github/stream29/kodex/cli/auth/BackendFileSystemAuthStoreTest.kt#L38
[oauth]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/server/src/commonMain/kotlin/io/github/stream29/kodex/rpc/server/BackendOAuth.kt#L56
[title]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/server/src/commonMain/kotlin/io/github/stream29/kodex/cli/sessiontitle/AgentTitleGeneration.kt#L23
[titlegenerator]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/server/src/commonMain/kotlin/io/github/stream29/kodex/cli/sessiontitle/OpenAiSessionTitleGenerator.kt#L20
[titlet]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/server/src/commonTest/kotlin/io/github/stream29/kodex/rpc/server/BackendTitleRpcTest.kt#L28
[notifications]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/server/src/commonMain/kotlin/io/github/stream29/kodex/rpc/server/BackendNotifications.kt#L15
[transport]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/in-memory/src/commonMain/kotlin/io/github/stream29/kodex/rpc/inmemory/WithInMemoryRpc.kt#L37
[patch]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/krpc-utils-patch/src/commonMain/kotlin/kotlinx/rpc/internal/utils/map/SynchronizedHashMap.kt#L13
[patchbuild]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/krpc-utils-patch/build.gradle.kts#L5
[rootbuild]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/build.gradle.kts#L3
[hookmodel]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/hook/spec/notification/src/commonMain/kotlin/io/github/stream29/kodex/rpc/models/NotificationHook.kt#L17
[hooks]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/hook/impl/notification/src/commonMain/kotlin/io/github/stream29/kodex/cli/notification/NotificationHooks.kt#L32
[hookt]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/hook/impl/notification/src/commonTest/kotlin/io/github/stream29/kodex/cli/notification/NotificationHooksTest.kt#L39
[hookrpct]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/hook/impl/notification/src/commonTest/kotlin/io/github/stream29/kodex/cli/notification/NotificationHooksRpcTest.kt#L36
[application]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/application/src/commonMain/kotlin/io/github/stream29/kodex/cli/app/Application.kt#L55
[proxies]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/rpc/src/commonMain/kotlin/io/github/stream29/kodex/cli/rpc/RpcServices.kt#L16
[binding]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/rpc/src/commonMain/kotlin/io/github/stream29/kodex/cli/rpc/RpcSessionBinding.kt#L97
[homepolicy]: file:///home/stream/ACodeSpace/push/BuildKodex/checklist/kodex-home.md#L34
[autobudget]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-runtime/impl/decorator/compact/src/commonMain/kotlin/io/github/stream29/kodex/agentruntime/decorator/compact/KodexAgentCompactionRuntimeImpl.kt#L119
[toolbudget]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/get-context-remaining/src/commonMain/kotlin/io/github/stream29/kodex/tool/getcontextremaining/GetContextRemainingTool.kt#L19

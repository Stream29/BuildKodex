# Task Tree

- `Trace moved backend contracts, implementations and production consumers`()
- `Review serialization, ownership, tests and ablation candidates`()
- `Publish evidence and unreviewed coverage`()

# Details

- Parent: [Spec/impl reacceptance](2026-10-05-reaccept-spec-impl-refactor.md).
- Own review scope: complete `agent-context`, `agent-storage`, `agent-session`, `hook`, `rpc`.
- Report only in this file; source, other tasks and shared resources are read-only.
- Apply parent acceptance rules and pinned baseline; no independent fixes or builds.
- Status: **REVIEW READY** — independent static audit complete; coordinator
  independently adjudicates blockers. No implementation/build authorization.

## Review report

## 1. Baseline and identity evidence

- Direct branch/HEAD/status checks: `refactor/spec`,
  `821b66275aae2f0a9541fa2e9e03023d875c30b5`, clean inner worktree.
- Loaded project/user AGENTS, change/checklist/kanban/planning/ask-user and
  document skills, parent/child tasks, canonical module-boundary checklist,
  Home/storage compatibility/lifecycle/AgentState/Session and relevant
  Hook/RPC checklists. No source, other report, fixture, IDE/device or build
  was modified.
- Actual `git show`/`diff-tree -r -M`:
  - `a6959173a724e50204d96e0d1ec1c5989a8f528e`: context/RPC/Hook,
    **137 Kotlin R100 renames**.
  - `d05e9cdb9bff5eb42609a5db505d57e66990a290`: storage,
    **73 Kotlin R100 renames**.
  - `b4cb23e2ce928673fdf0a2857bfc9304071e7eff`: Session,
    **10 Kotlin R100 renames**.
- Compared each original blob at the move's parent with its target at pinned
  HEAD; checked each old path with `git cat-file -e HEAD:<old-path>`.
  **All 220 old Kotlin paths are absent; every moved main source is still
  byte-identical to its original.** No replacement interface/DTO accompanied
  by a surviving old production declaration was found.
- Only two relocated Kotlin test files subsequently changed:
  `AuthenticationValuesTest.kt` and `AccountUsageValuesTest.kt` gained
  wire-name/default assertions in `05a472e9`. Later scoped build changes
  retarget storage/Session and genuine app-component spec owners
  (`fc9f1936`, `05a472e9`); scoped main sources did not change.
- Source-set plugins/targets/exposure remain unchanged except the intentional
  Hook helper merge: its storage `api` and JSON-codec `implementation` edges
  moved into hooks; old `hook/tool-utils/build.gradle.kts` was deleted.
  KRPC dependency substitution retargeted the relocated patch [G1].
- Table covers **30 original topics, 29 current projects** (one Hook merge),
  including unchanged Session test support. `R` denotes a traced static path,
  **not** freshly executed tests or cross-platform acceptance. D/B1/B2/U use
  the parent definitions. Findings below are inherited implementation issues,
  not behavior changes introduced by physical relocation.

## 2. Complete topic table

FQCN shorthand: **`K = io.github.stream29.kodex.`** Every `K…` below denotes
that exact package prefix. Extension functions include their owning package
and receiver. Evidence references resolve to absolute file URIs with lines.
Test column records inspected source intent only; §5 records limits.

### Agent Context — introducing move `a6959173`

| Original → current project | Level | Original contract → actual impl → production caller | Tests / limits |
| --- | --- | --- | --- |
| `agent-context/contract` → `agent-context/spec/contract` | D | `Kagentcontext.contract.AgentContextSettings` and original source settings/plan [C1] → real `Krpc.server.ContextSnapshot : AgentContextSettings` [P2] → Session/AgentState request [P3]. Shell impl edge is known debt, not an unused spec. | Prefix resolver [CT1], AgentState projection [CT5]; not exhaustive source-toggle coverage. |
| `agent-context/prompt-dsl` → `agent-context/spec/prompt-dsl` | R | Original `Kagentcontext.promptdsl.PromptXmlBuilder/promptXml` [C2] are pure shared code → prefix/catalog renderer [C8] → Responses [P3]. No artificial interface required. | `PromptXmlTest` [CT2]: nesting, escaping, compact fragments. |
| `agent-context/prefix/agents-md/contract` → `agent-context/spec/agents-md` | R | Original `Kagentcontext.prefix.agentsmd.contract.AgentsMdInstruction/Instructions/Snapshot/Warning` [C3] → `loadAgentsMd(plan)` [C4] → resolver [C7] → rendering [P3]. No second document model. | File loader [CT3] and resolver [CT1]. |
| `agent-context/prefix/agents-md/filesystem` → `agent-context/impl/agents-md-filesystem` | R | `Kagentcontext.prefix.agentsmd.filesystem.loadAgentsMd` returns actual spec snapshot [C4]; resolver line 35 calls it [C7]. Scoped `useSource`, UTF-8/truncation/read warnings and cancellation check retained. | [CT3] uses real isolated files; no fresh cancellation/permission experiment. |
| `agent-context/prefix/skill/contract` → `agent-context/spec/available-skill` | R | `Kagentcontext.prefix.skill.contract.AvailableSkill/SkillSource/SkillScope/SkillWarning` [C5] → filesystem metadata [C10] → original prefix receiver [C6] → catalog renderer [C12]. Identity is path + source/authority, not name. | Discovery [CT4], renderer [CT8]. |
| `agent-context/prefix/contract` → `agent-context/spec/prefix` | D | Original `Kagentcontext.prefix.contract.AgentContextPrefix/AgentSessionMeta` [C6] is actual resolver return and renderer input [C7], [C8] → model request [P3]. Direct Shell impl edge retained [DG1]. | Resolver/render tests [CT1], [CT6]; no parallel prefix DTO. |
| `agent-context/prefix/impl` → `agent-context/impl/prefix` | R | `Kagentcontext.prefix.impl.AgentContextPrefixResolver` [C7] captures one settings snapshot/shared deduplicated source plan; AgentState owns/calls that exact concrete helper [P3]. It is not a new resolver substituting for a missed old interface. | [CT1] tests real files/cwd changes, not all custom/Codex/enablement combinations. |
| `agent-context/prefix/render` → `agent-context/impl/prefix-render` | R | Original `Kagentcontext.prefix.render.AgentContextPrefix.render`, `List<AvailableSkill>.render`, `renderPlanningInstructions` [C8], [C12], [C13] → actual AgentState request [P3]. Developer skills, user AGENTS/environment and URI/name remain original. | [CT6], [CT5] cover ordering/content and transient request projection. |
| `agent-context/skill/contract` → `agent-context/spec/skill` | R | `Kagentcontext.skill.contract.SkillsResolver/ResolvedSkills/SkillDocument/SkillResourceResult` [C9], [C14] → actual `FileSystemSkillsResolver`/private `FileSystemResolvedSkills` [C10]. Production consumes resolved catalog; document/resource APIs are retained non-CLI paths, not parallel state. | [CT4] exercises catalog, document/resources and traversal rejection. |
| `agent-context/skill/filesystem` → `agent-context/impl/skill-filesystem` | R | `Kagentcontext.skill.filesystem.FileSystemSkillsResolver : SkillsResolver`, private `FileSystemResolvedSkills : ResolvedSkills` [C10]. Prefix resolver constructs it/calls `resolve(plan)` [C7]. Fingerprint cache, bounded scans, captured authority and read results preserved. | [CT4] checks addition/deletion/rename, metadata replacement/malformed input/Home changes. Legacy `resolve(cwd)` tests are not all production-plan tests. |
| `agent-context/skill/render` → `agent-context/impl/skill-render` | D | Original `Kagentcontext.skill.render.SkillDocument.render` [C11] consumes real spec document/DSL, but **no production caller or incoming current build dependency found**. Present before migration (`1f585015`), retained in complete mapping [H1]; not invented parallel DTO. | `SkillDocumentRendererTest` only [CT7]. Smallest dead-code deletion candidate, subject to API/retention approval. |

### Agent Storage — introducing move `d05e9cdb`

| Original → current project | Level | Original contract → actual impl → production caller | Tests / limits |
| --- | --- | --- | --- |
| `agent-storage/clean-models` → `agent-storage/spec/clean-models` | R | Single original `Kagentstorage.cleanmodels.CleanOpenAiEvent`, `.stable.CleanIndexEntry/StableIndexEvent/StableWorkEvent/StableCleanEvent.CompletedTool`, `.unstable.UnstableCleanEvent/PendingToolEvent` hierarchy [S1]. Every moved event/failure/history/compaction/retention/projection/serializer blob retained → actual file serializers [S3], durable request projection [P3], RPC completion [P4]. | Stable/unstable union, specialized-tool/patch/golden suites; representative golden evidence [ST1]. Pure projections/custom serializers are valid contracts. |
| `agent-storage/contract` → `agent-storage/spec/contract` | R | `Kagentstorage.contract.KodexAgentStorage/MutableKodexAgentStorage`, `IndexVersioned/MutableIndexVersioned`, `CachedIndexVersioned/ObservableKodexAgentStorage`, `TokenCountSnapshot/Kind/Diagnostics` [S2], [S8] → file/memory writable impls [S3], [S4] → actual Session writable/observable cache [N3] and separate RPC read cache [Q3]. | File/memory/cache/token-value tests [ST2], [ST3], [NT2], [QT10]. Six timelines and pure traversal/latest/floor/ceil/revert/append extensions retained. |
| `agent-storage/contract-ext` → `agent-storage/spec/contract-ext` | R | Original `Kagentstorage.contract.ext` initialize, active-window/latest-compaction, append-compaction and retained-prefix/token-estimation functions [S5], [S9], [S10] operate on real spec storage → backend create [P5], AgentState [P3], actual AgentState compaction commit [P6]. No second owner. | `AgentStorageContractExtTest` [ST4] uses real memory storage; file/runtime tests cover initialization/compaction downstream. Impl edge is commonTest, not main. |
| `agent-storage/filesystem` → `agent-storage/impl/filesystem` | B1/B2 | `Kagentstorage.filesystem.FileSystemAgentStorage : MutableKodexAgentStorage`, `FileSystemIndexVersioned<T> : MutableIndexVersioned<T>` [S3], [S6]. Original URI platform sources, six directories/serializers/empty pointers/raw fork → actual Session factory [N6] → cache/RPC [N3], [P4]. B1-3 and B2-1/2 qualify implementation guarantees. | [ST2]: sparsity/latest fallback/publication ordering/failed revert; **not** crash-residual reopen or cleanup-error suppression. |
| `agent-storage/filesystem-layout` → `agent-storage/impl/filesystem-layout` | D | Original `Kagentstorage.filesystemlayout` stateless raw helpers [S7], concrete `CoroutineFileSystem`, primitive `IntArray`, caller-specified timelines → Home validation and frozen v0_3_3/v0_4_5 migration calls [P7]. Documented support exception [H2], [H6]; no current-business codec/unused shim. | Four `FileSystemLayoutTest` bodies [ST5]: historical sets/canonical names/raw operations/missing timeline. **No cancellation scenario** in that source. |
| `agent-storage/in-memory` → `agent-storage/impl/in-memory` | R | `Kagentstorage.inmemory.InMemoryKodexAgentStorage : MutableKodexAgentStorage`; private `InMemoryIndexVersioned : MutableIndexVersioned` [S4] → actual memory Session repository [N4]/isolated tests. Not default CLI storage or a promise of file observable metadata. | [ST3]: initialization, sparsity, revert/constraints. No file publication/lease conclusion inferred. |

### Agent Session — introducing move `b4cb23e2`

| Original → current project | Level | Original contract → actual impl → production caller | Tests / limits |
| --- | --- | --- | --- |
| `agent-session/contract` → `agent-session/spec/contract` | D | `Kagentsession.contract.KodexAgentSession/KodexSessionRepository/KodexRootSessionRepository`, entries and `KodexAgentDependencies` [N1], [N7] → real file/memory repositories [N2], [N4] → backend host's same root contract [P8]. Shell mixed edge/stale “seven timelines” KDoc, not competing authority. | [NT1], [NT3], host/file lifecycle tests [QT2], [QT11]; no fresh run. |
| `agent-session/filesystem` → `agent-session/impl/filesystem` | B1 | `Kagentsession.filesystem.FileSystemKodexSessionRepository : KodexRootSessionRepository`, internal `FileSystemKodexAgentSession : KodexAgentSession` [N2], [N6]; `CachedAgentStorage : MutableKodexAgentStorage, ObservableKodexAgentStorage`/`CachedIndexVersionedImpl` [N3] → actual backend graph [P1]. B1-1 create cleanup and B1-3 fork suppression gap. | [NT1]: lease/identity/reopen/cache/CAS/layout/archive/delete/fork/failed fork. [NT2]: cache/nonce/owner. No file-create failure before returned index. |
| `agent-session/in-memory` → `agent-session/impl/in-memory` | R | `Kagentsession.inmemory.InMemoryKodexSessionRepository : KodexRootSessionRepository`; private actual session/storage-lifetime wrappers [N4] compose real storage/runtime. Isolated Session/RPC callers only, not default `withBackendServices`. Wrappers enforce active owner, not copied truth. | [NT3] and `AgentRuntimeRunningTurnTest`; no file crash/lease inference. |
| `agent-session/test` → unchanged `agent-session/test` | D | Original `Kagentsession.test.testKodexAgentDependencies` [N5] constructs actual spec dependency bundle with mock clients/NoOp hooks. Callers are tests/dedicated test support, not CLI production. Explicit retained-path classification [H3], no empty spec. | Factory used by [NT1], [NT3], [QT2]; not a fourth production implementation. |

### Hook — introducing move `a6959173`

| Original → current project | Level | Original contract → actual impl → production caller | Tests / limits |
| --- | --- | --- | --- |
| `hook/contract` → `hook/spec/hooks` | R | Original `Khook.contract.KodexHooks/NoOpKodexHooks`, narrow turn/tool/compaction/error ports, contexts/settings/manager [J1], [J5]. CLI passes **actual spec NoOpKodexHooks** [P1]; runtime borrows narrow ports. Configured control types retained for approved historical tests [H4]. | [JT1], [JT2], runtime decorator sources; not proof CLI runs configured old hooks. |
| `hook/impl` → `hook/impl/hooks` | D | `Khook.impl.KodexHooksImpl : KodexHooks`, `HookManagerImpl : HookManager` [J2], [J6]; command/projection algorithms [J3], [J7]. **Neither configured factory is assembled by current CLI.** Current notification-only executor is a deliberately different protocol [P9], [H4]. Historical retention, not migration-added unused spec. | [JT1], [JT2] test historical impl. Live historical integration [JT3] bypasses CLI; not run. |
| `hook/tool-utils` → merged `hook/impl/hooks` | R | Original `Khook.toolutils.ToolHooks.runPreToolUse/runPostToolUse` [J4] consumes actual Hook/storage/events → actual runtime tool decorator [P10]. Passed hooks are NoOp in CLI, but helper remains in runtime call path. Old project removed; no extra port/copied DTO. | Runtime tool tests [JT4] exercise helper with recording hooks. Hook-module tests alone do **not** demonstrate former-helper coverage. |

### RPC — introducing move `a6959173`

| Original → current project | Level | Original contract → actual impl → production caller | Tests / limits |
| --- | --- | --- | --- |
| `rpc/models` → `rpc/spec/models` | D | Original `Krpc.models.BackendSettings`, `CliFrontendSettings/CliSidebarSettings`, `AgentStateValue`, `ShellSessionState`, OAuth target/authorization, created-suggestion and notification/hook unions [Q1], [Q7]. Actual server returns them [P1], [P4], [P5]; real local stores/notification consumer use frontend models [P9]. app-shared edge is mixed debt, not parallel DTO. | Seven model suites [QT3] and reused-value tests [QT1]; all main blobs unchanged, full credential-bearing MCP settings/defaults retained. |
| `rpc/contract` → `rpc/spec/contract` | D | `Krpc.contract.GlobalRpc/AgentRuntimeRpc`, six concrete `@Rpc` timeline services redeclare `TimelineRpc<T>`, local `SuspendMutableStateFlow` [Q2], [Q8]. Actual server implements/delegates these [P4], [P11], [P15], registers all eight [P1] → frontend generated `withService` [P12]. Stale review-only KDoc, no unused parallel service. | Value/boundary tests [QT1], complete real JSON services [QT4]; generated binary not regenerated here. |
| `rpc/client` → `rpc/impl/client` | R | `Krpc.client.RestoringRpcClient : RpcClient`, private `RpcCachedIndexVersioned : CachedIndexVersioned`, `rpcStateIn/asSuspendMutableStateFlow` [Q3], [Q9]. Actual Application wraps **before** proxy creation [P9], [P12] → frontend Session binding [P13]. CAS/cache wrappers do not create backend authority. | Client restoring/delegation/state/CAS/cache suites [QT5], generated token timeline [QT6]; many probes are fake services, not full CLI. |
| `rpc/server` → `rpc/impl/server` | B1 | `Krpc.server.BackendGlobalRpc : GlobalRpc`, `BackendAgentRuntimeRpc : AgentRuntimeRpc`, five read `Backend*TimelineRpc` + writable `BackendSettingsTimelineRpc`; Services/SessionHost/SessionManagement/GlobalState/AccountState/OAuth/Notifications [P1], [P4], [P5], [P8], [P11], [P14], [P15]. Actual CLI assembly [P9]; B1-2 eager repository and downstream B1-1 exposure. | [QT2], [QT4], [QT7], [QT11] include real storage/JSON and isolated time tests, but miss identified failure paths. No fake “complete GlobalRpc” found. |
| `rpc/in-memory` → `rpc/impl/in-memory` | R | Original `Krpc.inmemory.withInMemoryRpc`, `ChannelRpcTransport : kotlinx.rpc.krpc.KrpcTransport` [Q4], [Q10] implement real scoped JSON transport via upstream contracts → actual Application's `backend.register`/proxies [P9]. Not a missing business interface. | Transport/connection sources [QT8], complete registration [QT4]; no Native/JS reruns. |
| `rpc/krpc-utils-patch` → `rpc/impl/krpc-utils-patch` | D | Original `kotlinx.rpc.internal.utils` API replacement: `map.SynchronizedHashMap` snapshots accessors [Q5], Native `actual` [Q6] → Gradle substitution/KRPC [G1]. Explicit versioned support exception [H5]; upstream surface, not project-owned fake spec. | `SynchronizedHashMapTest` [QT9] and parent-cancel connection tests [QT8]. JVM source cannot verify pinned Native KLIB substitution/link/concurrency. |

## 3. Specific findings for coordinator adjudication

These are **inherited implementation semantics at pinned HEAD**, not
relocation-induced API/wire changes. Prior passing-test claims cannot close
paths that the tests do not exercise.

### B1-1 — failed file `create()` leaves its reserved Session directory

- `FileSystemKodexSessionRepository.create()` reserves the numeric directory
  at 144/174–187 then calls `ofEmpty` without catch/finally [N2].
  `ofEmpty` performs fallible suspend creation of six timelines/pointers [S3].
- Concrete path: fail/cancel at a later timeline/pointer → create throws
  before returning index → numeric partial directory remains → repository
  reopen includes it at 301–315 [N2]. Failed creation can consume a slot or
  appear as a persisted half-Session.
- Real caller `BackendSessionManagement.create` invokes `repository.create()`
  **outside** its later initialization cleanup try (148–160) [P5].
  It cannot clean the index it never obtained. Default backend uses this
  file repository [P1], [P9].
- Required behavior: complete reserved-directory cleanup in NonCancellable
  scope [F1], lines 60–64. Reservation path introduced `cc3000f8`, retained
  byte-identically by `b4cb23e2`; earlier create also lacked this cleanup.
- Existing file test checks successful empty storage at 233 [NT1] and failed
  **fork** at 733. RPC test named “creation failure cleans its reserved entry”
  instead mocks memory repository **open after successful create** [FT1].
- Smallest repair candidate: local catch around `ofEmpty`; clean only this
  successfully reserved directory, suppress cleanup failure onto original.
  Coordinator experiment: fault/cancel each file initialization boundary,
  verify no target remains and next index remains reusable. Not run here.

### B1-2 — backend construction eagerly creates/scans `sessions/`

- Actual Application calls backend services before renderer or any Session/
  catalog command [P9], 71–87. Services supplies file repository factory
  [P1], 68–70; host immediately executes it [P8], 59–61.
- File factory creates Home/`sessions/` and lists it unconditionally [N2],
  451–466. This is a filesystem side effect, not merely constructing values.
  It contradicts on-demand Home rule [F1], line 37; invalid entries can also
  prevent startup before Session/catalog access.
- Current production assembly introduced `8397a155`, before `a6959173`.
  No inspected task grants a specific on-demand exemption.
- Complete-services test immediately queries catalog/creates Session [QT4];
  it never asserts absence of `sessions/` at BackendReady.
- Smallest hard-cutover candidate: defer the **single** repository factory
  until first authorized Session/catalog use; retain initial scan and shutdown
  contract when opened. Not a second repository or DTO.
  Coordinator must first recheck whether the rule was separately waived,
  then verify BackendReady in an isolated prepared Home before repository use.

### B1-3 — cleanup failures can replace primary I/O error/cancellation

- `FileSystemIndexVersioned.setUnsafe` finally at 242–245 and
  `writeAtomically` finally at 315–323 directly call fallible
  `delete(temporary)` [S6]. Original failed/cancelled write/move plus throwing
  delete makes Kotlin finally replace the primary failure.
- `FileSystemKodexSessionRepository.createFork` catch directly cleans before
  rethrow, 168–170 [N2]; throwing recursive delete similarly replaces the
  materialization failure.
- Actual writes go through Session cache [N3], 256–280; GlobalRpc fork uses
  the real file repository [P5], 94–114. Violates primary/suppressed cleanup
  requirement [F2], not merely a test utility concern.
- Current staged set algorithm introduced `d9e27f77`; bare atomic-helper
  cleanup dates to `bb82aa3a` (NonCancellable wrapper added in `d9e27f77`).
  Fork cleanup introduced `7556e2a6`. Both were moved
  unchanged in `d05e9cdb`/`b4cb23e2`.
- Tests inject publication/revert failure [ST2], 163/184, or backing-cache
  failure [NT2], 163–184, **not simultaneous cleanup failure**.
- Smallest candidate: local primary-failure preservation + suppressed cleanup
  error; retain cancellation and successful operation behavior.
  Coordinator experiment: independently fail write/move/copy and cleanup
  deletion, assert primary type/identity and suppressed error. No live repro.

### B2-1 — pointer repair is not interrupted write/revert residue handling

- Direct file KDoc explicitly says interruption is **not repaired on open**,
  18–27; implementation creates `.kodex-write-*`,
  `.kodex-revert-pending-*` and `.kodex-revert-committed-*` [S6].
- Actual Session open enumerates numbered indexes/reconciles pointer [N3],
  105–116; no processing of those residual namespaces was found in that
  owner path. Missing/dangling pointer recovery **does exist**; it does not
  establish residue recovery/cleanup.
- Current Home requires handling owned residues [F1], 65–66. Historical
  guidance permits a legal prefix/no durable journal but also says incomplete
  temporaries are cleaned [F3]. Implementation KDoc and documents need
  reconciliation before choosing restore versus discard for pending revert.
- Historical path/KDoc predates moves (`d9e27f77` and earlier storage history).
  Do **not** transfer this question to frozen Home migrations: layout helpers'
  raw/nontransactional historical behavior is explicitly approved [H6].
- Needed experiment: isolated pending/committed/write residues → reopen actual
  file Session → compare numbered truth, pointer, chosen prefix semantics,
  cleanup and preservation of unknown files. No fixture/process crash run.
  Smallest candidate is timeline-owned namespace processing, not Home sweep.

### B2-2 — file and historical-layout numeric parsers diverge

- Layout parser requires `index.toString() == number`, 140–145 [S7].
  Runtime file parser accepts digit aliases such as `01.json`, returning 1,
  332–336 [S6]; later read/reconcile uses `1.json`, not the alias.
- Risk path: only `01.json` on disk → cache enumerates 1 → reconcile requires
  nonexistent `1.json` [N3], 109–110 / [S6], 256–262 → open fails.
  Layout enumeration treats alias as unknown and preserves it.
  Normal writes generate canonical names in both paths.
- Runtime parser introduced `bb82aa3a`; canonical layout rule introduced
  `81d353af`. Both predate `d05e9cdb` and are unchanged by that move.
- Layout canonical-name test [ST5], 51–64 is not runtime parser parity.
  Determine intended treatment of unknown/noncanonical entries before
  elevating this to B1. It is not relocation-induced serializer regression.
- Smallest candidate: align the owning numeric acceptance rule while
  preserving unknown content; do not alter frozen migration sources.
  Coordinator should compare enumerators on identical isolated filenames.

## 4. Compatibility, ownership and documented boundaries

- **Persistence:** actual file storage selects original index/work/settings/
  Instant/token-count/list-unstable serializers [S3], 26–67. Common
  `StableCleanEvent` is a projection; persisted sealed roots remain
  `CleanIndexEntry/StableWorkEvent` [S1]. Token count retains kind,
  `total_tokens`, `legacy/initialization/compaction/response` names and null
  usage/diagnostics defaults [S8]. Representative pre-move goldens assert
  `suggest_subagent_task_tool_event`, `user_message`, `compaction_point`,
  `text_tool_event`, `context_compaction`, `command_execution_tool_event`,
  `call_id` and `encrypted_content` [ST1]. URI expect/actual and all
  event/custom-codec/default blobs are unchanged, not replaced DTOs.
- **Wire/defaults:** original models, sealed names, validation/default
  expressions and concrete `@Rpc` methods remain identical. BackendSettings
  fields are required; frontend ShiftEnter/sidebar/hooks defaults remain local
  [Q1], [Q7]. Later authentication/usage tests assert original FQCN
  discriminators `io.github.stream29.kodex.app.settings.contract.` +
  `SettingsAuthenticationState.Authenticated` and
  `SettingsAccountUsageState.{Unavailable,Loading,Failed}` [QT1], [QT12];
  those values are genuinely reused component specs, not fresh RPC DTOs.
- **Latest layout:** six empty timelines/pointers initialize to -1 [S3],
  109–126. Append publishes pointer before numbered record, compensating it
  on failure [S6], 221–241. Catalog valid-pointer fast path reads without
  lease/scan; invalid-pointer handling repairs under temporary lease, or
  scans without repair if lease held elsewhere [N2], 371–436. Full Session
  cache scans/reconciles once [N3], 105–116. B2-1 qualifies crash claims.
- **Borrowing:** Session/runtime borrows process-wide services and never
  closes them [N7], [N6], [N4]. Host joins actual Session/repository tasks
  before global resource/client teardown [P1], [P8], [P14].
  `KodexAgentDependencies` is a resource bundle, not a second graph;
  current backend closes the actual owning components, not borrowed copies.
- **Lease/cache:** Session owns renewable `lock.json` child/30-second lease,
  distinct from RPC TTL [N6], [N2]. Writable/observable cache views share one
  object [N3]; frontend is a different read-only cache impl [Q3].
  Append preserves nonce, destructive revert changes it, no-op preserves it
  [N3], 283–301. Catalog lease cleanup explicitly joins lease Job, not the
  NonCancellable caller Job, 418–425 [N2].
- **Accepted work:** host owns accepted commands before await; cancelled RPC
  waiter is not Stop [P8], 121–143. Only runningTurn renews 60-second
  residency at 20-second intervals, including cancellation cleanup;
  deactivation cancels/joins accepted operations before storage owner,
  197–256. Bound observation ends with SessionNotActive on deactivation;
  a collector does not close its shared entity.
- **Client/transport:** restoring client uses upstream `buffer(0)` and catch,
  borrows connection and does not decode downstream consumer failure [Q9].
  Two 16-message channels/fixed JSON/scoped endpoint jobs are real
  implementation [Q4], [Q10]; monitors stop before intentional teardown,
  10-second cleanup preserves primary failure. Bounds are neither business
  timeout nor encoded-byte bound.
- **Server topics:** GlobalRpc uses actual settings/file/manager CAS,
  MCP reconnect/import readers, original account stores and summary mapping
  [P11], [P14], [P16]. OAuth binds target/state/redirect/attempt identity,
  accepts code once and expires pending callback phase [P17].
  Notification is replay 0/64/DROP_OLDEST, not persistent state [P18].
  Output nonce/source captured in one snapshot [P4], 226–261 is approved wire
  projection, not competing AgentState.
- **Historical Hook boundary:** checklist expressly selects NoOp + frontend
  notification-only for CLI [H4]. Real configured executor/manager remains
  historical/test assembly [J2], [J6]. Snapshot/serial control ordering,
  concurrent observation, strict fail-open JSON, 600-second timeout,
  cancellation rethrow and old error plaintext stdin retained [J2], [J3],
  [J7]. New frontend notification protocol must not be measured against old
  control-script compatibility. Tool-utils merged under actual hooks spec,
  and actual runtime uses it [J4], [P10], [H1].
- **Support exceptions:** patch and filesystem-layout are expressly recorded
  support [H1], [H2], [H5], [H6]. Generic pairing policy is not proof every
  raw helper needs a fabricated interface. Layout avoids current clean-model/
  business codecs; patch substitutes original upstream API and Native KLIB
  identity `org.jetbrains.kotlinx:utils` [G2], not project RPC semantics.
  Session test support stays test-only
  at its explicitly recorded path [H3].
- **Dependency debts:** context contract/prefix expose
  `utils-shell-client-impl`; Session contract likewise [DG1], [DG2].
  RPC models expose mixed app-shared settings contract [DG4].
  Clean-models patch/file fixture edges, contract-ext memory impl edge and
  contract RPC framework fixtures are **test-only** [DG3], [DG5], [DG6].
  Session impl depends on unmigrated AgentState; server composes app-shared
  auth/settings/title. These edges alone are D, not migrated-root B1 or
  unused parallel specs.
- **KDoc debt:** Session still claims “seven” timelines [N1], 10–19, while
  storage has six. GlobalRpc still says its review contract has no client/server
  implementation [Q2], 21–28, contradicted by real registration [P1].
  Old AGENTS source-order comments describe four endpoints [C3], while
  production plan supports additional enabled roots [C7].
  Moves preserved stale docs; later documentation decisions must not infer
  nonexistent implementations or add artificial ports.

## 5. Test claims and explicitly unreviewed validation

- Prior checkpoints report JVM tests/selected downstream compilation passing
  and Native/JS/CLI not rerun [H7], [H3]; storage parent reports 55 tests [H2].
  **No results/counts reproduced here.** Source assertions/unchanged blobs
  establish intended coverage, not execution success.
- Synthetic pre-migration goldens [ST1] are stronger than round-trip alone,
  but representative, not every persisted historical event/file or platform
  descriptor. Actual spec serializers are tested; tests are not proof of all
  assembly/callback paths.
- Context real-file/resolver/AgentState projection evidence exists [CT1],
  [CT3], [CT4], [CT5]; all source permutations, warning consumption and
  per-boundary cancellation remain unverified.
- Prior “Hook implementation tests… including former tool-utils behavior”
  [H7], 164–165 cannot be attributed to that module alone:
  ConfiguredHooksTest directly calls hook requests; manager tests call manager.
  **Actual helper coverage is runtime tool tests** [JT4].
  Historical live integration [JT3] assembles its own configured services and
  real provider, not current CLI. No live test executed.
- Layout test source has four bodies and **no cancellation case** [ST5].
  File/session pointer/lease/fork tests do not cover B1-1/B1-3/B2-1.
  RPC “creation failure” mocks failing open after memory create [FT1];
  do not advertise it as failed file-create cleanup coverage.
- Complete backend test uses **real `withBackendServices`, file storage,
  generated services and JSON transport** [QT4]; only external model/login
  dependencies are mocks. Not fake full GlobalRpc.
  Host tests mix memory/fake-time and real file-lease evidence [QT2], [QT11].
  Client cache probes [QT5] plus generated token round-trip [QT6] are not
  actual CLI renderer acceptance.
- **U, not passed:** fresh JVM tests/reports; Gradle resolution/discovery;
  generated RPC compilation; Native KLIB/link/parent-cancellation regression;
  JS/Windows/macOS execution; real CLI lifecycle; live model/OAuth/MCP/scripts;
  fault/crash file fixtures; all coordinator experiments in §3.
  All 30 moved/original *topics* have static classifications; none is
  implicitly accepted because runtime validation is U.
- AgentState/app-shared were inspected only for real callers/debts, not
  independently accepted as migrated roots. Frozen Home migration payloads
  are not reimplemented or reaccepted by this report.

## 6. Smallest deletion / hard-cutover candidates

- **No duplicate migrated authority identified:** old declarations physically
  moved; no alias/adapter/forwarding project keeps these original roots alive.
  Real RPC/file adapters and pure contract extensions are not deletion targets.
- **Smallest dead-code candidate:** `agent-context/impl/skill-render` and its
  isolated renderer test [C11], [CT7], lacking incoming production dependency/
  caller. Retained SkillDocument/read APIs are a separate API decision.
  Approval required; nothing deleted.
- **Historical candidate:** configured Hook manager/executor/private command
  codecs/legacy references only after explicit retention and fixture review.
  Keep live NoOp/narrow ports and runtime tool projections; do not reactivate
  controls via frontend notifications. Larger than skill renderer, not a
  physical migration failure by itself.
- **Conditional patch cutover:** only remove after actual dependency's Native
  accessor/cancellation behavior is verified; update single substitution,
  rerun cross-target/CLI regression [H5]. Never load both original and patched
  Native utils KLIB.
- **Behavior candidates:** B1-1 local reserved-target cleanup; B1-3 local
  primary/suppressed handling; B1-2 one deferred repository owner.
  B2 residue/parser choices need coordinator clarification/tests first.
  Do not alter frozen migrations or expand audit into fixes.
- **REVIEW READY:** coordinator independently checks blockers and separates
  relocation identity from inherited implementation acceptance gates.
  This report authorizes no source edits, fixes, builds, commit or push.

## Evidence index

[G1]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/build.gradle.kts#L3-L9
[G2]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/krpc-utils-patch/build.gradle.kts#L5-L10
[C1]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-context/spec/contract/src/commonMain/kotlin/io/github/stream29/kodex/agentcontext/contract/AgentContextSettings.kt#L18-L54
[C2]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-context/spec/prompt-dsl/src/commonMain/kotlin/io/github/stream29/kodex/agentcontext/promptdsl/PromptXml.kt#L7-L82
[C3]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-context/spec/agents-md/src/commonMain/kotlin/io/github/stream29/kodex/agentcontext/prefix/agentsmd/contract/AgentsMdModels.kt#L11-L62
[C4]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-context/impl/agents-md-filesystem/src/commonMain/kotlin/io/github/stream29/kodex/agentcontext/prefix/agentsmd/filesystem/FileSystemAgentsMd.kt#L43-L142
[C5]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-context/spec/available-skill/src/commonMain/kotlin/io/github/stream29/kodex/agentcontext/prefix/skill/contract/AvailableSkillModels.kt#L6-L48
[C6]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-context/spec/prefix/src/commonMain/kotlin/io/github/stream29/kodex/agentcontext/prefix/contract/AgentContextPrefix.kt#L9-L29
[C7]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-context/impl/prefix/src/commonMain/kotlin/io/github/stream29/kodex/agentcontext/prefix/impl/AgentContextPrefixResolver.kt#L23-L99
[C8]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-context/impl/prefix-render/src/commonMain/kotlin/io/github/stream29/kodex/agentcontext/prefix/render/AgentContextPrefixRenderer.kt#L17-L51
[C9]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-context/spec/skill/src/commonMain/kotlin/io/github/stream29/kodex/agentcontext/skill/contract/SkillsResolver.kt#L7-L24
[C10]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-context/impl/skill-filesystem/src/commonMain/kotlin/io/github/stream29/kodex/agentcontext/skill/filesystem/FileSystemSkillsResolver.kt#L33-L299
[C11]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-context/impl/skill-render/src/commonMain/kotlin/io/github/stream29/kodex/agentcontext/skill/render/SkillDocumentRenderer.kt#L4-L17
[C12]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-context/impl/prefix-render/src/commonMain/kotlin/io/github/stream29/kodex/agentcontext/prefix/render/AvailableSkillRenderer.kt#L9-L28
[C13]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-context/impl/prefix-render/src/commonMain/kotlin/io/github/stream29/kodex/agentcontext/prefix/render/PlanningInstructionsRenderer.kt#L3-L38
[C14]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-context/spec/skill/src/commonMain/kotlin/io/github/stream29/kodex/agentcontext/skill/contract/SkillModels.kt#L6-L20
[CT1]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-context/impl/prefix/src/commonTest/kotlin/io/github/stream29/kodex/agentcontext/prefix/impl/AgentContextPrefixResolverTest.kt#L20-L135
[CT2]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-context/spec/prompt-dsl/src/commonTest/kotlin/io/github/stream29/kodex/agentcontext/promptdsl/PromptXmlTest.kt#L7-L42
[CT3]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-context/impl/agents-md-filesystem/src/commonTest/kotlin/io/github/stream29/kodex/agentcontext/prefix/agentsmd/filesystem/FileSystemAgentsMdTest.kt#L14-L139
[CT4]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-context/impl/skill-filesystem/src/commonTest/kotlin/io/github/stream29/kodex/agentcontext/skill/filesystem/FileSystemSkillsResolverTest.kt#L23-L285
[CT5]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-state/impl/src/commonTest/kotlin/io/github/stream29/kodex/agentstate/impl/AgentContextProjectionTest.kt#L44-L178
[CT6]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-context/impl/prefix-render/src/commonTest/kotlin/io/github/stream29/kodex/agentcontext/prefix/render/AgentContextPrefixRendererTest.kt#L23-L124
[CT7]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-context/impl/skill-render/src/commonTest/kotlin/io/github/stream29/kodex/agentcontext/skill/render/SkillDocumentRendererTest.kt#L12-L46
[CT8]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-context/impl/prefix-render/src/commonTest/kotlin/io/github/stream29/kodex/agentcontext/prefix/render/AvailableSkillRendererTest.kt#L10-L37
[S1]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-storage/spec/clean-models/src/commonMain/kotlin/io/github/stream29/kodex/agentstorage/cleanmodels/stable/StableCleanEvent.kt#L6-L22
[S2]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-storage/spec/contract/src/commonMain/kotlin/io/github/stream29/kodex/agentstorage/contract/KodexAgentStorage.kt#L16-L78
[S3]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-storage/impl/filesystem/src/commonMain/kotlin/io/github/stream29/kodex/agentstorage/filesystem/FileSystemAgentStorage.kt#L18-L147
[S4]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-storage/impl/in-memory/src/commonMain/kotlin/io/github/stream29/kodex/agentstorage/inmemory/InMemoryKodexAgentStorage.kt#L25-L191
[S5]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-storage/spec/contract-ext/src/commonMain/kotlin/io/github/stream29/kodex/agentstorage/contract/ext/AgentStorageInitialization.kt#L12-L28
[S6]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-storage/impl/filesystem/src/commonMain/kotlin/io/github/stream29/kodex/agentstorage/filesystem/FileSystemIndexVersioned.kt#L18-L350
[S7]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-storage/impl/filesystem-layout/src/commonMain/kotlin/io/github/stream29/kodex/agentstorage/filesystemlayout/FileSystemLayout.kt#L10-L145
[S8]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-storage/spec/contract/src/commonMain/kotlin/io/github/stream29/kodex/agentstorage/contract/TokenCountSnapshot.kt#L7-L56
[S9]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-storage/spec/contract-ext/src/commonMain/kotlin/io/github/stream29/kodex/agentstorage/contract/ext/AgentStorageProjection.kt#L11-L69
[S10]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-storage/spec/contract-ext/src/commonMain/kotlin/io/github/stream29/kodex/agentstorage/contract/ext/CompactionStorage.kt#L12-L33
[ST1]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-storage/spec/clean-models/src/commonTest/kotlin/io/github/stream29/kodex/agentstorage/cleanmodels/stable/StableTimelineGoldenTest.kt#L7-L58
[ST2]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-storage/impl/filesystem/src/commonTest/kotlin/io/github/stream29/kodex/agentstorage/filesystem/FileSystemIndexVersionedTest.kt#L36-L211
[ST3]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-storage/impl/in-memory/src/commonTest/kotlin/io/github/stream29/kodex/agentstorage/inmemory/InMemoryKodexAgentStorageTest.kt#L1-L150
[ST4]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-storage/spec/contract-ext/src/commonTest/kotlin/io/github/stream29/kodex/agentstorage/contract/ext/AgentStorageContractExtTest.kt#L89-L159
[ST5]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-storage/impl/filesystem-layout/src/commonTest/kotlin/io/github/stream29/kodex/agentstorage/filesystemlayout/FileSystemLayoutTest.kt#L41-L93
[N1]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-session/spec/contract/src/commonMain/kotlin/io/github/stream29/kodex/agentsession/contract/KodexSession.kt#L10-L145
[N2]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-session/impl/filesystem/src/commonMain/kotlin/io/github/stream29/kodex/agentsession/filesystem/FileSystemKodexSessionRepository.kt#L39-L473
[N3]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-session/impl/filesystem/src/commonMain/kotlin/io/github/stream29/kodex/agentsession/filesystem/CachedAgentStorage.kt#L33-L307
[N4]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-session/impl/in-memory/src/commonMain/kotlin/io/github/stream29/kodex/agentsession/inmemory/InMemoryKodexSessionRepository.kt#L31-L204
[N5]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-session/test/src/commonMain/kotlin/io/github/stream29/kodex/agentsession/test/TestKodexAgentDependencies.kt#L15-L47
[N6]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-session/impl/filesystem/src/commonMain/kotlin/io/github/stream29/kodex/agentsession/filesystem/FileSystemKodexAgentSession.kt#L21-L65
[N7]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-session/spec/contract/src/commonMain/kotlin/io/github/stream29/kodex/agentsession/contract/KodexAgentDependencies.kt#L12-L49
[NT1]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-session/impl/filesystem/src/commonTest/kotlin/io/github/stream29/kodex/agentsession/filesystem/FileSystemKodexSessionRepositoryTest.kt#L233-L768
[NT2]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-session/impl/filesystem/src/commonTest/kotlin/io/github/stream29/kodex/agentsession/filesystem/CachedAgentStorageTest.kt#L42-L296
[NT3]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-session/impl/in-memory/src/commonTest/kotlin/io/github/stream29/kodex/agentsession/inmemory/InMemoryKodexSessionRepositoryTest.kt#L1-L170
[J1]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/hook/spec/hooks/src/commonMain/kotlin/io/github/stream29/kodex/hook/contract/KodexHooks.kt#L15-L31
[J2]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/hook/impl/hooks/src/commonMain/kotlin/io/github/stream29/kodex/hook/impl/KodexHooksImpl.kt#L41-L225
[J3]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/hook/impl/hooks/src/commonMain/kotlin/io/github/stream29/kodex/hook/impl/ShellClientHook.kt#L25-L90
[J4]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/hook/impl/hooks/src/commonMain/kotlin/io/github/stream29/kodex/hook/toolutils/ToolHookUtils.kt#L20-L147
[J5]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/hook/spec/hooks/src/commonMain/kotlin/io/github/stream29/kodex/hook/contract/HookSettings.kt#L6-L49
[J6]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/hook/impl/hooks/src/commonMain/kotlin/io/github/stream29/kodex/hook/impl/HookManagerImpl.kt#L19-L133
[J7]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/hook/impl/hooks/src/commonMain/kotlin/io/github/stream29/kodex/hook/impl/projection/HookProjection.kt#L15-L66
[JT1]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/hook/impl/hooks/src/commonTest/kotlin/io/github/stream29/kodex/hook/impl/ConfiguredHooksTest.kt#L169-L498
[JT2]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/hook/impl/hooks/src/commonTest/kotlin/io/github/stream29/kodex/hook/impl/HookManagerImplTest.kt#L21-L111
[JT3]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/integration-test/src/commonTest/kotlin/io/github/stream29/kodex/integrationtest/HooksIntegrationTest.kt#L55-L125
[JT4]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-runtime/impl/decorator/tool/src/commonTest/kotlin/io/github/stream29/kodex/agentruntime/decorator/tool/KodexToolRuntimeTest.kt#L379-L639
[Q1]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/spec/models/src/commonMain/kotlin/io/github/stream29/kodex/rpc/models/BackendSettings.kt#L11-L37
[Q2]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/spec/contract/src/commonMain/kotlin/io/github/stream29/kodex/rpc/contract/GlobalRpc.kt#L21-L80
[Q3]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/client/src/commonMain/kotlin/io/github/stream29/kodex/rpc/client/RpcCachedIndexVersioned.kt#L31-L208
[Q4]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/in-memory/src/commonMain/kotlin/io/github/stream29/kodex/rpc/inmemory/WithInMemoryRpc.kt#L22-L102
[Q5]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/krpc-utils-patch/src/commonMain/kotlin/kotlinx/rpc/internal/utils/map/SynchronizedHashMap.kt#L1-L67
[Q6]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/krpc-utils-patch/src/nativeMain/kotlin/kotlinx/rpc/internal/utils/map/ConcurrentHashMap.native.kt#L1-L14
[Q7]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/spec/models/src/commonMain/kotlin/io/github/stream29/kodex/rpc/models/CliFrontendSettings.kt#L6-L28
[Q8]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/spec/contract/src/commonMain/kotlin/io/github/stream29/kodex/rpc/contract/TimelineRpc.kt#L1-L95
[Q9]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/client/src/commonMain/kotlin/io/github/stream29/kodex/rpc/client/RestoringRpcClient.kt#L10-L36
[Q10]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/in-memory/src/commonMain/kotlin/io/github/stream29/kodex/rpc/inmemory/ChannelRpcTransport.kt#L10-L28
[QT1]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/spec/contract/src/commonTest/kotlin/io/github/stream29/kodex/rpc/contract/AccountUsageValuesTest.kt#L56-L146
[QT2]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/server/src/commonTest/kotlin/io/github/stream29/kodex/rpc/server/BackendSessionHostTest.kt#L57-L489
[QT3]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/spec/models/src/commonTest/kotlin/io/github/stream29/kodex/rpc/models/SettingsModelsTest.kt#L1-L103
[QT4]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/server/src/commonTest/kotlin/io/github/stream29/kodex/rpc/server/BackendServicesTest.kt#L32-L181
[QT5]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/client/src/commonTest/kotlin/io/github/stream29/kodex/rpc/client/RpcCachedIndexVersionedTest.kt#L36-L430
[QT6]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/client/src/commonTest/kotlin/io/github/stream29/kodex/rpc/client/TokenCountTimelineRpcTest.kt#L23-L79
[QT7]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/server/src/commonTest/kotlin/io/github/stream29/kodex/rpc/server/BackendOAuthTest.kt#L46-L247
[QT8]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/in-memory/src/commonTest/kotlin/io/github/stream29/kodex/rpc/inmemory/InMemoryRpcTest.kt#L95-L371
[QT9]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/krpc-utils-patch/src/commonTest/kotlin/kotlinx/rpc/internal/utils/map/SynchronizedHashMapTest.kt#L1-L43
[QT10]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/spec/contract/src/commonTest/kotlin/io/github/stream29/kodex/rpc/contract/TokenCountValuesTest.kt#L18-L66
[QT11]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/server/src/commonTest/kotlin/io/github/stream29/kodex/rpc/server/BackendSessionHostFileTest.kt#L1-L72
[QT12]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/spec/contract/src/commonTest/kotlin/io/github/stream29/kodex/rpc/contract/AuthenticationValuesTest.kt#L16-L63
[P1]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/server/src/commonMain/kotlin/io/github/stream29/kodex/rpc/server/BackendServices.kt#L35-L124
[P2]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/server/src/commonMain/kotlin/io/github/stream29/kodex/rpc/server/BackendGlobalState.kt#L69-L150
[P3]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-state/impl/src/commonMain/kotlin/io/github/stream29/kodex/agentstate/impl/KodexAgentStateImpl.kt#L132-L194
[P4]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/server/src/commonMain/kotlin/io/github/stream29/kodex/rpc/server/BackendAgentRuntimeRpc.kt#L38-L268
[P5]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/server/src/commonMain/kotlin/io/github/stream29/kodex/rpc/server/BackendSessionManagement.kt#L80-L161
[P6]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-state/impl/src/commonMain/kotlin/io/github/stream29/kodex/agentstate/impl/KodexAgentStateImpl.kt#L625-L649
[P7]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/migration/impl/src/commonMain/kotlin/io/github/stream29/kodex/app/migration/KodexHomeMigration.kt#L167-L192
[P8]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/server/src/commonMain/kotlin/io/github/stream29/kodex/rpc/server/BackendSessionHost.kt#L35-L303
[P9]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/viewmodel/application/src/commonMain/kotlin/io/github/stream29/kodex/cli/app/Application.kt#L55-L180
[P10]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-runtime/impl/decorator/tool/src/commonMain/kotlin/io/github/stream29/kodex/agentruntime/decorator/tool/KodexToolRuntimeImpl.kt#L105-L127
[P11]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/server/src/commonMain/kotlin/io/github/stream29/kodex/rpc/server/BackendGlobalRpc.kt#L17-L65
[P12]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/viewmodel/rpc/src/commonMain/kotlin/io/github/stream29/kodex/cli/rpc/RpcServices.kt#L14-L24
[P13]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/viewmodel/rpc/src/commonMain/kotlin/io/github/stream29/kodex/cli/rpc/RpcSessionBinding.kt#L85-L130
[P14]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/server/src/commonMain/kotlin/io/github/stream29/kodex/rpc/server/BackendGlobalState.kt#L87-L150
[P15]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/server/src/commonMain/kotlin/io/github/stream29/kodex/rpc/server/BackendTimelines.kt#L18-L70
[P16]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/server/src/commonMain/kotlin/io/github/stream29/kodex/rpc/server/BackendAccountState.kt#L23-L72
[P17]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/server/src/commonMain/kotlin/io/github/stream29/kodex/rpc/server/BackendOAuth.kt#L53-L202
[P18]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/server/src/commonMain/kotlin/io/github/stream29/kodex/rpc/server/BackendNotifications.kt#L14-L44
[DG1]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-context/spec/prefix/build.gradle.kts#L7-L11
[DG2]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-session/spec/contract/build.gradle.kts#L7-L16
[DG3]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-storage/spec/clean-models/build.gradle.kts#L8-L24
[DG4]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/spec/models/build.gradle.kts#L7-L15
[DG5]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-storage/spec/contract-ext/build.gradle.kts#L7-L14
[DG6]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/spec/contract/build.gradle.kts#L9-L28
[FT1]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/server/src/commonTest/kotlin/io/github/stream29/kodex/rpc/server/BackendSessionManagementTest.kt#L220-L257
[F1]: file:///home/stream/ACodeSpace/push/BuildKodex/checklist/kodex-home.md#L32-L71
[F2]: file:///home/stream/ACodeSpace/push/BuildKodex/checklist/coroutine-resource-lifecycle.md#L3-L11
[F3]: file:///home/stream/ACodeSpace/push/BuildKodex/shared-context/findings/agent-storage-compensation-semantics.md#L23-L35
[H1]: file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-01-migrate-agent-context-rpc-hook-roots.md#L50-L125
[H2]: file:///home/stream/ACodeSpace/push/BuildKodex/kanban/executable/2026-09-28-plan-domain-spec-impl-boundaries.md#L80-L94
[H3]: file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-02-migrate-agent-session-root.md#L95-L113
[H4]: file:///home/stream/ACodeSpace/push/BuildKodex/checklist/hooks.md#L1-L6
[H5]: file:///home/stream/ACodeSpace/push/BuildKodex/checklist/rpc-architecture.md#L53-L57
[H6]: file:///home/stream/ACodeSpace/push/BuildKodex/checklist/kodex-home.md#L115-L130
[H7]: file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-01-migrate-agent-context-rpc-hook-roots.md#L173-L189

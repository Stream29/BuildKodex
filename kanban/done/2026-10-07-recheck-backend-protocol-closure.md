# REVIEW READY — independent Backend/protocol repair re-review

- 审查脚本已完成并归档；本报告对应第一固定快照，原缺陷与 B2 保留，
  后续修复与运行证据见 [当前主任务](2026-10-07-plan-spec-impl-closure-and-consistency.md)。

- **Scoped verdict: NOT ACCEPTED yet.** One confirmed B1 remains in retryable
  repository acquisition. The repaired MCP identity/admission, auth primary-error,
  Runtime primary-cancellation and SearchIndex input boundaries are statically
  supported. RPC teardown has a separately labelled B2 reporting-race suspicion.
- Reviewer did not implement these repairs. This is a fixed-source, read-only
  review, not a build/test pass or whole-repository acceptance.

# Task Tree

- `Independently review fixed backend assembly and command boundaries`()
- `Trace real production contracts, captured identities and errors()`
- `Report defects and test limitations without inheriting prior passes()`

# Details

- 用户授权修复后重新审查；本任务只读，不授权继续修复或提交。
- 固定快照与统一证据见
  [修复主任务](2026-10-07-plan-spec-impl-closure-and-consistency.md)。
- 独占审查：Backend 按需 repository、认证清理/代次、MCP admission/连接目录/
  等待认证工具、Runtime cancellation、in-memory RPC teardown、SearchIndex。
- 验证唯一原 owner、真实装配、SDK/原工具路由及异常出口；区分资源结果值和
  新 owner 包装，检查关闭阶段窄异常过滤是否遮蔽运行期或非目标故障。
- 保持用户决定的 compaction 默认 20 与工具能力收紧；原 wire/raw name 不变。
- 只改本文件报告；不构建或跑服务，不改原审查/实现任务。每项列准确证据、
  准入与释放路径、测试不足、最小消融。疑点不作为确认故障。

## Independent report

### Provenance and evidence rules

- Independently verified [manifest][manifest] SHA-256:
  `3d661ec7abdf3e33a8de2b103ad7a19b8504cfd2b8a1168f00980fbd7765335e`.
  Rehashed **all 1290 files**, including every scoped source/test/build file:
  no mismatches, missing entries or unlisted files. These counts establish input
  integrity, not behavioral acceptance.
- Current source exclusively comes from
  `file:///tmp/kodex-closure-review-6b7129fa-20261007/`.
  Each source URI below points there, with snapshot line numbers. Map its relative
  suffix to `file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/` for the original
  project URI; that original worktree is DIRTY and was **not** read as current source.
- Original Git history baseline:
  `6b7129fa0e719dde8f88a0034b9d572679984a03`
  (`test: verify History viewport ownership and Home lease lifecycle`).
  Introducing diffs below mean `git show <baseline>:<relative path>` versus the
  snapshot bytes, not an attribution to an uncommitted repair commit.
- MCP external implementation inspected by `git show` at original gitlink
  `53f717661712d618cd4822d1760392e903989ead`, not its mutable checkout.
  kRPC dependency is pinned to `0.10.3` in the snapshot version catalog; local
  upstream tag resolves to `fc69ae6288cb4c8596493cd0b7ea2d79cac78f0d`.
  Its source was inspected at that commit. No claim about freshly resolved
  Gradle artifacts or all external/transitive dependencies.
- External URI mapping: [KrpcConnector.kt:76,268–280][krpcconnector],
  [KrpcSendHandler.kt:73–83][krpcsender] and
  [KrpcEndpoint.kt:19–54][krpcendpoint] name local upstream locations, with
  line numbers from `git show fc69ae6288cb4c8596493cd0b7ea2d79cac78f0d:<path>`.
  They are **not** claims about current mutable upstream checkout bytes.
- Loaded associated change/kanban/planning/checklist/document skills, Draft,
  parent executable and this task; consulted MCP client/management, RPC MCP/auth/
  architecture/settings/Session/Runtime, AgentState/Runtime, Home, lifecycle,
  spec/impl and search checklists. No previous READY verdict was used as evidence.
- Only this report was written, using `apply_patch`. No source fixes, other
  documents/reports, builds, test runs, live services, network, IDE/windows/device
  control, commit/switch/push or owned temporary resources.

Selected independently matched scoped SHA-256 values:

| Snapshot file | SHA-256 |
| --- | --- |
| [BackendSessionHost.kt][host] | `f152d0193fac47f864a3a5b99ac6d8f8894e9c6bcf7eb75fc23f6bec65ba30f5` |
| [FileSystemKodexSessionRepository.kt][fsrepo] | `eebdd676a1089ca44f6b1f0995a5b0aabc471627c27267809a315941b025816b` |
| [BackendFileSystemAuthStore.kt][auth] | `4a090b969081b7abc8cba8d86f9d5818d81b58e2c954cb7391d797d0ff948346` |
| [McpClientImpl.kt][mcpowner] | `6c177c7dd851b6577fb04f7ab451553fef0de678cb7bf57e82f8b1b8a07d3d76` |
| [McpServiceImpl.kt][mcpservice] | `57d79721f1803248e922eee6b072dea40185886f2ced872e356899a262aee507` |
| [ChannelRpcTransport.kt][transport] | `2cd43bcd02d0320f90e5466c23c72d14806d935cecbc0235b77a59d31e41e2da` |
| [WithInMemoryRpc.kt][connection] | `a995758f296f50c8bcbfad88db384bf228fbf724adfd19604b1577c867b3c7fe` |
| [KodexAgentRuntimeComposition.kt][runtime] | `6fd2c25d0d51ee529b893087e1813caab22ec8f8c0dc9d8014d6fc14658111e6` |
| [SearchIndexFactory.kt][searchfactory] | `5114282dde92dc7fc73825f91ceeaa1eaf13fe9accfe9a30dc5b6867ebc29b7c` |

### Classification by owned topic

- **R**: repaired/resolved boundary supported by inspected code and test intent,
  not a newly executed test pass.
- **B1**: confirmed defect, including deterministic static proof.
- **B2**: specific suspected defect requiring a controlled experiment.
- **D**: bounded design/documentation debt; **U**: evidence/coverage uncertainty.

| Owned topic | R | Confirmed B1 | Suspected B2 | D | U |
| --- | --- | --- | --- | --- | --- |
| Backend lazy repository/management/services | Original owner and typed factory; startup remains lazy | B1-1: failed scan leaves child until host shutdown | None additionally established | Repeated `repository()` creates accepted async operations, not a new owner | Failure/retry with real factory and surviving host lacks a gate |
| Backend auth cleanup/generation | Primary/suppressed and bound two-source authority | None found in repaired branch | None newly established | Failed cleanup may leave its temporary; no new recovery guarantee | Injected CE + IO differs from real cancelled Job + cleanup IO |
| MCP route admission and raw wire | Collision rejection at correct boundaries | None found in repaired branch | None newly established | New pure guard documents “Throws”, not checklist-required `@throws` | Paginated duplicate validation traced, not executed here |
| MCP SDK catalog/refresh/reconnect | Exact SDK identity under original locks | None found in repaired branch | None for the cross-reconnect late-refresh case | Same-SDK concurrent refresh ordering is not upgraded to total ordering | Owner gate is not a full service-transition test |
| MCP authentication-blocked tools/retirement | Blocked views never delegate execution; retired owner cannot reopen | None found in repaired branch | No immediate-retirement claim made | Async retirement leaves an old healthy handle window until its writer queues | Gate covers blocked view, not every previously captured healthy handle schedule |
| Runtime cancellation/borrowed owners | CE primary, cleanup suppressed, turn slot finally released | None found in repaired branch | None newly established | Pending cleanup remains per-tool, not transactional | Double-fault regression injects CE, not parent cancellation during disk write |
| In-memory RPC shutdown | Actual transport phase flag/handler; both endpoints cancelled before joins | None conclusively demonstrated | B2-1: phase-at-report-time may hide delayed live closed-send reporting | Class+phase filter has no failure-origin discriminator | Concurrent teardown suite and direct handler policy test are distinct |
| SearchIndex copied input | Real list snapshot before platform construction | None found in repaired branch | None newly established | No deep clone of arbitrary `T` promised | JVM/Native/JS behavior tests not run in this review |

### 1. Backend original owner, startup and remaining B1

- FQCNs: `io.github.stream29.kodex.rpc.server.BackendSessionHost`,
  `BackendSessionManagement`, `BackendServices` and top-level
  `withBackendServices`; actual repository is
  `io.github.stream29.kodex.agentsession.filesystem.FileSystemKodexSessionRepository`.
- Real chain: `io.github.stream29.kodex.cli.app.withKodexApplication`
  [Application.kt:71–88][application] → `withBackendServices`
  [BackendServices.kt:54–79][services] → `withBackendSessionHost` with
  `suspend CoroutineScope.() -> KodexRootSessionRepository`
  [BackendSessionHost.kt:44–61][hostfactory].
  The factory receiver is the original host supervisor owner; the result stored
  at [102–110][hostacquire] is that **same original repository**, not a Provider,
  LazyRepository, DTO proxy or per-tab resource graph.
- Get/settings/auth/models/MCP global paths [BackendGlobalRpc.kt:26–52][global]
  use global state/account/OAuth or `host.inBackend`, not `repository()`.
  Frontend construction borrows RPC proxies. Initial NewSession is a draft.
  Neither auth startup nor settings/global initialization acquires `sessions/`.
  First catalog/create/keepAlive path does:
  GlobalRpc → management [33–50,148–165][management] / host [119–133][activation]
  → `repository()` → filesystem factory [469–484][fsrepo].
  Explicit catalog refresh is a real operation, even when it returns empty.
- Acquisition mutex serializes successful first publication. It checks operation,
  owner and repository liveness before setting `acquiredRepository`; waiter
  cancellation does not cancel the accepted owner async [166–169][accepted].
  Host teardown cancels/joins the original owner and completes deactivation in
  NonCancellable [65–75][hostcleanup]. Untouched shutdown does not invoke factory.
  The liveness checks are not one atomic transaction with parent cancellation;
  a cancellation immediately afterwards still cascades to the same child.
- Failure guards: live reads use `SessionNotActive` without acquisition;
  explicit management checks persistence before `SessionNotFound`; cancelled
  acquisition propagates CE, ordinary factory failure propagates its exception.
  Existing original Session binding/TTL/runningTurn ownership is retained.
- **B1-1 — medium-severity Job lifetime leak, statically confirmed.**
  In the real factory [478–484][fsrepo], Kotlin evaluates the named argument
  `scope = supervisorChildScope()` **before**
  `initialEntries = sessionDirectories(...)`. The latter can suspend/throw:
  filesystem `list`/metadata errors, invalid numeric entry names, or cancellation
  [311–324][scan]. No local catch cancels/joins that already-created child.
  `supervisorChildScope()` really attaches an active `SupervisorJob` to its
  receiver [ChildScope.kt:41–48][childscope]; it is not an inert wrapper.
- Reachable failure: frontend catalog → BackendGlobalRpc → management →
  `host.repository()` → real factory → scan throws; RPC reports failure while the
  host remains live. `acquiredRepository` remains null, but the unreturned child
  remains attached to the stable host owner. Every failed retry can accumulate
  another child. No file descriptor/process/heartbeat leak is claimed; the
  surviving child is empty. Whole-host shutdown eventually releases all of them.
- Introducing diff: baseline `withBackendSessionHost` old lines 59–60 acquired
  eagerly inside its outer try; factory failure therefore immediately entered
  owner teardown. Snapshot line 60 only constructs host; new lines 102–110
  acquire through independent supervised `inBackend`. This exposes the existing
  filesystem factory's missing failed-construction cleanup to retryable requests.
- Tests inspected: [BackendStartupTest.kt:18–60][startuptest] checks real backend
  filesystem startup stays absent through settings/auth/models/MCP and then
  catalog/create/reopen; [HostTest:58–159][hosttests] gates eight first accesses,
  cancelled waiter, failed/cancelled attempts, owner-cancelled factory cleanup and
  already-cancelled result. The retry test throws **before creating a child**.
  [HostTest:581–595][hostfailuretest] creates a child then fails, but lets the
  whole host exit. Neither refutes B1-1.
- Smallest meaningful change: complete the fallible scan **before** allocating
  the repository child, or locally cancel/join that exact child if construction
  fails. Do not add a lazy repository wrapper or second scope manager.
  Missing deterministic gate: inject a scan failure in the **real filesystem
  factory**, catch it while the same host survives, compare active receiver-owner
  children before/after failure and retry, then verify normal host close.

### 2. Backend auth primary error, generation and authentic sources

- FQCN: `io.github.stream29.kodex.cli.auth.BackendFileSystemAuthStore`.
  Real login chain: GlobalRpc start/complete → `BackendOAuth.start` bound
  `OAuthTarget.OpenAi(source)` → `beginLogin(source)` →
  prepared exchange → `commitLogin(source,generation,tokens)`
  [BackendOAuth.kt:79–121][oauth].
  Remove cancels matching attempts then calls credentials.remove [158–166][remove].
  Maintenance is backend-owned for **both** files, not renderer polling.
- `writes` mutex covers begin/commit/remove/reload/refresh. Per-source generation
  increments on begin/remove; commit validates its exact generation and current
  owner/operation before saving [AuthStore:87–116][authgeneration].
  Refresh holds that same write lock through HTTP, rereads the authentic file
  before persistence, and abandons an observed externally changed token snapshot
  [129–165][authrefresh]. This is not cross-process CAS or protection against an
  independent process changing the file immediately after the reread.
- Sources/codecs remain unchanged: Codex uses `CodexCliStorage(codexHome)` and
  `auth.json`; Kodex uses `AuthYaml`/`KodexAuthFile` and private `auth.yml`
  [184–221,244–246][authread]. Selection is not rewritten by login/remove.
  Refresh preserves Codex metadata, does not clone credentials across sources.
  Missing credentials do not create files. Read CE propagates; decode/IO/other
  read failure maps to the original unavailable state.
- Repair/diff at old 225–229 → snapshot [225–240][authcleanup]:
  capture primary from private temp write/atomicMove; **inside** NonCancellable,
  use `runCatching(delete).exceptionOrNull()` so cleanup is returned as a value.
  With primary, attach distinct cleanup as suppressed and rethrow original;
  without primary, throw cleanup. This does not suppress failed persistence or
  falsely publish new credentials. Atomic move may already have committed when
  later cleanup fails; operation failure is not rollback evidence.
- Tests: [AuthTest:40–73][authtests] checks both sources, injected IOException/CE
  plus cleanup IOException and successful move plus failed cleanup.
  [93–115][authgentest] checks stale generation rejection/source refresh;
  [196–265][authracetest] gates external Codex replacement, refresh/remove,
  failed persistence, and actual caller cancellation with successful temp cleanup.
  Borrowed login client is asserted not closed [306–331][authfixture].
- U: double-fault tests inject CE without cancelling the Job; the real cancelled
  persistence test has no cleanup fault. No new run is claimed. Smallest ablation
  of `runCatching` **inside** NonCancellable restores cleanup-over-primary failure;
  no new cleanup framework or authentication provider is needed.

### 3. MCP admission, original routes, and spec/composition boundary

- FQCNs: `io.github.stream29.kodex.mcp.contract.toModelToolName`,
  `requireUniqueMcpModelNames`; `io.github.stream29.kodex.mcp.impl`
  `validateMcpConfigurationUpdate`, `McpManagerImpl`, `McpServiceImpl`,
  `McpClientOwner`, `McpToolImpl`.
  Inspected all MCP contract/stdio/HTTP spec files and composition production
  sources; original manager/service/client/tool remain the actual command,
  connection/catalog and dispatch owners.
- Diff: the identical raw→model character projection moved from private
  McpTool.kt old 109–122 to [McpModelToolNames.kt:3–14][names].
  Added collision guard groups that projection without changing persisted raw
  strings. Disabled servers also participate. Server keys must be globally
  unique after projection; full paginated tool names must be unique per server.
  Duplicate identical SDK tool names are rejected, not silently deduplicated.
- Real persistent CAS chain: GlobalRpc.compareAndSetSettings [28–29][global] →
  global state [111–115][globalcas] → manager command boundary →
  actual file CAS [63–77][filecas] →
  `validateMcpConfigurationUpdate` [16–17][validation] **before write**.
  Stale/equal CAS skips validation and preserves existing no-op semantics.
  Manager add/edit/import also call the same guard **inside** their atomic
  transforms [123,154–155,450][manager]. No second dispatcher or hash routing.
- Initial persisted server collisions fail synchronously at service factory
  [329–333][serviceadmission], before MCP resource/lifecycle child creation or
  stdio process start. This does not mean auth/settings backend children have
  never existed; enclosing backend failure cleanup still owns them.
- Tool collisions cannot be known before connecting/listing the server:
  `openConnection` validates the complete paginated catalog, closes the candidate
  SDK connection and reports `Failed(ToolCatalog)` before catalog publication
  [282–298,323–355][catalogread]. Refresh rejects ambiguity **while holding its
  original reader**, changes health to ToolCatalog failure and retains previous
  immutable tool spec [187–219][refresh]. No invalid remote catalog is exposed.
- Raw wire is preserved: namespace `mcp__<original projection>`, model tool
  projected name [McpTool.kt:39–49][mcptool]; remote SDK call still uses
  `tool.name`, not model name [68–77][rawcall]. Stable completion retains original
  pending callId/itemId/name/namespace/arguments. MCP secret/configuration
  serialization declarations and RPC signatures did not change in this repair.
- Tests: [RouteAdmissionTest:45–127][routetests] uses real SDK Client with offline
  typed transport plus actual State/tool runtime routing; rejects two server
  projections, rejects tool catalogs and preserves a working local route,
  raw SDK call name, and old catalog through bad refresh/reconnect.
  [McpConfigurationAdmissionTest:17–49][admissiontests] checks backend CAS and
  unchanged persisted ambiguous file/startup failure; first test calls real
  server implementation directly, **not encoded RPC round-trip**.
- D: [new guard KDoc:16–22][nameguard] uses “Throws” but lacks `@throws
  IllegalArgumentException` on the operation as required by spec checklist.
  Smallest documentation improvement is that tag, not an exception wrapper.
  Pure admission/name operations are meaningful shared logic; no empty
  Service/Controller/VM is warranted. Ablation: remove guard and actual original
  tool router again receives ambiguous routes; restoring `distinctBy(rawName)`
  would also hide identical duplicates instead of admitting a valid catalog.

### 4. MCP exact SDK identity, lock order and late false publication

- New `McpClientCatalog.sourceClient` [37–40][catalogidentity] is typed
  `io.modelcontextprotocol.kotlin.sdk.client.Client?`. `Client.readCatalog`
  sets it to **this actual SDK instance** [339–344][sdkidentity].
  Null belongs to the empty, connecting logical view and is never publishable.
  This is neither a second ViewModel nor a synthetic generation counter.
- Actual service publication [293–301][publication]:
  `transitionMutex` checks the **same** `McpClientOwner` is still registered →
  `owner.publishIfCurrent` takes connection reader →
  compare `activeClient === catalog.sourceClient` and non-Closed →
  atomically replace original `clients` map generation.
- Lock order inspected: service transition → connection reader on publication;
  reconnect uses reconnectMutex → connection writer for close/open/read, then
  **releases writer before entering publication** [138–175][reconnect].
  Refresh also releases its catalog-read reader before service publication.
  Reconcile/invalidate/release drop transition lock before closing retired owners.
  No reverse connection-lock→transition-lock nesting was found in these paths.
  SDK close callbacks schedule owner writer work rather than taking service lock.
- Failure scenario addressed: refresh reads old SDK under reader, releases it,
  reconnect replaces SDK and publishes new tools, old refresh then publishes.
  The new identity guard returns false; refresh [214–219][refresh] ignores false
  without closing anything. **It does not close the new SDK client.**
- The successful reconnect branch does close owner on false [167–170][reconnect],
  but another reconnect on that same owner cannot overtake it:
  reconnectMutex is held through its NonCancellable publication.
  Service-owner replacement yields a different owner and closes only this retired
  instance; Closed state prevents old reconnect handles reopening it [141,249][retire].
  A transport-close callback can invalidate the same attempt; closing that dead
  attempt is not closing a newer same-owner reconnect.
- Test [McpOwnerPublicationGateTest:32–69][identitytest] gates stale refresh
  publication around a genuine SDK reconnect, verifying new route remains.
  It uses actual owner admission but not service transitionMutex/reconciliation.
  Retained IO suite covers actual service reload/refresh/reconnect and token refresh;
  no execution or deterministic whole-service race pass claimed here.
- U/D: two simultaneous refreshes on the **same** SDK are not totally ordered
  by the new identity check; minimum cross-SDK identity safety is the reviewed
  repair, not a new refresh sequencing protocol. Smallest meaningful ablation:
  remove either owner-registration check or SDK identity check and a retired/
  old-connection catalog can overwrite the current published generation.

### 5. MCP blocked tools and exact retired-owner behavior

- FQCN: `io.github.stream29.kodex.mcp.impl.McpAuthenticationBlockedClient`.
  Reconcile [147–168][blockedpublication] retains previous tool specifications
  in a logical enabled blocked client, before awaiting retired owner close.
  New wrapper [86–98][blocked] delegates metadata/spec only:
  `handle` constructs original `StableMcpToolEvent` with authentication error
  and **does not call the retained tool or SDK**. reconnect is a no-op.
- This is not a second connection owner. Original healthy tools go through
  McpToolImpl → McpClientImpl.call → `McpClientOwner.call`:
  original connection reader validates Healthy and exact active SDK
  [222–245][ownercall]. Retirement's writer sets Closed, clears activeClient,
  closes SDK and releases its server HTTP view exactly once [247–257][retire].
  Later old client/tool/reconnect handles cannot reopen that owner.
- Boundary caveat: already captured **healthy** handles can remain healthy
  between blocked-map publication and queuing the retirement writer.
  The blocked projection fix does not retroactively rewrite those handles or
  cancel admitted calls. Once writer queues, ReadWriteMutex blocks new readers;
  once it retires, original handles are unavailable. No stronger immediate/
  joint settings+connection atomicity is claimed.
- Test [OwnerPublicationGateTest:71–119][blockedtest] holds publication-to-
  retirement gate, retains actual SDK tool specs, invokes **blocked** tool and
  asserts no remote call before owner writer can retire. It does not invoke
  previously captured healthy handles in that window.
  Actual [IO service test:292–350][blockedio] checks blocked logical client,
  retained spec identity and former owner Closed after transition.
- Smallest ablation: directly returning old executable tools from blocked client
  (the baseline behavior) lets that projection invoke a still-healthy old owner.
  Keep the metadata-only delegation and original owner's Closed guard; no new
  resource wrapper, timer, reconnection policy or router is needed.

### 6. Runtime CE/suppressed and runningSlot release

- FQCN: `io.github.stream29.kodex.agentruntime.impl.AgentRuntimeImpl`
  (private actual runtime), implementing
  `io.github.stream29.kodex.agentruntime.contract.AgentRuntime`.
  Real command: BackendAgentRuntimeRpc.resume [104–120][rpcruntime] →
  host.inSession operation [147–163][sessionoperation] → composed runtime →
  compaction/steer/tool → original State.
  Stop cancels that actual `runningTurn` Job, not Session or RPC observer.
- Diff at runtime old 154–155 → snapshot [153–161][runtimecancel]:
  `runCatching(delegate.clearPending()).exceptionOrNull()` runs **inside**
  NonCancellable; attach distinct cleanup failure to original CE and rethrow CE.
  `withRunningTurn` finally releases exact caller slot [176–185][runtimeslot],
  even if clearPending fails. Failed pending durable completion remains pending,
  not fake successful cleanup. Manual compaction does not acquire resume's
  pending interruption semantics.
- The factory still returns `AgentRuntime` backed by original delegated State.
  `UnifiedExecClient` is the same composition-created process client exposed
  through its spec. Backend-owned operations survive frontend waiter cancellation;
  closing Session cascades its operations. Shared model/MCP/auth resources remain
  borrowed dependencies, not closed by turn/observer cleanup.
- Spec now explicitly documents CE/suppressed behavior and occupied-slot
  exception on `AgentRuntime.resume`; no Job/wire or RPC signatures changed.
- Test [AgentRuntimeRunningTurnTest:70–107][runtimefaulttest] publishes real
  pending State, injects CE after request, faults actual cleanup completion port,
  then asserts exact CE, exact suppressed cleanup, null slot and still-pending
  call. This tests composition/error algebra, not real parent cancellation
  while filesystem cleanup is failing.
  Existing backend RPC Stop/cancelled-waiter tests [163–205][rpcstoptest] and
  compaction cancellation/slot gates are separate ownership evidence.
- Ablation: returning to unguarded clearPending in NonCancellable substitutes
  IO for CE; deleting finally slot release leaves stale runningTurn. Keep both
  small original-boundary operations; do not add Runtime cleanup owner/framework.

### 7. In-memory RPC shutdown and B2 reporting boundary

- FQCNs: `io.github.stream29.kodex.rpc.inmemory.ChannelRpcTransport` and
  top-level `withInMemoryRpc`; actual entry is Application.kt:76.
  Production server/client receive this exact transport coroutineContext,
  including original child `Job`, new handler and captured previous handler.
  This is not just a test-side swallowing handler.
  This proves the transport-scope callback path, not that every internal kRPC
  coroutine root inherits that handler; upstream client internal roots differ.
- Diff: [ChannelRpcTransport.kt:19–32][transport] adds volatile shutdown flag,
  inherited error handler and exact `shuttingDown && failure is
  ClosedSendChannelException` predicate; [WithInMemoryRpc.kt:83–89][shutdown]
  sets both flags before intentional endpoint cancellation.
  While flag false, closed-send is forwarded; after flag true, unrelated errors,
  wrapped/cause-only errors and `CompletionHandlerException` are **not** filtered.
  Existing handler receives original context/throwable; absent it, failure is
  thrown into normal coroutine unhandled-error reporting. No broad catch/cause
  walk/success substitution is introduced.
- Actual shutdown protocol: stop monitors; mark both transports; cancel **both**
  endpoint Jobs and both channels before joining either; NonCancellable joins
  within existing 10-second budget [80–102][shutdown].
  Ordinary call/Flow failure and cancelled subscription do not close connection.
  Unexpected live endpoint termination fails connection through active monitors
  [54–67][monitors]. Outer primary failure remains caught/rethrown; cleanup
  timeout is suppressed onto it. Backend owner failures occur outside this
  transport handler and are not converted into successful backend results.
- Upstream pin inspection supports the distinction between public bounded
  channels and kRPC's private send queue:
  `kotlinx.rpc.krpc.internal.KrpcConnector.sendChannel` is unbounded and
  `drainSendQueueAndClose` closes it; `KrpcSendHandler.sendMessage` can throw direct
  ClosedSendChannelException. Endpoint cancellation sends/drains over the
  transport scope. See immutable-commit source locations in provenance above.
  The local filter does not modify kRPC shutdown, add shutdown RPC or give
  transport ownership of arbitrary registered backend resources.
- **B2-1 — suspected reporting race, not confirmed swallowed production fault.**
  The predicate tests the phase **when the handler is called**, not when failure
  originated. A live private-queue ClosedSend failure reported after another
  thread begins finalization would satisfy the filter; it cannot distinguish
  that from an expected late cancellation reply. Ordinary non-ClosedSend errors
  and existing captured primary failures remain visible. No evidence here proves
  such a delayed live error bypasses parent Job failure/caught primary paths.
- Evidence distinction:
  [InMemoryRpcTest:95–120][parallelrpctest] uses actual kRPC with 32 simultaneous
  collectors over 32 rounds on Dispatchers.Default, cancels parent and requires
  no completion-handler errors. [228–259][rpcownertest] checks operation release
  and original CE. This is materially stronger teardown evidence than a direct
  handler call, but neither was run by this reviewer.
  [ChannelRpcTransportTest:25–49][handlerpolicytest] directly invokes the actual
  handler for live ClosedSend, teardown ClosedSend and unrelated teardown error.
  It proves predicate policy/inherited-handler preservation only—not real
  asynchronous error origin, nor the race window.
- Smallest missing experiment: gate a **real live kRPC closed-send failure**
  before its error-handler delivery; start independent intentional shutdown;
  release report and assert live fault/owner failure remains observable.
  Also gate parent cancellation before local finally sets flags. Do not mark
  the class+phase predicate as a proven failure-origin discriminator.
  Ablation: remove the handler and the actual parallel teardown gate should
  expose expected library noise again; that does not justify swallowing live
  faults or expanding the exception predicate.

### 8. SearchIndex snapshot and approved policy limits

- FQCNs: `io.github.stream29.kodex.utils.searchindex.createSearchIndex`,
  `SearchIndex`, platform `LuceneSearchIndex` / `FallbackSearchIndex`.
  Caller chain: root runtime mcpTools/toolSearch [100–123][toolstates] →
  `io.github.stream29.kodex.tool.toolsearch.ToolSearchEngine`
  [11–18][searchcaller] → factory → platform actual.
- Diff: [SearchIndexFactory.kt:6–7][searchfactory] changes direct input forwarding
  to `documents.toList()` **before** platform construction. This detaches mutable
  list structure/order/membership; `SearchDocument` has immutable value/text
  properties. It is not `asList`/unmodifiable-view wrapping.
  JS fallback searches retained detached documents; Lucene stores IDs and maps
  hit IDs back to that same detached list, so list replacement/clear cannot
  silently substitute another hit value.
- Test [SearchIndexTest.kt:21–30][searchtest] mutates element then clears caller
  list and requires original result each time. No test run or deep cloning of
  arbitrary mutable `T` claimed. Existing SearchIndex API/actual types unchanged.
  Ablation: remove `toList()` and JS membership plus Lucene hit-ID→value mapping
  again depend on caller mutation. No new factory interface or API wrapper needed.
- User policy preserved: remote-compaction default is still **20 retries** in
  [OpenAiClientConfig.kt:13][compactpolicy], consumed by remote compaction's
  streaming retry budget [OpenAiClient.kt:263][compactcaller]; this is not inferred
  from the separate Agent response retry policy also using 20.
  Request-user-input instructions say omit retained `autoResolutionMs`; imagegen
  explicitly advertises no history-image selection, while original fields/
  serialization remain. Relevant narrowed descriptions:
  [RequestUserInputTools.kt:8–12][inputpolicy],
  [ImageGenerationTools.kt:24–35][imagepolicy].
  No new timer, auto-answer, history-image algorithm or changed wire is accepted
  by this review.

### Handoff limits

- B1-1 blocks lifecycle closure despite supported repaired paths elsewhere.
  B2-1 remains a targeted experiment, not an additional confirmed blocker.
- Re-review the smallest B1 repair increment against this snapshot; no repair was
  made here. Acceptance needs central execution evidence for that new gate and
  existing ownership/teardown regressions; this report does not wait for or claim
  those builds.
- Snapshot retained for the coordinating reviewers; this reviewer did not create
  or own it and must not delete their shared review input.

### URI references

[manifest]: file:///tmp/kodex-closure-review-6b7129fa-20261007/REVIEW_SOURCE_MANIFEST.json#L1
[host]: file:///tmp/kodex-closure-review-6b7129fa-20261007/rpc/impl/server/src/commonMain/kotlin/io/github/stream29/kodex/rpc/server/BackendSessionHost.kt#L85
[hostfactory]: file:///tmp/kodex-closure-review-6b7129fa-20261007/rpc/impl/server/src/commonMain/kotlin/io/github/stream29/kodex/rpc/server/BackendSessionHost.kt#L44
[hostacquire]: file:///tmp/kodex-closure-review-6b7129fa-20261007/rpc/impl/server/src/commonMain/kotlin/io/github/stream29/kodex/rpc/server/BackendSessionHost.kt#L102
[accepted]: file:///tmp/kodex-closure-review-6b7129fa-20261007/rpc/impl/server/src/commonMain/kotlin/io/github/stream29/kodex/rpc/server/BackendSessionHost.kt#L166
[hostcleanup]: file:///tmp/kodex-closure-review-6b7129fa-20261007/rpc/impl/server/src/commonMain/kotlin/io/github/stream29/kodex/rpc/server/BackendSessionHost.kt#L65
[activation]: file:///tmp/kodex-closure-review-6b7129fa-20261007/rpc/impl/server/src/commonMain/kotlin/io/github/stream29/kodex/rpc/server/BackendSessionHost.kt#L119
[sessionoperation]: file:///tmp/kodex-closure-review-6b7129fa-20261007/rpc/impl/server/src/commonMain/kotlin/io/github/stream29/kodex/rpc/server/BackendSessionHost.kt#L147
[services]: file:///tmp/kodex-closure-review-6b7129fa-20261007/rpc/impl/server/src/commonMain/kotlin/io/github/stream29/kodex/rpc/server/BackendServices.kt#L54
[management]: file:///tmp/kodex-closure-review-6b7129fa-20261007/rpc/impl/server/src/commonMain/kotlin/io/github/stream29/kodex/rpc/server/BackendSessionManagement.kt#L33
[global]: file:///tmp/kodex-closure-review-6b7129fa-20261007/rpc/impl/server/src/commonMain/kotlin/io/github/stream29/kodex/rpc/server/BackendGlobalRpc.kt#L26
[application]: file:///tmp/kodex-closure-review-6b7129fa-20261007/app/impl/application/src/commonMain/kotlin/io/github/stream29/kodex/cli/app/Application.kt#L71
[fsrepo]: file:///tmp/kodex-closure-review-6b7129fa-20261007/agent-session/impl/filesystem/src/commonMain/kotlin/io/github/stream29/kodex/agentsession/filesystem/FileSystemKodexSessionRepository.kt#L469
[scan]: file:///tmp/kodex-closure-review-6b7129fa-20261007/agent-session/impl/filesystem/src/commonMain/kotlin/io/github/stream29/kodex/agentsession/filesystem/FileSystemKodexSessionRepository.kt#L311
[childscope]: file:///tmp/kodex-closure-review-6b7129fa-20261007/utils/coroutines/spec/src/commonMain/kotlin/io/github/stream29/kodex/utils/coroutines/ChildScope.kt#L41
[startuptest]: file:///tmp/kodex-closure-review-6b7129fa-20261007/rpc/impl/server/src/commonTest/kotlin/io/github/stream29/kodex/rpc/server/BackendStartupTest.kt#L18
[hosttests]: file:///tmp/kodex-closure-review-6b7129fa-20261007/rpc/impl/server/src/commonTest/kotlin/io/github/stream29/kodex/rpc/server/BackendSessionHostTest.kt#L58
[hostfailuretest]: file:///tmp/kodex-closure-review-6b7129fa-20261007/rpc/impl/server/src/commonTest/kotlin/io/github/stream29/kodex/rpc/server/BackendSessionHostTest.kt#L581
[auth]: file:///tmp/kodex-closure-review-6b7129fa-20261007/rpc/impl/server/src/commonMain/kotlin/io/github/stream29/kodex/cli/auth/BackendFileSystemAuthStore.kt#L44
[oauth]: file:///tmp/kodex-closure-review-6b7129fa-20261007/rpc/impl/server/src/commonMain/kotlin/io/github/stream29/kodex/rpc/server/BackendOAuth.kt#L79
[remove]: file:///tmp/kodex-closure-review-6b7129fa-20261007/rpc/impl/server/src/commonMain/kotlin/io/github/stream29/kodex/rpc/server/BackendOAuth.kt#L158
[authgeneration]: file:///tmp/kodex-closure-review-6b7129fa-20261007/rpc/impl/server/src/commonMain/kotlin/io/github/stream29/kodex/cli/auth/BackendFileSystemAuthStore.kt#L87
[authrefresh]: file:///tmp/kodex-closure-review-6b7129fa-20261007/rpc/impl/server/src/commonMain/kotlin/io/github/stream29/kodex/cli/auth/BackendFileSystemAuthStore.kt#L129
[authread]: file:///tmp/kodex-closure-review-6b7129fa-20261007/rpc/impl/server/src/commonMain/kotlin/io/github/stream29/kodex/cli/auth/BackendFileSystemAuthStore.kt#L184
[authcleanup]: file:///tmp/kodex-closure-review-6b7129fa-20261007/rpc/impl/server/src/commonMain/kotlin/io/github/stream29/kodex/cli/auth/BackendFileSystemAuthStore.kt#L225
[authtests]: file:///tmp/kodex-closure-review-6b7129fa-20261007/rpc/impl/server/src/commonTest/kotlin/io/github/stream29/kodex/cli/auth/BackendFileSystemAuthStoreTest.kt#L40
[authgentest]: file:///tmp/kodex-closure-review-6b7129fa-20261007/rpc/impl/server/src/commonTest/kotlin/io/github/stream29/kodex/cli/auth/BackendFileSystemAuthStoreTest.kt#L93
[authracetest]: file:///tmp/kodex-closure-review-6b7129fa-20261007/rpc/impl/server/src/commonTest/kotlin/io/github/stream29/kodex/cli/auth/BackendFileSystemAuthStoreTest.kt#L196
[authfixture]: file:///tmp/kodex-closure-review-6b7129fa-20261007/rpc/impl/server/src/commonTest/kotlin/io/github/stream29/kodex/cli/auth/BackendFileSystemAuthStoreTest.kt#L306
[names]: file:///tmp/kodex-closure-review-6b7129fa-20261007/mcp/spec/contract/src/commonMain/kotlin/io/github/stream29/kodex/mcp/contract/McpModelToolNames.kt#L3
[nameguard]: file:///tmp/kodex-closure-review-6b7129fa-20261007/mcp/spec/contract/src/commonMain/kotlin/io/github/stream29/kodex/mcp/contract/McpModelToolNames.kt#L16
[globalcas]: file:///tmp/kodex-closure-review-6b7129fa-20261007/rpc/impl/server/src/commonMain/kotlin/io/github/stream29/kodex/rpc/server/BackendGlobalState.kt#L111
[filecas]: file:///tmp/kodex-closure-review-6b7129fa-20261007/app/settings/impl/filesystem/src/commonMain/kotlin/io/github/stream29/kodex/cli/settings/FileSystemBackendSettingsStore.kt#L63
[validation]: file:///tmp/kodex-closure-review-6b7129fa-20261007/mcp/impl/composition/src/commonMain/kotlin/io/github/stream29/kodex/mcp/impl/McpConfigurationValidation.kt#L12
[manager]: file:///tmp/kodex-closure-review-6b7129fa-20261007/mcp/impl/composition/src/commonMain/kotlin/io/github/stream29/kodex/mcp/impl/McpManagerImpl.kt#L114
[mcpowner]: file:///tmp/kodex-closure-review-6b7129fa-20261007/mcp/impl/composition/src/commonMain/kotlin/io/github/stream29/kodex/mcp/impl/McpClientImpl.kt#L104
[mcpservice]: file:///tmp/kodex-closure-review-6b7129fa-20261007/mcp/impl/composition/src/commonMain/kotlin/io/github/stream29/kodex/mcp/impl/McpServiceImpl.kt#L42
[serviceadmission]: file:///tmp/kodex-closure-review-6b7129fa-20261007/mcp/impl/composition/src/commonMain/kotlin/io/github/stream29/kodex/mcp/impl/McpServiceImpl.kt#L329
[catalogread]: file:///tmp/kodex-closure-review-6b7129fa-20261007/mcp/impl/composition/src/commonMain/kotlin/io/github/stream29/kodex/mcp/impl/McpClientImpl.kt#L282
[sdkidentity]: file:///tmp/kodex-closure-review-6b7129fa-20261007/mcp/impl/composition/src/commonMain/kotlin/io/github/stream29/kodex/mcp/impl/McpClientImpl.kt#L339
[refresh]: file:///tmp/kodex-closure-review-6b7129fa-20261007/mcp/impl/composition/src/commonMain/kotlin/io/github/stream29/kodex/mcp/impl/McpClientImpl.kt#L181
[mcptool]: file:///tmp/kodex-closure-review-6b7129fa-20261007/mcp/impl/composition/src/commonMain/kotlin/io/github/stream29/kodex/mcp/impl/McpTool.kt#L39
[rawcall]: file:///tmp/kodex-closure-review-6b7129fa-20261007/mcp/impl/composition/src/commonMain/kotlin/io/github/stream29/kodex/mcp/impl/McpTool.kt#L68
[routetests]: file:///tmp/kodex-closure-review-6b7129fa-20261007/mcp/impl/composition/src/commonTest/kotlin/io/github/stream29/kodex/mcp/impl/McpRouteAdmissionTest.kt#L45
[admissiontests]: file:///tmp/kodex-closure-review-6b7129fa-20261007/rpc/impl/server/src/commonTest/kotlin/io/github/stream29/kodex/rpc/server/McpConfigurationAdmissionTest.kt#L17
[catalogidentity]: file:///tmp/kodex-closure-review-6b7129fa-20261007/mcp/impl/composition/src/commonMain/kotlin/io/github/stream29/kodex/mcp/impl/McpClientImpl.kt#L37
[publication]: file:///tmp/kodex-closure-review-6b7129fa-20261007/mcp/impl/composition/src/commonMain/kotlin/io/github/stream29/kodex/mcp/impl/McpServiceImpl.kt#L293
[reconnect]: file:///tmp/kodex-closure-review-6b7129fa-20261007/mcp/impl/composition/src/commonMain/kotlin/io/github/stream29/kodex/mcp/impl/McpClientImpl.kt#L138
[retire]: file:///tmp/kodex-closure-review-6b7129fa-20261007/mcp/impl/composition/src/commonMain/kotlin/io/github/stream29/kodex/mcp/impl/McpClientImpl.kt#L247
[identitytest]: file:///tmp/kodex-closure-review-6b7129fa-20261007/mcp/impl/composition/src/jvmTest/kotlin/io/github/stream29/kodex/mcp/impl/McpOwnerPublicationGateTest.kt#L32
[blockedpublication]: file:///tmp/kodex-closure-review-6b7129fa-20261007/mcp/impl/composition/src/commonMain/kotlin/io/github/stream29/kodex/mcp/impl/McpServiceImpl.kt#L147
[blocked]: file:///tmp/kodex-closure-review-6b7129fa-20261007/mcp/impl/composition/src/commonMain/kotlin/io/github/stream29/kodex/mcp/impl/McpClientImpl.kt#L86
[ownercall]: file:///tmp/kodex-closure-review-6b7129fa-20261007/mcp/impl/composition/src/commonMain/kotlin/io/github/stream29/kodex/mcp/impl/McpClientImpl.kt#L222
[blockedtest]: file:///tmp/kodex-closure-review-6b7129fa-20261007/mcp/impl/composition/src/jvmTest/kotlin/io/github/stream29/kodex/mcp/impl/McpOwnerPublicationGateTest.kt#L71
[blockedio]: file:///tmp/kodex-closure-review-6b7129fa-20261007/mcp/impl/composition/src/jvmTest/kotlin/io/github/stream29/kodex/mcp/impl/McpServiceImplIoTest.kt#L292
[runtime]: file:///tmp/kodex-closure-review-6b7129fa-20261007/agent-runtime/impl/composition/src/commonMain/kotlin/io/github/stream29/kodex/agentruntime/impl/KodexAgentRuntimeComposition.kt#L137
[runtimecancel]: file:///tmp/kodex-closure-review-6b7129fa-20261007/agent-runtime/impl/composition/src/commonMain/kotlin/io/github/stream29/kodex/agentruntime/impl/KodexAgentRuntimeComposition.kt#L153
[runtimeslot]: file:///tmp/kodex-closure-review-6b7129fa-20261007/agent-runtime/impl/composition/src/commonMain/kotlin/io/github/stream29/kodex/agentruntime/impl/KodexAgentRuntimeComposition.kt#L176
[rpcruntime]: file:///tmp/kodex-closure-review-6b7129fa-20261007/rpc/impl/server/src/commonMain/kotlin/io/github/stream29/kodex/rpc/server/BackendAgentRuntimeRpc.kt#L104
[runtimefaulttest]: file:///tmp/kodex-closure-review-6b7129fa-20261007/agent-session/impl/in-memory/src/commonTest/kotlin/io/github/stream29/kodex/agentsession/inmemory/AgentRuntimeRunningTurnTest.kt#L70
[rpcstoptest]: file:///tmp/kodex-closure-review-6b7129fa-20261007/rpc/impl/server/src/commonTest/kotlin/io/github/stream29/kodex/rpc/server/BackendRuntimeRpcTest.kt#L163
[transport]: file:///tmp/kodex-closure-review-6b7129fa-20261007/rpc/impl/in-memory/src/commonMain/kotlin/io/github/stream29/kodex/rpc/inmemory/ChannelRpcTransport.kt#L19
[connection]: file:///tmp/kodex-closure-review-6b7129fa-20261007/rpc/impl/in-memory/src/commonMain/kotlin/io/github/stream29/kodex/rpc/inmemory/WithInMemoryRpc.kt#L37
[shutdown]: file:///tmp/kodex-closure-review-6b7129fa-20261007/rpc/impl/in-memory/src/commonMain/kotlin/io/github/stream29/kodex/rpc/inmemory/WithInMemoryRpc.kt#L80
[monitors]: file:///tmp/kodex-closure-review-6b7129fa-20261007/rpc/impl/in-memory/src/commonMain/kotlin/io/github/stream29/kodex/rpc/inmemory/WithInMemoryRpc.kt#L54
[parallelrpctest]: file:///tmp/kodex-closure-review-6b7129fa-20261007/rpc/impl/in-memory/src/commonTest/kotlin/io/github/stream29/kodex/rpc/inmemory/InMemoryRpcTest.kt#L95
[rpcownertest]: file:///tmp/kodex-closure-review-6b7129fa-20261007/rpc/impl/in-memory/src/commonTest/kotlin/io/github/stream29/kodex/rpc/inmemory/InMemoryRpcTest.kt#L228
[handlerpolicytest]: file:///tmp/kodex-closure-review-6b7129fa-20261007/rpc/impl/in-memory/src/commonTest/kotlin/io/github/stream29/kodex/rpc/inmemory/ChannelRpcTransportTest.kt#L25
[searchfactory]: file:///tmp/kodex-closure-review-6b7129fa-20261007/utils/search-index/impl/src/commonMain/kotlin/io/github/stream29/kodex/utils/searchindex/SearchIndexFactory.kt#L6
[searchcaller]: file:///tmp/kodex-closure-review-6b7129fa-20261007/tool/impl/tool-search/src/commonMain/kotlin/io/github/stream29/kodex/tool/toolsearch/ToolSearchEngine.kt#L11
[toolstates]: file:///tmp/kodex-closure-review-6b7129fa-20261007/agent-runtime/impl/composition/src/commonMain/kotlin/io/github/stream29/kodex/agentruntime/impl/KodexAgentTools.kt#L100
[searchtest]: file:///tmp/kodex-closure-review-6b7129fa-20261007/utils/search-index/impl/src/commonTest/kotlin/io/github/stream29/kodex/utils/searchindex/SearchIndexTest.kt#L21
[compactpolicy]: file:///tmp/kodex-closure-review-6b7129fa-20261007/openai/impl/client/src/commonMain/kotlin/io/github/stream29/kodex/openai/client/OpenAiClientConfig.kt#L13
[compactcaller]: file:///tmp/kodex-closure-review-6b7129fa-20261007/openai/impl/client/src/commonMain/kotlin/io/github/stream29/kodex/openai/client/OpenAiClient.kt#L263
[inputpolicy]: file:///tmp/kodex-closure-review-6b7129fa-20261007/tool/impl/request-user-input/src/commonMain/kotlin/io/github/stream29/kodex/tool/requestuserinput/RequestUserInputTools.kt#L8
[imagepolicy]: file:///tmp/kodex-closure-review-6b7129fa-20261007/tool/impl/image-generation/src/commonMain/kotlin/io/github/stream29/kodex/tool/imagegeneration/ImageGenerationTools.kt#L24
[krpcconnector]: file:///home/stream/ACodeSpace/fork/kotlinx-rpc/krpc/krpc-core/src/commonMain/kotlin/kotlinx/rpc/krpc/internal/KrpcConnector.kt#L76
[krpcsender]: file:///home/stream/ACodeSpace/fork/kotlinx-rpc/krpc/krpc-core/src/commonMain/kotlin/kotlinx/rpc/krpc/internal/KrpcSendHandler.kt#L73
[krpcendpoint]: file:///home/stream/ACodeSpace/fork/kotlinx-rpc/krpc/krpc-core/src/commonMain/kotlin/kotlinx/rpc/krpc/internal/KrpcEndpoint.kt#L19

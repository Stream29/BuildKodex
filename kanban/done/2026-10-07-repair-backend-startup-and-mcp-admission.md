# Task Tree

- `Trace startup acquisition and actual MCP executable route admission`()
- `Repair lazy repository acquisition and reject ambiguous routes`()
- `Execute central regression and gated SDK fixtures`()
- `Publish startup and route test evidence for independent review`()
- `Receive authorization to seal and commit the scoped repair`()

# Details

- 协调者：RPC server 135、MCP composition 27 / contract 4 项 JVM 通过；
  实际初始扫描失败 child 泄漏补修，隔离 CLI 仅编辑/Settings 不创建 sessions。
  独立复审支持启动/身份准入的静态修复；RPC 错误报告阶段疑点仍为 B2，
  未宣称被该 admission 修复覆盖。
  [最终统一证据](2026-10-07-plan-spec-impl-closure-and-consistency.md#final-scoped-closure-evidence)。

- [修复主任务](2026-10-07-plan-spec-impl-closure-and-consistency.md)，用户已授权
  修复后重新审查；基线 `6b7129fa`。不变更 provider retry、raw name/wire 格式。
- 本线独占 RPC server `BackendSessionHost.kt`、`BackendSessionManagement.kt`、
  `BackendServices.kt` 及准确 startup/host 专属测试，`mcp/{spec,impl/composition}`
  和 RPC MCP 配置 validation 源/测试。backend auth/Runtime/主构建为协调者，
  前端 Settings source/queue 为另一线，其他 Source 不动。
- BackendSessionHost 保留唯一 factory/repository 身份，真实初始化延迟到首个
  create/open/catalog 访问，以既有 owner 与一个准确 acquisition Mutex 防重复。
  不造 LazyRepository wrapper、Provider、第二 registry。实际消费者直接改成
  suspend acquisition；失败/cancel 不发布半成品，原 child 关闭语义保留。
- 初始 App/Backend Settings/model/auth 读取不得创建 sessions；实际首次 catalog/
  create/open 初始化一次。测试在隔离 Home 检查路径、factory 调用数、
  并发首次访问、取消/失败、未访问直接关闭。
- MCP raw names 与原 noncolliding executable route 不变；只把明确歧义在实际
  admission/catalog 处拒绝并显示现有 failure 状态，不让重复路由在执行别的
  pending tool 时才炸 dispatcher。共享唯一实际 normalization helper，不复制算法。
  两 server `a-b/a_b`、一个 catalog `x-y/x_y` 的离线 SDK/路由 fixture 必测。
  旧已保存歧义配置不静默改写；读取/重连失败应明确，不挂死 owner。
- refresh/reconnect stale publish 与 auth-blocked retained owner 两个疑点先
  controlled fixture 复现；若确认，只改原 owner 的 generation/admission，
  不造 retry manager、router 或额外 mutable tool registry。
- 不运行 Gradle、真实服务/网络/IDE、提交/推送；写本文件 Integration handoff，
  列所有直接 consumer spelling、实际 API 与测试、中央验证待执行。

## Integration handoff

- **READY for central validation — source review only, not tests green.**
- `withBackendSessionHost(createRepository: suspend CoroutineScope.() -> KodexRootSessionRepository,
  block: suspend CoroutineScope.(BackendSessionHost) -> R)` retains its spelling.
  Its internal `timeSource` overload also retains its spelling. Neither invokes the
  factory until actual repository access.
- Exact internal constructor: `BackendSessionHost(owner: CoroutineScope,
  createRepository: suspend CoroutineScope.() -> KodexRootSessionRepository,
  origin: TimeMark, backgroundFailure: CompletableDeferred<Throwable>)`.
- Actual consumer API: `public suspend fun repository(): KodexRootSessionRepository`;
  replace `host.repository.create/open/getEntry/listEntries/readCreatedAt/createFork/delete/entries`
  with `host.repository().create/open/getEntry/listEntries/readCreatedAt/createFork/delete/entries`.
  Host receiver helper spelling becomes `repository().create()`. No facade or
  changed repository static type. `requireEntry` becomes suspend.
- KDoc contract: factory executes under the original backend owner, only on first
  actual acquisition; one suspend mutex publishes one fully acquired live repository.
  Cancelled RPC waiters do not cancel accepted host work; factory failure/owner
  cancellation does not publish a repository. Untouched host shutdown invokes no
  factory and allocates no repository resources. Host shutdown still awaits children.
- This lane retargets `BackendSessionManagement` and host/management/file tests.
  No current production repository references in Runtime/Timeline source were found.
  **Main owns `BackendRuntimeRpcTest`** (two `host.repository` accesses) and all
  auth/title tests/build adapters; please use the same `repository()` spelling.
  Existing Host test factory-failure case must now actually call `repository()`;
  factory-captured `lateinit repository` must not be read before that call.
- Central Gradle/build dependency changes belong to main. No new actual projects.
  Exact fixture dependencies and suite list are recorded below.
  No builds, network servers, IDE or resource operations run.
- **Central test dependencies:** `:mcp-impl-composition`
  `commonTest` needs `:agent-runtime-impl-decorator-tool`, `:agent-state-impl-state`,
  `:agent-state-test`, `:agent-storage-impl-in-memory`, `:openai-impl-client-test`,
  `:tool-impl-plan` for the offline actual SDK owner → model projection → actual ToolRuntime route
  fixture. SDK typed-response transport uses the already-present SDK client/core;
  no SDK server/testing dependency or socket/process fixture is needed.
- Shared normalization now lives in `mcp/spec/contract/McpModelToolNames.kt`:
  `String.toModelToolName()` and `requireUniqueMcpModelNames(names, kind)`.
  Original implementation algorithm is moved, not duplicated. Full-value RPC
  validation and manager add/edit/import reject server collisions before persistence;
  the service factory rejects old persisted collisions synchronously before children.
- B2 refresh/reconnect and blocked-owner timing remain gated on controlled fixture
  execution by central validation; no generation/retirement business changes yet.

### Ready implementation and validation boundary

- Exact source anchors: [host acquisition](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/server/src/commonMain/kotlin/io/github/stream29/kodex/rpc/server/BackendSessionHost.kt#L102),
  [shared normalization/admission](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/spec/contract/src/commonMain/kotlin/io/github/stream29/kodex/mcp/contract/McpModelToolNames.kt#L4),
  [SDK/routing fixture](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/impl/composition/src/commonTest/kotlin/io/github/stream29/kodex/mcp/impl/McpRouteAdmissionTest.kt#L44),
  [gated owner experiments](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/impl/composition/src/jvmTest/kotlin/io/github/stream29/kodex/mcp/impl/McpOwnerPublicationGateTest.kt#L31).

- The host stores the factory and uses one acquisition Mutex under its original
  owner. The factory runs inside accepted backend work, not the RPC waiter's Job.
  Failed/cancelled/dead results are not cached; successful concurrent accesses
  return the actual same `KodexRootSessionRepository`. Host lifecycle/TTL semantics
  and original child cleanup remain unchanged. No LazyRepository/Provider facade,
  per-tab root or alternate repository registry was introduced.
- Production consumer retargeting was limited to getter spellings and the necessary
  `suspend requireEntry`. Runtime/Timeline production had no repository getter
  accesses and were not edited. Main has independently retargeted its Runtime tests;
  all six requested MCP fixture dependencies are now present in main-owned Gradle.
- Complete settings proposals and manager add/edit/import reject normalized server
  collisions before publishing. Startup rejects persisted ambiguous names before
  service resource/child creation. SDK initial/reconnect catalogs are checked after
  complete pagination, before publication; ambiguous refresh keeps the previous
  catalog and sets existing `Failed(ToolCatalog)`. Transport/OAuth retry, connection
  ownership, raw call names, serialized configuration and noncolliding routes stay
  unchanged. There is no new failure enum, hash route or tool registry.
- Ordinary suites to run centrally:
  - `backendSessionHostTest`: untouched close; concurrent first acquisition + lost
    waiter; failed/cancelled/dead factory results; owner cancellation with real
    repository child cleanup; original activation/TTL/cleanup cases.
  - `backendSessionHostFileTest`: isolated absent sessions path and one concurrent
    file factory/catalog acquisition; original expiry/lease/reopen case.
  - `backendStartupTest`: complete real services, fake external OpenAI clients,
    settings/auth/models/MCP reads before absent sessions; first catalog/create;
    persisted reopening through keepAlive.
  - `mcpModelToolNamesTest`, `mcpManagerImplTest`,
    `mcpConfigurationAdmissionTest`: pure normalization, add/rename/import rejection,
    full-value RPC CAS rejection, and persisted old-file startup rejection with
    byte preservation and no sessions acquisition.
  - `mcpRouteAdmissionTest`: actual SDK Client with in-memory typed-response
    transport, two `a-b/a_b` echo servers, one `x-y/x_y` catalog, preserved raw
    SDK names, rejected refresh/reconnect retention, actual State + ToolRuntime
    routing and a healthy local `update_plan` despite rejected MCP routes.
- **Gated B2 experiments:** set `KODEX_MCP_OWNER_EXPERIMENTS=1` for
  `mcpOwnerPublicationGateTest` (JVM). These intended-behavior assertions are not
  enabled by default and have not been executed here:
  - R1 uses old refresh publication held after its reader releases, then successful
    reconnect/new publication, then releases the old callback. It asserts the new
    route survives; central reproduction must precede a generation fix.
  - R2 uses the exact blocked client construction and retained SDK tools, holding
    the publication-to-retirement boundary **before** writer acquisition.
    ReadWriteMutex blocks later readers once the writer queues; that is not the
    suspected gap. This owner/blocked-view fixture does not independently prove
    scheduling of the full service collector. It asserts no remote call is admitted.
  - No B2 fix or behavioral pass is claimed. Confirmed failures require only original
    owner generation/admission/close changes, not another router or auth tool facade.
- Static checks performed: scoped and whole-worktree `git diff --check` had no
  output; one actual normalization declaration; no residual property-form
  `host.repository.` consumers found; branch/HEAD remain `refactor/spec`/`6b7129fa`.
  These are source checks, not compiler/test execution.
- Actual source/test inventory in the three existing owned projects: MCP contract
  **9 Kotlin files** (baseline 7), MCP composition **13** (baseline 11), RPC server
  **38** (baseline 36); **60 actual files**, six added by this lane (one pure
  production helper and five test files). **New projects: 0.**
  This lane's change set is 20 Kotlin source/test files plus this handoff; shared
  Gradle/auth/Runtime changes shown by Git belong to main, not this lane.
- No Gradle, server/network, IDE, resource operation, commit, push or switch was
  executed. No temporary files or acquired resources remain. Original independent
  reports were not edited. Central compilation/regression and independent acceptance
  remain pending; leave this task executable rather than moving it to done.

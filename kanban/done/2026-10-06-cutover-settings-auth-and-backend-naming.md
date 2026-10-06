# Task Tree

- `Verify live backend authentication and title owners`()
- `Map real code and shared helpers into the existing backend`()
- `Specify legacy API removal and retained frontend ports`()
- `Define RPC lifecycle, title races, and frontend subscription tests`()
- `Receive explicit implementation start`()
- `Cut over live authentication and backend naming`()
- `Publish exact API and test handoff`()
- `Receive centralized build and acceptance evidence`()
- `Receive user acceptance`()

# Details

## Scope and ownership

- 用户验收通过，随主任务归档 done；内层本地提交 `206266c8`，未推送。
- 父任务：[行为重新归属](2026-10-06-reassign-shared-application-behavior.md)；
  依赖：[配置与持久化](2026-10-06-cutover-settings-models-and-persistence.md)。
- 同一工作单元负责认证、命名以及所有 `rpc/impl/server` 文件，避免多线争写
  BackendServices/BackendAgentRuntimeRpc。基线 `e89c5094`。
- Settings/Authentication/Login 现有真实组件保留；改变的是后台实现归位与旧路径删除，
  不把凭据、维护 Job、模型请求搬到 UI spec 或 ViewModel。

## Authentication cutover

| 原代码 | 唯一目标/动作 |
| --- | --- |
| BackendFileSystemAuthStore.kt | 原 rpc/impl/server，继续实现真实 OpenAiAuthStore，保留 scope factory |
| PreparedOpenAiLogin.kt | 同 server，继续由 BackendOAuth 调用；不持有前端 listener |
| KodexAuthFile.kt、SubscriptionJwt.kt | 同 server 内部文件，不进入 RPC wire 或 UI spec |
| ActiveSubscriptionAuth、AuthLoadResult、toAuthLoadResult、subscriptionRefreshAt | 从旧 FileSystemKodexAuthStore.kt 拆出到 server 的实际 auth helper 文件 |
| pkceCodeChallenge 与其 SHA256/helpers/constants | 从 LocalKodexLogin.kt 拆出到 server；PreparedOpenAiLogin 真正使用，保留原算法 |
| KodexAuthStore、InMemoryKodexAuthStore | 删除旧 API；不新建与 RPC 并存的前端 auth service |
| KodexAuthLoginAttempt | 原名直接迁入现有 OpenAI Login spec，由协调者接管；真实 RpcOAuth/UI 使用，不能删除 |
| FileSystemKodexAuthStoreImpl/工厂、LocalKodexLoginAttempt/CallbackServer | 删除旧组合入口和旧后端 listener |

- 工厂 FQCN 与生产方法保持原名称；internal helper 直接移动，不保留旧文件转发。
- 仅删除前必须拆出的 helper，而不是把含有真实依赖的旧文件整体删除。
- 执行接线复核补充：旧 auth contract 文件同时包含真实前端 KodexAuthLoginAttempt。
  原声明已由协调者归位 OpenAI Login spec，后端线删除旧文件中的声明，
  不在 server 或新 auth spec 再复制；当前接口名和依赖注入保持不变。
- 两来源都由后端加载、续期、回存、login/remove；source selection 属于 BackendSettings，
  login 成功不隐式切换 source，不同步复制凭据到另一来源。
- BackendOAuth 的真实 attempt、10 分钟 waiting expiry、捕获 source、PKCE、
  generation/cancel/late refresh 防护保持原行为，不复制第二组状态。
- 前端 RpcOAuth 的 loopback listener 和浏览器 effect 留前端；
  删除的是旧 LocalKodexLogin 后端 listener，不是删除现行前端 OAuth。
- 借用 login client 的关闭责任不改变。正常维护不依赖 Settings Popup，
  root backend shutdown 仍先结束依赖工作，再关 client。
- Server 移入 private文件/YAML/schema 需要 serialization plugin 与原 Kaml/schema 依赖；
  不增加通用 provider/credentials adapter、工厂 service locator 或空认证 spec。
- 外部可观察能力继续来自真实 GlobalRpc/OpenAiAuthStore；implementation 内的文件机制
  不是新增远程服务。现有 SettingsAuthenticationState 摘要与 wire 保持不变。

## Legacy tests and probe

- BackendFileSystemAuthStoreTest、SubscriptionJwtTest 迁入 server commonTest。
- FileSystemKodexAuthStoreTest 的旧 Codex 只读/旧流程测试删除；
  仍有效的账号解析、刷新 fallback/窗口、失败保留和 PKCE 断言迁到真实后端 suite。
- LocalKodexLoginAttemptJvmTest 旧 listener lifecycle 用例删除；其真实 PKCE vector/交换参数
  断言迁入 backend helper/OAuth 测试，现行 frontend listener 由 RpcOAuth 测试覆盖。
- `integration-test/.../OpenAiResponsesProbeJvmTest.kt` 有旧 loader/in-memory 的真实源码引用：
  协调者将 loader 改为当前 BackendFileSystemAuthStore，用 `StateFlow<KodexAuthSource>`，
  snapshot 后如果需要固定 auth，只在 probe 内创建实现实际 OpenAiAuthStore 的测试对象。
  不为 probe 保留生产 InMemoryKodexAuthStore 或创建新测试支持项目。
- Probe 仅编译，不调用真实供应商、不读真实凭据；实际运行必须另获授权。

## Backend naming cutover

- `SessionTitleGenerator` 和 `SessionTitleGenerationResult` 原名/FQCN迁入
  `rpc/spec/contract`，它们是 withBackendServices 的真实本地 dependency，
  不是新 `@Rpc` 服务、wire DTO、查询状态或 ViewModel。
- 完整 generateTitle 的输入/结果/throws/cancellation KDoc 移入该真实 port。
  原 public factory/createTitleGenerator test seam 保持其能力，不加 forwarding wrapper。
- DefaultSessionTitleModel 移入纯 `session-title-settings/spec`；
  前端 RpcConfigurationSettingsDependencies 不再引用 `app-shared-session-title` 实现。
- AgentTitleGeneration、OpenAiSessionTitleGenerator、SessionTitleSchema 与所有
  sanitize/output/Unicode helpers 直接迁入 server implementation，原 FQCN保持，
  两套真实测试跟随实现；不设新 manager/project。
- 后端仍在成功 append 首条有效文本后启动；初始化空 Session 时不请求模型。
  runtime scope 拥有任务，创建 RPC/前端页面结束不能关闭它。
- 保留 consumed/attempt/Job、明确改名、settings CAS、replaceHistory 的一致性规则，
  不用删“额外状态”之名移除真实 late-result 防护。
- 测试用 `useSettingsCas=false` 的旧机制不能继续成为平行生产策略：
  生产当前使用 true。优先让迁移后的测试也走真实 CAS 模式；
  若 false 路径无其他生产消费者，删除该选项与旧分支，保留 true 的真实算法。
  不因删除参数改变已接受请求、命名竞争或错误处理。
- 前端仅通过现有设置 timeline/目录 metadata 观察名称，不发起 generate 或持有 title Job。

## Shared host integration

- 本线独占 BackendServices 的更新：
  - auth/title 工厂指向唯一归位实现。
  - settings open 指向新实际文件实现。
  - KodexAgentDependencies 删除最后 hooks 参数/NoOp import，与 Hook 线交接的五字段签名一致。
  - 原 client、account、OAuth、MCP、Session 创建/关闭顺序保留。
- `rpc/impl/server` 移除 app-shared-auth/settings/session-title 依赖，改为新 persistence impl、
  pure配置与既有 RPC specs；原 app-contract-settings 只为摘要 re-export 的边可删除，
  直接依赖真实 Authentication/Account Usage/Session Catalog spec，
  后端不依赖 Settings 父 ViewModel、组件 view/viewmodel。
- Application/RpcSettings/配置组件接线由协调者负责，BackendServices 不经前端 VM 装配认证。

## Required tests

- `:rpc-impl-server:jvmTest`：迁入 auth/JWT/title suites、BackendOAuth、
  BackendAccountState、BackendRuntimeRpc、BackendServices、SessionManagement 全部覆盖。
- 认证：两来源 read/refresh/write、刷新失败旧状态不丢、remove 后 late login/refresh
  不复活、exact attempt、10 分钟 waiting expiry、无 UI 仍维护、borrowed client close。
- 命名：文本 eligibility、disabled/已有名称不请求、一次性、fallback default、
  failed/incomplete/empty response、Unicode/normalization、explicit rename/settings CAS/history
  replace 竞争、取消及 late result、辅助失败不改变 append 成功。
- 必须新增/保留真实 RPC 链验证：
  1. fake delayed title generator 仅替代模型供应商，真实 Session/runtime/storage 正常运行。
  2. 前端发首条消息后关闭借用 view/Settings，不关 backend owner。
  3. 释放 generator，另一存活的前端 observation 从原 SettingsTimelineRpc 观察新名称；
     目录 metadata 与 tab binding 的名称投影亦核对。
  4. explicit rename/历史替换抢先发生时，迟到 title 不覆盖新值。
- 现有 frontend Settings→Login→return、Authentication/Title Settings renderer、
  RpcOAuth/current binding 回归由协调者集中执行，不只用 isolated VM 证明接线。
- `:integration-test:compileTestKotlinJvm` 与 CLI 主源码编译验证没有旧 loader 引用；
  不运行 live probe、真实浏览器或真实 Home。
- Linux x64 主源码 compile 按 convention 实际可用目标记录；实际运行与编译分开。
  当前 `kodex.kmp-cli` 无 JS target，不执行或宣称 JS 验证。

## Readiness

- 当前真实 helper、公开 factory 与测试消费者已定位，工程就绪，无新增语义阻塞。
- 用户明确启动，已进入 executable。若生成结果的发布路径缺测试证据，补真实 existing RPC observation 用例；
  不默认增加通知、title 状态服务或前端生成行为。
- 拆原文件后检查 single definition 和 private secret 日志；剩余 source/helper 不得复制到
  新旧两处。所有 server 构建和真实测试由协调者集中运行。

## Integration handoff — backend lane

- Status: implementation ready; centralized build/runtime validation pending.
  No Gradle/IDE/resource operations, real Home/credentials, commits or pushes by this lane.
- Auth factory remains
  `io.github.stream29.kodex.cli.auth.CoroutineScope.BackendFileSystemAuthStore(dataDirectory, codexHome, selectedSource: StateFlow<KodexAuthSource>, loginClient, fileSystem = SystemCoroutineFileSystem, clock = Clock.System)`.
  Its real `BackendFileSystemAuthStore : OpenAiAuthStore` and `PreparedOpenAiLogin` move to
  `rpc/impl/server/src/commonMain/kotlin/io/github/stream29/kodex/cli/auth/`.
  `prepareOpenAiLogin(client, redirectUri)` remains the listener-free BackendOAuth entry.
- Same auth directory receives `KodexAuthFile.kt`, `SubscriptionJwt.kt`,
  `SubscriptionAuth.kt` (actual ActiveSubscriptionAuth, AuthLoadResult,
  both toAuthLoadResult overloads, subscriptionRefreshAt and refresh constants),
  and `Pkce.kt` (actual PKCE/SHA256 helpers and constants extracted before retiring LocalKodexLogin).
- Local generator port/result keep FQCN
  `io.github.stream29.kodex.cli.sessiontitle.SessionTitleGenerator` /
  `SessionTitleGenerationResult`, moving to
  `rpc/spec/contract/src/commonMain/kotlin/io/github/stream29/kodex/cli/sessiontitle/SessionTitleGenerator.kt`.
  No service registration or wire change. Settings lane owns the copied
  DefaultSessionTitleModel declaration; backend lane deletes its old source.
- AgentTitleGeneration, OpenAiSessionTitleGenerator, SessionTitleSchema and both suites move to
  the corresponding `cli/sessiontitle` directories in server. All callers use real settings CAS;
  the unused false branch and constructor flag are removed.
- `withBackendServices` keeps its existing `createTitleGenerator: (Client) -> SessionTitleGenerator`
  seam and raw scope lifetime. BackendServices consumes the Settings lane's unchanged
  `io.github.stream29.kodex.cli.settings.openBackendSettings` factory/store interface.
  KodexAgentDependencies construction becomes five fields:
  client, modelCatalog, contextSettings, shellSettings, mcpService; no hooks/NoOp.
- Main-owned server Gradle changes needed: serialization plugin; Kaml,
  kotlinx.schema.json, kotlinx.serialization.json, ktor.utils,
  utils-kotlinx-io-coroutines-impl, openai-spec-json-codec,
  openai-spec-client/models and agent-state-contract as direct dependencies where used;
  replace shared settings/auth/title edges with app-settings-impl-filesystem,
  persistence/pure component specs and existing RPC specs. No ktor server/listener dependency
  is needed by migrated auth. Retain current in-memory RPC/client test dependencies.
  **Add server commonTest `implementation(project(":app-viewmodel-rpc"))`** for
  `BackendTitleRpcTest` to close actual RpcSessionViews/RpcGlobalEditor/RpcGlobalSettings,
  rather than inventing a fake frontend binding. This is test-only: app-viewmodel-rpc
  main does not depend on server, so neither main graph nor test compilation has a cycle.
  `utils-logging-impl` is not needed by the extracted helpers: the removed legacy
  loader was its only auth consumer. Main can omit that provisional dependency.
- Central test matrix: moved BackendFileSystemAuthStore/JWT/title suites, valid legacy
  auth parsing/refresh/PKCE assertions, existing OAuth/Account/Services/Runtime/Session tests;
  add delayed-title real RPC tests with actual filesystem Session/runtime/storage,
  borrowed frontend observation close while backend lives, surviving SettingsTimelineRpc
  name observation and catalog query, and rename/history replacement beating late output.
  External clients/generator alone are mocked. Central builds and runtime acceptance remain pending.

### Implementation evidence

- Auth store and JWT implementation, title provider and schema are byte-identical moves.
  PKCE/SHA256 helpers, round constants, subscription helper types/functions and refresh
  constants compare identically with their original baseline sections.
  Only the listener-free/private-format comments were clarified.
- AgentTitleGeneration preserves the existing true-CAS loop; repository-wide caller inventory
  found no other production strategy. Its old flag and unconditional generated-title write
  are gone; tests use the ordinary runtime's actual compareAndSetSettings.
- `BackendTitleRpcTest` contains three real eight-service RPC cases: accepted append then
  actual borrowed view/Settings popup/global observations close; surviving tab settings and
  direct SettingsTimelineRpc/catalog naming update; settings CAS rename and destructive
  history replacement defeating late generation, with replacement eligibility restored.
- `SubscriptionAuthTest` retains valid RFC7636, JWT account/plan/email, supported mode,
  partial refresh and 5-minute/8-day scheduling assertions. Moved backend store tests add
  both-source refresh-error preservation, Kodex persistence failure/re-read and maintenance
  with no frontend owner. Moved title tests add image/blank eligibility, disabled/nondefault
  consumption and rejected/throwing one-shot attempts. BackendRuntimeRpcTest adds owner-close cancellation.
- Existing OAuth, Account, Global state, Notifications, Runtime, Session management/host and
  Services suites remain. The shared Services fixture only gains an optional real-port
  generator injection for the new RPC suite; original default behavior remains disabled.
- Static checks: no old assigned auth/title source/test files remain; no old backend
  store/listener/NoOp/useSettingsCas symbols remain; generator/result/default each have
  one declaration; tracked and new-file whitespace checks are clean.
  Frozen migration main source is unchanged. This lane did not edit migration files;
  the shared migration test harness has another lane's explicit legacy-loader expectation update.
- No test execution is claimed. Main owns existing Gradle/host/probe integration, platform
  compilation, complete JVM suites and frontend Settings→Login→return/UI acceptance.

### Central validation

- RPC server 118 项 JVM 测试通过，含迁入的真实 auth/JWT/PKCE/title suites 及三条
  BackendTitleRpc 用例；原 OAuth/Account/Runtime/Session/Services suite 保留。
- 借用前端关闭后 backend 命名仍发布至 tab、SettingsTimelineRpc 和 catalog；
  显式改名/历史替换击败迟到结果，原一次性 eligibility/CAS 保持。
- server Linux x64 主源码、CLI JVM、Integration JVM 测试源码编译通过；
  live probe 只改为实际 backend loader，未访问真实 Home 或网络。
- Settings/Auth/Login/Title renderer 与宿主回归、完整计数和未验证平台见
  [统一验收证据](2026-10-06-reassign-shared-application-behavior.md#central-acceptance-evidence)。
  用户已验收，内层本地提交 `206266c8`，未推送。

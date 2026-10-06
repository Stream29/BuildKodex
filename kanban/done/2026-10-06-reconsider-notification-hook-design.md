# Task Tree

- `Trace historical Hook semantics and current production assembly`()
- `Separate notification configuration, execution, and backend event production`()
- `Apply established RPC notification-only semantics`()
- `Record complete legacy code retirement scope`()
- `Review real Hook model and execution ownership`()
- `Specify real contracts, project mapping, and old-path deletion`()
- `Define original-UI and full-host acceptance tests`()
- `Return confirmed design to parent planning`()
- `Receive explicit implementation start`()
- `Cut over notification models and remove complete control chain`()
- `Publish exact API and test handoff`()
- `Receive centralized build and acceptance evidence`()
- `Receive user acceptance`()

# Details

## Scope and status

- 父任务：[按行为重新归属 app/shared](2026-10-06-reassign-shared-application-behavior.md)。
- 用户要求重新检查多次演变的 Hook 设计；允许提出调整，不等于批准新的事件、
  控制行为、文件格式或执行策略。
- 状态：Done，用户已验收，本地代码提交 `206266c8`，未推送；
  RPC 设计早已确定只执行通知型 Hook、禁止旧控制语义。
  用户本轮进一步确认完整退役保留的旧控制代码。工程映射、删除闭包与验证计划已补齐，
  开始按已定方案实施，不重开已有产品决策。基线为 `e89c5094`。
- 当前规范：[notification-only Hooks](../../checklist/rpc-settings.md#frontend-notification-only)。
  未确认的新方案只记在本计划，不覆盖已确认 checklist。

## History — three distinct boundaries

| 阶段 | 实际设计 | 本轮意义 |
| --- | --- | --- |
| 旧 Stop 交互 | Hook 能取消待输入工具或注入续跑片段 | 不是当前通知语义，不能通过搬目录复活 |
| Kodex 原生控制 Hook | 名称映射、pre/post tool、prompt、Stop、compaction 等协议，控制/观察行为不同 | 基线有旧代码但未装配，本轮完整退役 |
| RPC notification-only | 后端发布 Stop 事实，前端命令本地提醒，不影响 Agent | 当前生产行为与文件规则的基线 |

- 历史依据：[Request User Input Stop](../done/2026-08-10-run-stop-hook-for-request-user-input.md)、
  [原生 Hook 重构](../done/2026-08-17-decouple-hooks-from-codex.md)、
  [RPC 通知实施](../done/2026-09-27-rpc-notification-hooks.md)。
- [Hook checklist](../../checklist/hooks.md) 已收缩为当前通知型能力及其真实归属；
  不保留旧控制协议作为当前规范。

## Production trace

1. BackendAgentRuntimeRpc 在真实运行停止/失败处调用 BackendNotifications。
2. BackendNotifications 发布四类 Notification.Stop，GlobalRpc 提供唯一全局流。
3. Application 在 frontend scope 启动一个 collectNotificationHooks。
4. 每条通知读取 CliFrontendSettings.hooks 快照，匹配 types，按列表顺序执行。
5. Hook Settings 拥有配置交互、准确 dialog token、最新草稿字段更新；
   实际持久化属于原应用级写入队列，不属于 Popup 短期 scope。

- [后端事件源](../../Kodex/rpc/impl/server/src/commonMain/kotlin/io/github/stream29/kodex/rpc/server/BackendNotifications.kt)；
  [生产装配](../../Kodex/rpc/impl/server/src/commonMain/kotlin/io/github/stream29/kodex/rpc/server/BackendServices.kt)；
  [前端订阅](../../Kodex/app/viewmodel/application/src/commonMain/kotlin/io/github/stream29/kodex/cli/app/Application.kt)；
  [执行器](../../Kodex/hook/impl/notification/src/commonMain/kotlin/io/github/stream29/kodex/cli/notification/NotificationHooks.kt)；
  [现有 Hook Settings spec](../../Kodex/app/component/hook-settings/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/hooksettings/HookSettingsViewModel.kt)。

## Existing semantics — unchanged unless separately approved

- 后端不执行用户 Hook、不等待控制结果；本轮删除 NoOp 与控制端口，不用空实现维持旧装配。
- 通知四类：AssistantMessage、RequestUserInput、SuggestSubagent、UnhandledError。
  一次等待按请求类型聚合，同类保留全部调用；取消与辅助命名失败不当作运行错误通知。
- 流 replay=0、缓冲 64、DROP_OLDEST、tryEmit；不重放、不持久化、不做确认。
- 前端 name/types/command 有序列表；每通知固定快照，多类型匹配同一 Hook 只执行一次。
- 普通 collect，命令串行，10 秒，stdin 是原 Notification JSON，固定前端启动 cwd。
- Hook 输出不控制后端，不直接写 TUI；失败诊断后继续。前端取消向上传播并清理当前进程，
  不为排空积压延长退出；关闭标签或 Settings 不停止全局消费者。
- 原 settings migration 不转换或自动执行旧控制脚本，新通知 Hook 默认空。
- Settings 编辑的 Accepted 是队列准入，不是持久化成功；保留最新 exact-token
  字段更新和关闭后已接受写入排空，不恢复旧 whole-draft 覆盖。

## Ownership

- **Hook Settings ViewModel/View**：配置列表、草稿、校验、菜单/对话框，
  不订阅运行通知、不启动 Shell、不镜像执行器状态。
- **前端 Hook 执行实现**：读取配置快照、匹配通知、串行命令、资源清理；
  普通函数已能表达行为，不要求新增 Executor/Dispatcher/Manager 链。
- **前端宿主**：拥有唯一消费者 scope 与启动 cwd，关闭 UI 子组件不重新挂载执行器。
- **后端**：拥有真实停止事实的生产；Notification 不是另一份可写的 Agent 状态。
- **基础设施**：继续使用真实 ShellClient；不把 Shell discovery 或用户进程放进组件 spec。

### H1 — established semantics and confirmed code retirement

- [原 RPC 边界](../../checklist/rpc-settings.md#frontend-notification-only) 已禁止控制 Hook；
  当时保留历史实现/测试并使用 NoOp 装配，不代表当前仍支持控制脚本。
- 用户本轮通过结构化选择确认“完整退役旧控制链”，将保留代码也纳入清理。
  本批规划完整退役旧控制 Hook，而不是仅删除
  hook/impl 后保留 NoOp 与无作用参数支撑旧骨架。
- 删除面需包括旧 HookManager、控制 request/result、运行时 callback/helper、
  Turn/Tool/Compaction 使用点、Session 创建/组合签名与旧设置运行模型。
- **已知闭包**：Runtime composition 仍依赖 TurnHook decorator，Tool/Compact 使用旧
  Hook 端口，Session contract 公开相关参数，integration/test fixtures 亦有引用。
  “CLI 不使用真实脚本”不等于“可以直接删根目录”。
- 消融实验：移除旧注入与控制分支，用真实生产链和 Runtime/Session 测试证明行为；
  旧测试按产品退役处理，不复制同样的模型到新 spec 以让旧测试继续通过。
- 不再把旧控制能力作为独立支持的库功能保留，也不与前端 NotificationHook
  共用一个假统一协议。冻结 migration 的历史文件格式声明不是待复活运行接口。
- 精确闭包见 Engineering cutover；真实回归在实施后执行，当前不提前标记通过。

### H2 — concrete model and execution placement

- `hook/spec/notification` 持有原 NotificationHook/NotificationHookType，保留 FQCN、
  序列化值与校验；`hook/impl/notification` 持有原 collect/执行函数，配置 UI
  仍在 app/component/hook-settings。不再独立 app/shared/notification。
- 模型 spec 只依赖 serialization，不导入 Notification wire 类型或 Shell implementation。
  原 KDoc 中跨模块 Notification 类型链接改为文字说明，避免创建反向代码依赖。
- Notification 本身确实跨 RPC，作为事件 published language 可继续保留真实 wire 声明；
  不因为文件中有 Notification 字样就把所有类型都搬到前端。
- RPC models 的 CliFrontendSettings、Hook Settings spec 及真实执行器依赖新模型 spec。
  impl 依赖 rpc/models 完成事件匹配和 JSON 输入；调用仍是原唯一函数，不新增接口链。
- 模型 KDoc 声明配置快照、匹配次数与顺序等可观察语义；执行函数保留原资源/取消 KDoc。
  spec 的真实值模型已被持久化、编辑与执行使用，不是空标记或虚构执行 API。
- 依赖为 `rpc/models -> hook/spec/notification`，
  `hook/impl/notification -> rpc/models + hook/spec/notification + Shell impl`，无环。

### H3 — apply existing execution semantics, not a new decision gate

- notification-only、前端唯一消费者、无 replay、串行命令和 10 秒预算已经由
  RPC 设计确认，直接作为本轮验收基线，不再要求用户重新选择。
- 本轮只解决领域混放与旧代码退役。若用户另行提出产品能力变更，再单独评审：
  - 更多事件：何处生产、载荷及是否泄露内容；不是复活 pre/post 控制协议。
  - 串行改并行：执行顺序、并发限制、进程数量、取消与失败隔离。
  - 可配超时/重试：文件变化、重复副作用、退出清理和兼容性。
  - replay/可靠投递：身份、积压、重连与去重，不能用 StateFlow 最后一条假装解决。
  - 诊断 UI：若需要则明确可观察状态与存储范围，不预建通知中心或每进程 child VM。
- 上述不是本批待办或阻塞项；迁移不得暗中改变预算、等待规则或通知必达承诺。

## Hard-cutover and acceptance gates

- 产品语义沿用原 RPC 决策；以以下精确映射完成旧代码退役。
- 保留同一前端配置真源与同一执行入口；旧配置不会被复活，停止/错误不会双发。
- 真 RPC 生产→订阅→配置匹配→进程 fixture 全链，而非只测 fake executor。
- 测试无订阅/重连、慢消费者溢出、同类多个 pending、两种 pending 混合、
  失败/取消不误通知、一次快照、类型去重、顺序、失败继续。
- Settings 原 UI/键盘回归、同帧多字段编辑、迟到 token、关闭后 queue 排空。
- 进程 start 取得交接、stdin 挂起、非零退出、超时/退出等待、清理失败、
  frontend close、Settings/标签切换保持单订阅。
- 借助隔离命令与临时目录，不运行用户脚本、不修改真实 Home。
- 原重验遗留的取得交接/lease/平台运行风险分别记录；新测试未证明前不升级为通过。

## Engineering cutover

### Retain and move

- 移动原 NotificationHook.kt 到 `hook/spec/notification`；
  Notification.kt 与 BackendNotifications 保持原模型/生产 owner。
- 移动原 NotificationHooks.kt 与 NotificationHooksTest.kt 到 `hook/impl/notification`；
  公共入口 FQCN 与 Flow/settings/cwd 签名不变，仍使用真实 ShellClient。
- Hook Settings spec 添加新模型 spec 的直接依赖，不为编辑器增加执行端口。
- Application 仅改执行项目依赖与入口位置，不增加第二消费者；
  Settings/标签变化不能导致重新订阅。

### Delete full control chain

| 删除项 | 同步修改 | 必须保留的行为 |
| --- | --- | --- |
| `hook/spec/hooks`、`hook/impl/hooks` | 删除控制模型、HookManager/KodexHooks/NoOp、toolutils、旧命令协议测试 | 新通知模型/执行链不共用这些类型 |
| Runtime `spec/decorator/turn-hook` 与 `impl/decorator/turn-hook` | 删除 TurnHookRuntime/Impl/Projection 与旧控制测试、composition 调用层 | Agent resume、steer、运行互斥、日志、资源关闭及正常停止 |
| Tool runtime 的 ToolHooks 参数与 pre/post 调用 | `tool.handle` 直接执行，删除 Block 分支与 helper 依赖 | 固定/动态路由、搜索、invalid call、MCP unavailable、state-bound tool 不重复完成 |
| Compact runtime 的可选 CompactionHooks 参数与 request 转换 | 直接 underlying compact，保留自动/显式流程与异常 | retry 现有策略、compaction trigger/reason/phase、取消与失败日志 |
| KodexAgentDependencies.hooks 及 close 中取消 Hook Job | 更新所有生产/测试构造；后端线更新 BackendServices | 先停止 Session，MCP/catalog/client 的原关闭与 suppressed 语义 |
| 旧合并 settings 中 HookConfiguration/HookSettings | 配置线删除旧合并模型，冻结 migration 不动 | 当前 NotificationHook 列表、空默认、前端文件与 queue |

- `agent-runtime-impl-composition` 从 compact/tool/steer 组合真实层，不新增替代 TurnHook。
- Hook 相关 ErrorHooks/request/result 也随旧根删除，不留下不可调用公共协议。
- 旧 control-only integration fixtures/tests 删除或改为新通知行为；测试专用依赖不维持旧 API。
  正常工具、并发、pending、停止、compaction 的有效断言迁到保留测试，不删掉证据。
- 后端线独占 BackendServices；Hook 线交接 KodexAgentDependencies 五参数签名，
  不跨线编辑同一 server 文件。协调者集中改所有现有宿主 Gradle 与 registry。
- 全仓扫描包括 common/jvm/js/native/actual/test、build scripts 和代码生成输入；
  冻结 migration 中历史 Hook 字段允许存在但必须标明是历史文件格式。

### Concrete verification

- 保留目标：`:hook-spec-notification:compileKotlinJvm`、
  `:hook-impl-notification:jvmTest`、Hook Settings 两 impl 的 jvmTest。
- 消融目标：Runtime Tool/Compact/Steer/Composition、Session filesystem/in-memory、
  RPC server 的 jvmTest；删除 TurnHook 目标后不再把它们写入命令矩阵。
- 新增一条真实 in-memory RPC 订阅＋隔离本机命令 fixture，证明生产事件确实路由到
  当前前端 Hook，失败/超时不使 Agent 失败，不运行用户脚本。
- 同时验证 backend publish 不背压、无 replay 与缓冲溢出；实时 stop 不能双发。
- Settings 原交互与真实 Application scope 的单消费者测试继续保留；
  config snapshot、同帧编辑、queue close 与回调 target 不改变。
- 执行前保存原 UI fixtures；运行后对照文本、顺序与键盘操作，不用直接调用 VM
  替代真实 renderer 回调证明。

## Current checkpoint

- 完成三阶段设计对比与当前调用链核对。
- 已复核原 RPC 专门声明，纠正把既有语义重新列为决策的问题。
- Hook lane 实施完成并已交接；原通知语义、wire/文件声明和 retry policy 保持不变。
  两新项目承接真实模型/执行器，四旧项目 source/test 退役；旧 build/registry
  清理由协调者完成。当前挂起于集中构建与宿主验收，不等待其结果占用工作线。
- 静态检查已完成；尚无本工作线运行测试通过结论。

## Integration handoff

- Hook lane started on `e89c5094`, shared `refactor/spec`; no builds, IDE/resource
  operations, commits, switches, live commands or Home access in this lane.
- Exact retained public entry:
  `suspend fun collectNotificationHooks(notifications: Flow<Notification>,
  settings: StateFlow<CliFrontendSettings>, workingDirectory: Path): Unit`
  in `io.github.stream29.kodex.cli.notification`; host imports remain unchanged.
- Exact new constructor:
  `KodexAgentDependencies(client: OpenAiClient, modelCatalog: OpenAiModelCatalogStore,
  contextSettings: StateFlow<AgentContextSettings>, shellSettings: StateFlow<ShellSettings>,
  mcpService: McpService)`. No sixth/default Hook argument. Close order remains
  MCP → model catalog → client, attempting all and suppressing later failures.
- Exact runtime replacement:
  `KodexAgentState.compactionRuntime(modelCatalog: OpenAiModelCatalogStore, logger: KLogger)`
  → `.steerRuntime(logger) { pendingSteer.getAndUpdate { emptyList() } }`
  → `.toolRuntime(fixedTools: List<Tool>, dynamicTools: StateFlow<List<Tool>>,
  toolSearch: StateFlow<ToolSearchEngine>, logger: KLogger)`.
  No outer `turnHookRuntime`; existing running-turn slot, cancellation, pending/logging
  and resource ownership remain in the root runtime.
- Concrete public constructor is now
  `KodexAgentCompactionRuntimeImpl(delegate: KodexAgentState,
  modelCatalog: OpenAiModelCatalogStore, logger: KLogger)`; no optional Hook parameter.
- Removed public families: `KodexHooks`/NoOp, HookManager/settings/configuration,
  Hook context/protocol, TurnHooks/ToolHooks/CompactionHooks/ErrorHooks and their
  request/result types, toolutils helpers, TurnHookRuntime/Impl/Projection.
- Notification models retain `io.github.stream29.kodex.rpc.models` and serializer
  descriptors in `:hook-spec-notification`; spec depends only on serialization.
  Impl is `:hook-impl-notification`, using RPC models and real Shell impl;
  Hook Settings spec retarget is owned here.
- Fixture scope owned here: `agent-session/test/TestKodexAgentDependencies.kt`,
  filesystem `FileSystemSessionCleanupTest.kt`, `FileSystemKodexSessionRepositoryTest.kt`,
  in-memory `AgentRuntimeRunningTurnTest.kt`, `InMemoryKodexSessionRepositoryTest.kt`,
  integration `MinimalAgentConversationTest.kt`.
  Only the shared test dependency factory and Minimal conversation fixture needed
  direct argument/import removal; the other Session fixtures already use that factory
  and remain intact. RunningTurn fixture also gains the normal-composition regressions below.
- Coordinator import/argument-only cleanup candidates outside this lane:
  `app/test-support/rpc/RpcFrontendFixture.kt`; History VM tests
  `HistoryPagingReadinessTest.kt`, `AgentHistoryModelsTest.kt`, `HistoryItemReleaseTest.kt`;
  History mosaic tests `AgentHistoryAnchorTest.kt`, `AgentHistoryBoundedWindowTest.kt`.
  Backend lane owns all server fixtures and BackendServices five-argument construction.
  Backend naming lane owns the moved `AgentTitleGenerationTest.kt` fixture.
- Deleted control-only `HooksIntegrationTest.kt` and its private control fixtures;
  normal Minimal Agent/Session tests remain. Replacement is
  `hook/impl/notification/src/commonTest/.../NotificationHooksRpcTest.kt`:
  real backend composition → JSON in-memory RPC → public collect function → actual Shell.
  It tests failed-command continuation, ignored legacy control output, real ten-second
  timeout followed by a command, and preserved mixed pending calls/aggregation.
  An excluded isolated Session probes subscription readiness; no extra notification
  queue, fake executor, live API, user script or real Home is involved.
  New process fixtures support POSIX shells and PowerShell; Cmd-only fixture execution
  remains an explicit unsupported-platform case, not a claimed pass.
- Preserved the original notification executor internals byte-for-byte after the public
  KDoc boundary, and retained all eight execution plus two real-process tests. Added
  concrete four-branch matching. New model-spec tests cover descriptors, enum wire
  values and validation independently of RPC models.
- Tool tests retain search, current/dynamic routing, pending/unowned boundaries,
  removed MCP, invalid input, current cwd, original output, independent command
  completions and nonduplicate state-bound plan completion. Former Hook assertions
  now prove direct handling and uninterpreted legacy-looking output.
- Compact tests retain retry/continuation, auto pre-/mid-turn, pending and token reset;
  former observation tests now prove one direct manual commit and preserved metadata.
  Session composition adds natural stop/no injected context or continuation, later turn
  identity, mixed pending preservation, and MCP → catalog → client close/suppression.
  Coordinator owns all old Gradle/registry deletion; new-project test dependencies
  are recorded in the impl build here. Runtime retry policy is untouched.
- Static evidence: inner/outer `git diff --check` clean; new trees have no trailing
  whitespace; old Hook/TurnHook source trees contain no files; model/enum/public collect
  each have one definition. Current shared source scan has no `hook.contract`, `hook.impl`,
  TurnHook/NoOp, ToolHooks or CompactionHooks references. Original model declarations
  and executor internals compare identically to baseline. Frozen migration main sources
  and `CompactionRetryPolicy.kt` have no diff.
- Status: **implementation ready, pending central build**. No Gradle, IDE, native
  execution, live resources, commit or push performed by this lane. Coordinator owns
  full-host UI regression, target compilation/testing, old build deletion and registry.

### Central validation

- 通知 impl 14 项 JVM 测试通过，含真实 RPC → 本机隔离命令的三条用例；
  验证非零退出/十秒超时、清理、JSON 输入、旧控制输出无效与 mixed pending 保留。
- Runtime 三 decorator 共 33 项、Session 两实现 126 项 JVM 测试通过；
  composition 编译通过，无独立 test 源码，不将 NO-SOURCE 计为测试。
- Hook Settings VM/View 23 项及 Application 唯一订阅/Settings→Login 生命周期回归通过。
- 新 Hook spec/impl、Runtime 与 Session Linux x64 主源码编译通过；
  原 Notification wire/models 与 Shell 执行算法保持。
- 删除四个旧控制项目及全部 API/注入参数；当前 SOP 已更新为通知型 Hook 唯一归属。
  完整范围见[统一验收证据](2026-10-06-reassign-shared-application-behavior.md#central-acceptance-evidence)。
  用户已验收，内层本地提交 `206266c8`，未推送。

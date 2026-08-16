# CLI Session 与 Agent ViewModel 边界

## 所有权层级

- 保留一个应用级 `ApplicationViewModel`，并为每个已打开的真实 Session 建立一个稳定的 `PersistedSessionViewModel`。
- 每个 persisted Session ViewModel 直接包含常驻 `rootAgent`、轻量 Agent topology、`StateFlow<AgentViewModel>`
  `selectedAgent` 和按需 materialize 的 Agent ViewModel registry。
- 使用 root `sessionIndex` 标识 Session ViewModel，使用 `(sessionIndex, agentId)` 标识其中的 Agent ViewModel；Agent
  storage id 不能单独充当跨 Session 地址。
- Root Agent ViewModel 在创建 Session ViewModel 时立即 materialize；subagent ViewModel 只在被选择、展开或其他 UI 明确请求时按需
  materialize。
- Agent ViewModel 通过 parent address、direct-child slots 和所属 Session ViewModel 的 registry 表达递归树关系；构造父节点不得递归构造全部后代。
- Application、Session 和 Agent ViewModel 的状态所有权与 history source
  必须遵守[CLI ViewModel状态与懒History](cli-view-model-state.md)。
- 将虚拟 `NewSession` 建模为应用级 tab registry 中的独立 `NewSessionViewModel`；每个可见 New session tab 有自己的
  draft，不得伪造真实 Session 或 Agent identity、storage、lease、runtime。
- 将终端布局、焦点、hover、popup anchor 和 Agent tree expansion 保留为 renderer-local 状态；History 的
  `LazyListState`、follow-latest 和展开状态由对应 `AgentHistoryViewModel` 持有。

## Contract模块

- 应用模块按层级优先组织：`app/contract/<domain>`、`app/viewmodel/<domain>` 与 `app/view/<domain>`；不要用显式 `shared` 包裹
  ViewModel 层。
- `app/view` 通过 `mosaicMain` 承载当前唯一 renderer 实现，`app/viewmodel` 承载具体实现，`app/contract`
  承载公开边界；JVM desktop 与多 frontend 设计当前不在支持范围。
- `app/cli` 只保留 CLI entrypoint、Mosaic host 和进程生命周期；不得重新吸收领域 screen、terminal component 或
  renderer-local state。
- 使用 `app-contract-history`、`app-contract-agent`、`app-contract-session`、`app-contract-session-catalog`、
  `app-contract-application`、`app-contract-settings` 与 `app-contract-path-picker`；统一 `SessionViewModel`
  parent、persisted child 与 New Session child 已归入同一个 Session contract，不保留 `session-v2` 或独立 New Session
  contract。
- 依赖方向固定为 `lazy-list <- history <- agent <- session <- application`、`session-catalog <- application` 与
  `settings <- application`；settings contract 不反向依赖 Application。
- Opened child registry 归 Application；catalog 列表归独立 `SessionCatalogViewModel`，该 child 只在 Application 打开
  Select Session popup 时创建。
- NewSession contract 的 `materialize()` 在自身命令边界捕获最新 settings 与 composer，直接返回稳定
  `PersistedSessionViewModel`；不发布 materialization state、request 或 result wrapper。
- Application navigation 在同一 immutable snapshot 中只发布 `List<SessionViewModel>` 与 `selectedIndex`；同一 child
  handle 不得重复出现，selected child 只从下标派生。
- `ApplicationViewModel.materializeNewSession(tabIndex)` 以当前 navigation slot 为父级 mutation address，不接收 New
  Session child handle；参数不得与 persisted `sessionIndex` 混用。
- Persisted Session 直接发布稳定 `rootAgent` 和已 materialized 的 `selectedAgent` handle；选择命令先 materialize，再更新该
  Flow。
- Agent 持有稳定 composer、history、request-user-input 与 Shell registry child handle；父级 state 不复制这些 child 的
  mutable state。
- Contract 模块只公开接口、必要状态、identity、命令、child handle 与 factory port；不得依赖 Koin、具体 runtime、
  repository、storage 或 client 实现。当前单一 frontend 的 History contract 可以公开 `app-contract-lazy-list` 中的
  `LazyListState` 与 scroll interaction contract，但不得公开 renderer composable。
- 只公开跨实现模块真实使用的 factory port；进程 composition root、Agent runtime/history bridge 和其他基础设施参数对象留在实现层。
- `app/viewmodel` 模块统一应用 Koin Compiler Plugin、annotations、compile safety、strict safety 与 unsafe DSL checks；
  `app/contract` 不应用 Koin。
- `KodexApplication` 是唯一进程 composition root；它使用 typed `koinApplication` 创建隔离图，并把路径、repository、scope、auth
  与 client 等运行时值注入准确的实现定义。

## Application-global ViewModel

- 应用级 ViewModel 只持有 opened `SessionViewModel` registry、有序导航、当前独占 popup、registry command 和 shutdown
  command。
- `SessionCatalogViewModel.state` 必须显式区分 `Unloaded`、`Loading` 与 `Loaded`；`Loaded.sessions` 包含 root
  `threadName` 等轻量摘要，必须无需打开 root runtime 读取；每次 Select Session popup 打开时创建新 child，构造时不执行
  catalog I/O，frontend 取得 child 后才调用 `refresh()`。
- 当前单终端 frontend 的 `selectedIndex` 与有序 child registry 属于应用级 ViewModel；切换标签只更新下标，不销毁未关闭的
  persisted Session 或 New Session draft。
- Session tab 创建/打开/关闭/delete 和 application shutdown 由应用级 ViewModel 编排；fork 由准确
  `PersistedSessionViewModel` 执行，Session browser 数据和 global settings 命令分别由准确 child ViewModel 负责。
- 保留一个应用级命令串行化边界处理跨 Session 生命周期和 shutdown flush；全局设置提交由 `GlobalSettingsViewModel`
  串行化，Agent response job 继续由现有执行层并发运行。
- Application 不建立全局 notification 或 lifecycle 投影；命令通过返回值或异常报告结果，准确 child 的通知留在对应 child。
- Application popup state 只能表达 `Closed` 或一个具有对象身份的 `Open` handle；open variant 直接携带 Session
  Catalog、Settings、Rename、Delete 或 Working Directory 的准确 child ViewModel，不使用 `content + requestId`
  消息模型。
- Popup opening 必须先创建新 child，再原子替换 state 并关闭旧 child；dismiss 必须携带 expected open handle，只有它仍是当前
  popup 时才能关闭，owner 失效时也必须关闭对应 popup。
- Popup anchor、焦点、布局和 renderer 组件实例留在 frontend；popup child 的 draft、目标和确认命令归准确 child ViewModel。

## Per-session ViewModel

- 每个 persisted Session ViewModel 持有 session index、生命周期状态、稳定 `rootAgent`、轻量 Agent topology、materialized
  Agent registry 和 `selectedAgent` handle。
- Session root title、aggregate running、Agent tree、Session 级通知、打开/关闭状态和 Agent selection 属于 Session ViewModel。
- Session ViewModel 必须从轻量 topology 构造 Agent 树，不得为了显示树节点而读取所有 Agent history 或 materialize 全部
  Agent ViewModel。
- Session ViewModel 负责按 Agent address `getOrPut` Agent ViewModel，并保证同一 Agent 在一个 Session 中只有一个共享实例。
- Session ViewModel 拥有 Agent ViewModel，但不得把子 Agent 的 mutable state 扁平复制进 Session state。
- Session ViewModel 的 root Agent ViewModel 始终存在；subagent slot 可以只包含轻量摘要而没有已 materialize 的 ViewModel。
- 选择未 materialize Agent 时先从 registry 创建或取得稳定 Agent ViewModel，再原子发布 `selectedAgent`；frontend 不通过
  address 二次解析 active Agent。
- Persisted Session 的 `fork(source, target)` 只接受自身 registry 中的 exact Agent child，在该 Agent 的 committed
  boundary 创建新 persisted root Session 并返回 index；foreign child、stale target 或 running source 必须失败且不修改
  source。
- Fork 不修改 Application navigation，也不自动打开新 Session；是否将返回的 index 交给 `ApplicationViewModel.openSession()`
  是独立 frontend navigation 选择。
- Session ViewModel 的 `shutdown()` 停止接收新命令并按确定顺序 flush/关闭全部已 materialize Agent ViewModel；同步
  `close()` 只作为 scope disposal 的幂等取消后备。
- Session ViewModel 关闭全部 materialized child handle 后才释放所属 Session manager 资源；Agent runtime、storage 和
  coordinator 的底层释放仍由 manager 负责。

## Per-agent ViewModel

- 每个 Agent ViewModel 只发布一个 Agent 的完整 `StateFlow<KodexAgentSettings>`、execution、token count、direct
  children、history action、notification 与 lifecycle，并直接持有自己的 child ViewModel；committed、pending tool 和
  streaming History 状态由 history child 统一拥有和分别发布。
- `threadName` 与 `plan` 从 `KodexAgentSettings` 读取；不得另建 Agent summary 或 plan state。
- Agent settings 通过 model、working directory、reasoning effort、service tier、agent mode 与 request-user-input mode 的按字段方法更新；每个方法必须基于该
  Agent 的最新完整快照保留其他 runtime-owned 字段。
- Agent运行期间settings命令继续写入同一个AgentState串行边界，并作为后续请求的配置；frontend不得因active turn将这些命令标记为不可编辑。
- Composer、composer revision、checkout 确认、Agent 通知和 request-user-input answer draft 按 Agent ViewModel 隔离。
- Request-user-input identity 使用 Agent address 与 call id 的组合；auto-resolution job 按 Agent ViewModel 独立持有和取消。
- Request-user-input 作为 Agent 的稳定 child handle，以 `Idle` 或携带 call id、args、answers、revision、submission phase 的
  `Pending` 原子发布；每次 edit 和 submit 都必须校验显式 call id。
- Agent ViewModel 必须暴露 parent address 和 direct children 的轻量 slot；child slot 可以没有已 materialize 的 ViewModel。
- Direct children 使用显式 `Unloaded`、`Loading`、`Loaded` 或等价状态；展开一个节点只允许 materialize 它的 direct
  children，不递归 materialize descendants。
- 首版可以将已 materialize 的非 root Agent ViewModel 缓存到 Session 关闭，以保留 composer 和 dirty draft；不得因此提前
  materialize 未访问节点。
- Agent ViewModel 的事件和异步 completion 必须捕获显式 Agent address 与 revision，不能在执行时重新解析应用级 active
  Session 或 Session 级 selected Agent。
- Session/Agent ViewModel 不随 selected tab 或 selected Agent 的变化而销毁；切换只替换当前 renderer 的显示对象。
- frontend 提交一旦被 Agent ViewModel 接受，initial turn、resume、compact 与 request-user-input continuation 必须由
  仍然存活的 Agent ViewModel scope 持有；旧 renderer 的调用协程被取消时，只结束 frontend 的等待，不能取消已接受的后台工作。
- 只有 Agent 的显式 Stop、Agent close 或 Session shutdown 可以取消 Agent ViewModel 已接受的长时运行工作；不得以
  `NonCancellable` 或常驻隐藏 renderer 掩盖错误的任务所有权。
- root thread 的 `threadName` 由其 Agent Runtime ViewModel 持久化；Session 级 Rename 必须始终定位 root Agent，不能改写当前选中的
  subagent。
- Agent ViewModel 关闭时只取消自身 UI job、timer 和订阅；Agent runtime、storage 和 coordinator 的资源关闭仍由 Session
  manager 执行。

## 轻量拓扑与详细投影

- 使用 coordinator 发布的 Agent identity、parent、task name、status 和 activity version 构建 Session ViewModel
  的轻量树索引；展示树结构不得要求读取完整 Agent history。
- 未 materialize 的 Agent 只保留树摘要和执行层本来就需要的 transient state，不构造或缓存完整 conversation projection。
- 将当前 `SessionSnapshot` 明确为 Agent 级详细状态并改用 Agent 命名；Session ViewModel state 与 Agent 详细 state
  必须使用不同模型。
- Materialize Agent ViewModel 时绑定既有 AgentSession storage，只读取轻量状态与有限 history tail，并合并执行层仍在进行的
  stream/activity；不得构造一次完整 history 快照。
- 后台 Agent 的 status 和 activity version 继续更新所属 Session ViewModel 的轻量 topology；只有已 materialize Agent
  ViewModel 接收按需 history source、stream 和详细 UI 投影。
- Session manager 操作必须显式接收 Session/Agent identity 或稳定 handle；只移除隐式 selected routing，不改变现有 runtime
  链和 response 执行语义。

## NewSession ViewModel

- 每个可见的 New session tab 持有一个 NewSession ViewModel；它只持有进程内 `MutableStateFlow<KodexAgentSettings>` 与独立
  composer。
- NewSession的默认标签名只用于草稿显示；未显式命名的草稿物化时必须以`Session <sessionIndex>`初始化root thread，保持自动标题生成资格，显式命名则原样持久化。
- `KodexGlobalSettings.newSession` 继续是 defaults 的唯一持久化真源；NewSession ViewModel 只在创建时将 defaults
  转为自己的非持久化完整 settings，不建立第二份持久化 authority。
- NewSession草稿创建时复制默认`RequestUserInputMode`，subagent创建时复制父Agent当前值；复制后每个草稿或Agent独立更新该字段。
- Application 在 surviving command scope 中按 `tabIndex` 解析 exact New Session child 并调用其 `materialize()`；成功后用一次
  navigation update 在原位置替换返回的 persisted child、保持 `selectedIndex` 并关闭已消费的虚拟 child。
- Materialize 开始时 index 无效或 slot 不再是 New Session，以及 child materialization 失败，都不得改变 navigation；后者保留原
  draft。
- Materialization 失败直接抛给调用方且不改变 navigation；原 New Session settings 与 composer 保持可编辑。
- 应用级 tab registry 支持同时存在多个未物化 NewSession draft；它们仅在当前应用进程存活，不写入持久化 Session catalog。

## 生命周期与验证

- Startup 只创建初始 NewSession ViewModel 和 Application 自身状态，不创建或加载 Session catalog popup child，也不为未打开的持久化
  Session 建立 Session ViewModel、Agent ViewModel 或 runtime。
- Open Session 创建或复用 Session ViewModel，并且只 materialize root Agent ViewModel；既有 subagent 不得触发完整 UI 投影。
- 展开或选择 subagent 时由所属 Session ViewModel materialize 所需路径或 direct children，并复用 registry 中已有的 Agent
  ViewModel。
- 切换 selected Session 只更新应用级下标；每个 Session ViewModel 保留自己的 selected Agent、Agent registry 和 Agent UI
  drafts。
- 验证 Agent ViewModel 在 frontend 调用域取消前后保持同一存活实例，已接受 turn 的 execution 与 Session aggregate running
  仍保持运行并能正常完成；再验证显式 Stop 与 Session shutdown 仍会取消它。
- Close Session 先在一个 navigation transaction 中移除目标并计算仍有效的 `selectedIndex`，再调用对应 Session `shutdown()`
  关闭已 materialize projection，最后释放 manager 资源。
- Application `shutdown()` 停止接收新命令，先关闭当前 popup child，再关闭全部 NewSession child、调用每个 Session 的
  `shutdown()`，最后释放 application resources；同步 `close()` 只作为进程 disposal 的幂等取消后备。
- 验证打开含大量 completed subagent 的 Session 只创建一个 Session ViewModel 和 root Agent ViewModel，不读取所有 subagent
  history。
- 验证按需展开只 materialize direct children，选择深层 Agent 只加载所需路径且 parent/children 关系正确。
- 验证两个 Session ViewModel 的 selected Agent、Session 通知和 Agent registry 相互隔离。
- 验证多个 Agent ViewModel 的 composer、设置草稿、pending input、通知及 auto-resolution 相互隔离。
- 验证 fork 只能使用所属 Session 的 exact Agent handle 和当前 generation committed target，并且成功或失败都不改变
  Application navigation。
- 验证未 materialize 的后台 Agent 继续发布轻量运行状态，materialize 后能按视口读取 committed history 并取得当前 transient
  stream。
- 验证全局设置和模型目录更新可被全部 Session/Agent ViewModel 观察，但不会改写已有 Agent 的持久化 settings。
- 验证当前单一 Mosaic frontend 在切换 Session 或 Agent 后复用对应稳定 ViewModel，并恢复其 History 滚动与展开状态。

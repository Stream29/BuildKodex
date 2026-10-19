# Task Tree

- `Confirm singleton New Session navigation and settings design`()
- `Specify contract and checklist updates for singleton New Session`()
- `Review submission boundaries and resolve blockers with user`()
- `Freeze interfaces, ownership, and acceptance gates`()
- `Update current project checklists`()
- `Approve first worker batch`()
- `coroutineScope {`
  - `val backend = async {`
    - `Implement backend defaults creation`()
  - `}`
  - `val draft = async {`
    - `Implement reusable singleton draft`()
  - `}`
  - `val settings = async {`
    - `Implement persisted Session and Settings boundaries`()
  - `}`
  - `awaitAll(backend, draft, settings)`
- `}`
- `Review foundation diffs against frozen interfaces`()
- `Approve Application worker`()
- `coroutineScope {`
  - `val application = async {`
    - `Implement Application navigation and production assembly`()
  - `}`
  - `val rpcGate = async {`
    - `Run independent foundation RPC validation`()
  - `}`
  - `awaitAll(application, rpcGate)`
- `}`
- `Review Application ownership and submission integration`()
- `Approve final-wave workers`()
- `coroutineScope {`
  - `val renderer = async {`
    - `Implement singleton TUI rendering`()
  - `}`
  - `val integration = async {`
    - `Implement cross-module regression coverage`()
  - `}`
  - `awaitAll(renderer, integration)`
- `}`
- `for (gate in acceptanceGates) {`
  - `Run parent-owned validation and review evidence`(gate)
  - `while (hasTaskRegressions(gate)) {`
    - `Resolve scoped defects and revalidate`(gate)
  - `}`
- `}`
- `Release owned validation resources and present acceptance evidence`()

# Details

- 2026-10-19 用户先完成设计审阅，随后授权细化编排、拆分给 subagent 实现，由主会话验收；任务进入 executable。普通 Session 的创建仍经工具确认才启动，不把任务记录当作后台执行已经开始的证明。
- 基线：BuildKodex `68a62c821f02c566fbd4a3ecf2ddf40fa15b122c`，Kodex `a611bba9f835ff54a23d41834b95f752d0557ca6`。开始前 Kodex 仅 `gradle.properties` 有既存修改，必须原样保留；外层其他 discussion/executable/done 未提交文件不属本任务。
- 本地 IDEA 进程启动目录为 PersonalSkills；不操作窗口、不触发本地 Sync。其他任务已使用远端做隔离实验，重型验证必须先检查资源占用，不能沿用旧任务授权或抢占其设备。
- 派发前已冻结 1,300 个受跟踪文件的工作树内容、SHA-256、submodule HEAD 与既存 diff，保存在 [本轮基线](file:///home/stream/ACodeSpace/demo/kodex-new-session-948/baseline/)；源码压缩包不含嵌套 submodule 内容，需结合所记录的 submodule HEAD/现有 checkout 验证。

## 1. 已确认设计决策汇总

- **全局唯一 `NewSession` 与联合导航态**：
  - `New Session` 移出 `tabs`，作为 `ApplicationViewModel` 持有的全局唯一实例（`val newSession: NewSessionViewModel`）。
  - `ApplicationNavigationState.tabs` 仅包含 `List<PersistedSessionViewModel>`（允许为空列表 `emptyList()`）。
  - 导航选中态为联合类型 `ApplicationTabSelection`：`NewSession | SelectedTab(index: Int)`；当 `tabs` 为空或关闭最后一个 tab 时自动处于 `NewSession`。
- **顶栏 `+` 按钮与标签交互**：
  - 点击顶栏右侧 `+` 按钮切换到 `NewSession`；当处于 `NewSession` 态时，`+` 按钮与普通选中 Session Tab 一样高亮（`selected = true`），普通 Session Tab 均不高亮。
  - 点击任意 `SessionTabItem` 切换到对应的 `SelectedTab(index)`。
- **状态保留、提交重置与设置双向同步**：
  - 切换到普通 Session Tab 不会丢失 `NewSession` 的未发送 `composer` 文本与当前 `cwd`。
  - `NewSession` 的 4 项模型设置（`model`、`reasoningEffort`、`serviceTier`、`requestUserInputMode`）与全局 `New Session Defaults`（[`KodexNewSessionSettings`](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/new-session-defaults/viewmodel/src/specMain/kotlin/io/github/stream29/kodex/cli/settings/KodexNewSessionSettings.kt#L18-L23)，即 `BackendSettings.newSession`）双向同步，共用同一持久化设置真源。
  - `cwd` 初始为 CLI 启动工作目录，仅在成功提交创建并启动新 Session 后，连同 `composer` 一起重置（`composer` 清空，`cwd` 恢复为 CLI 启动工作目录）。
- **提交（创建并跳转）与异常边界**：
  - 只有 `text.length == 0` 时不提交、不创建；空格、换行等非零长度字符串也是有效输入，按原文发送，不调用 `trim()` 或用 `isBlank()` 判空。
  - 普通 `createSession` 只接收 `cwd`；后端在实际创建时读取自己持有的 `BackendSettings.newSession`，组装初始 `KodexAgentSettings`。前端的设置投影仅供显示和编辑，不作为创建时四项默认配置的真源。
  - 不为设置修改与发送增加等待、禁发或跨请求同步机制；采用后端实际创建时已持有的默认值，不承诺尚未到达后端的编辑已经生效。
  - 正常路径依次为：捕获输入与 `cwd`、创建并打开持久化 Session、追加首条消息、调用该 Session 的 `resume()`；然后追加并选中新 Tab、清空已消费输入、将 `cwd` 重置为 CLI 启动目录。
  - 成功边界是首条消息已写入且 `resume()` 已接受启动请求，不等待模型首个输出或整轮完成。`resume()` 后异步运行失败走既有错误出口，不恢复 New Session 草稿。
  - 用户确认创建/提交失败属于异常路径，本轮不设计专门的重试、续交、草稿转移、补偿或失败后自动跳转。异常交给既有错误出口；已确认创建的 Session 不自动删除，不盲目重放结果未知的创建或追加，也不承诺再次操作会复用同一 Session。
- **移除草稿伪装层与收敛接口层级**：
  - [`NewSessionViewModel`](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/new-session/viewmodel/src/specMain/kotlin/io/github/stream29/kodex/app/session/contract/NewSessionViewModel.kt#L18-L45) 不再继承已删除的共同接口 `SessionViewModel`，移除草稿级 `name`、`rename()`、`clearExplicitThreadName()` 与 `nextDraftOrdinal`。
  - 移除 `SessionSettingsTargetKind.NewSessionDraft` 与 `RpcDraftSettingsSource`；无真实目标时使用 [`UnavailableSessionSettingsDataSource`](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/rpc/src/commonMain/kotlin/io/github/stream29/kodex/cli/rpc/UnavailableSessionSettingsDataSource.kt#L14)。
  - [`ApplicationViewModel.openSettingsPopup`](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/application/src/specMain/kotlin/io/github/stream29/kodex/app/application/contract/ApplicationViewModel.kt#L168-L171) 与 [`SettingsViewModelArguments`](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/settings/viewmodel/src/specMain/kotlin/io/github/stream29/kodex/app/settings/contract/SettingsViewModel.kt#L96-L101) 的 `target` 改为可空 `PersistedSessionViewModel?`；在 `NewSession` 视图点击底栏 `Settings` 时传入 `target = null, initialPage = SettingsPage.NewSession`，此时 `SettingsPage.CurrentSession` 展示 `SessionSettingsState.Unavailable`（`"No selected session"`）。

---

## 2. 模块与契约变更规格

### 2.1 `app/session/contract` 与 `app/component/new-session`

- **共同接口 `SessionViewModel.kt` 删除；成员合入 [`PersistedSessionViewModel.kt`](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/session/contract/src/specMain/kotlin/io/github/stream29/kodex/app/session/contract/PersistedSessionViewModel.kt#L27)**：
  - 将 `SessionViewModel` 的 `name: StateFlow<String>`、`rename(name: String)`、`AgentSettingsViewModel`、`AutoCloseable` 直接合入 `PersistedSessionViewModel`，删除失去共同消费者的 `SessionViewModel` 父接口；同步更新 Settings、Rename popup、registry 及测试的目标类型。
- **[`NewSessionViewModel.kt` (specMain)](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/new-session/viewmodel/src/specMain/kotlin/io/github/stream29/kodex/app/session/contract/NewSessionViewModel.kt#L16)**：
  - 独立实现 `AgentSettingsViewModel` 与 `AutoCloseable`，不再继承 `SessionViewModel`。
  - 移除 `name`、`rename()`、`clearExplicitThreadName()`，以及 `NewSessionViewModelArguments.defaultName`；构造参数只保留本地启动目录等准确输入，不再复制一份独立可写的默认配置。
  - 保留 `composer: ComposerViewModel`、`runtimeConfiguration: RuntimeConfigurationViewModel`、`settings: StateFlow<KodexAgentSettings>`、`models: StateFlow<List<ModelInfo>>`。
  - 4 项模型设置更新（`updateModel`、`updateReasoningEffort`、`updateServiceTier`、`updateRequestUserInputMode`、`updateModelConfiguration`）直接作用于共享的 `BackendSettings.newSession`（通过与 `NewSessionDefaults` 一致的后端 `GlobalRpc` CAS 更新）；`updateWorkingDirectory(workingDirectory: Path)` 更新本地 `cwd: StateFlow<Path>`。
  - `submit(expectedRevision: Long): PersistedSessionViewModel?` 替换 `materialize()`，捕获 `composer` 的准确 revision、原文及本地 `cwd`；只将 `cwd` 交给后端创建，追加时使用原文。零长度、过期或进行中重复提交返回 `null`，成功边界与重置时机遵循第 1 节。
  - 提交 admission 必须拒绝重复/进行中的提交，不能只依赖 mutex 将重复事件排队。宿主回调携带捕获的 composer revision，并在命令边界校验；成功后的旧回调不得消费后续新草稿。
  - 已接受提交不随 New Session renderer 卸载而取消；使用存活的前端 owner 完成编排。输入清空校验准确 revision，cwd 重置不得覆盖提交后新发生的目录编辑；不引入失败恢复状态机。
- **[`RpcSessionDraft.kt`](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/rpc/src/commonMain/kotlin/io/github/stream29/kodex/cli/rpc/RpcSessionDraft.kt#L34-L145) & [`NewSessionViewModel.kt` (commonMain)](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/new-session/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/cli/newsession/NewSessionViewModel.kt#L14-L61)**：
  - 重构为全局可复用的 `NewSession` 状态持有者：订阅 `RpcGlobalSettings.settings.map { it.newSession }` 与本地 `mutableCwd = MutableStateFlow(initialCwd)` 组合出 `settings: StateFlow<KodexAgentSettings>`。
  - 模型字段修改直接通过 `global.settings.editField(...)` 原子写入 `BackendSettings.newSession`；`updateWorkingDirectory` 修改 `mutableCwd`。
  - 每次被接受的有效新提交创建一个持久化 Session；移除单次消费的 `createdIndex`、`submitted`、草稿命名状态与永久 `mutableEditable = false`。正常完成后单例继续可用，提交去重与输入保护遵循上述 admission 规则。
  - `settings` 由后端默认值投影与本地 cwd 派生，不通过 CAS 回执回写镜像；字段冲突、取消与持久化失败沿用现有规则，原子 model/effort/tier 更新保留其他默认字段。

### 2.2 `app/component/session-settings` 与 `app/component/settings`

- **[`SessionSettingsViewModel.kt`](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-settings/viewmodel/src/specMain/kotlin/io/github/stream29/kodex/app/settings/contract/SessionSettingsViewModel.kt#L20-L42)**：
  - 移除 `SessionSettingsTargetKind.NewSessionDraft` 及只剩单一类别的 `SessionSettingsTargetKind` 枚举，因 `SessionSettings` 仅针对已持久化 Session。
- **[`RpcSessionSettings.kt`](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/rpc/src/commonMain/kotlin/io/github/stream29/kodex/cli/rpc/RpcSessionSettings.kt#L21)**：
  - 删除 `RpcDraftSettingsSource`。
  - 提供独立的 [`UnavailableSessionSettingsDataSource`](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/rpc/src/commonMain/kotlin/io/github/stream29/kodex/cli/rpc/UnavailableSessionSettingsDataSource.kt#L14)（state 恒为 Unavailable），供 `target == null` 时装配 `SessionSettingsViewModel`。
- **[`SettingsViewModel.kt`](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/settings/viewmodel/src/specMain/kotlin/io/github/stream29/kodex/app/settings/contract/SettingsViewModel.kt#L96-L101)**：
  - `SettingsViewModelArguments(val target: PersistedSessionViewModel?, val initialPage: SettingsPage)` 允许 `target` 为 `null`。

### 2.3 `app/application`

- **[`ApplicationNavigationState.kt`](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/application/src/specMain/kotlin/io/github/stream29/kodex/app/application/contract/ApplicationNavigationState.kt#L16-L40)**：
  - 定义联合选择类型：
    ```kotlin
    public sealed interface ApplicationTabSelection {
        public data object NewSession : ApplicationTabSelection
        public data class SelectedTab(public val index: Int) : ApplicationTabSelection
    }
    ```
  - `ApplicationNavigationState(val tabs: List<PersistedSessionViewModel>, val selection: ApplicationTabSelection)`：
    - 允许 `tabs` 为空（此时 `selection` 必须为 `ApplicationTabSelection.NewSession`）。
    - 当 `selection is SelectedTab` 时，校验 `selection.index in tabs.indices`。
    - 校验 `tabs` 中无重复引用实例（`===`）。
    - 提供派生属性 `val selectedSession: PersistedSessionViewModel?`。
- **[`ApplicationViewModel.kt` (specMain & commonMain)](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/application/src/specMain/kotlin/io/github/stream29/kodex/app/application/contract/ApplicationViewModel.kt#L37-L252)**：
  - 新增稳定属性 `public val newSession: NewSessionViewModel`。
  - 初始 `navigation` 为 `ApplicationNavigationState(tabs = emptyList(), selection = ApplicationTabSelection.NewSession)`。
  - 将 `createNewSessionTab()` 替换为 `selectNewSession(): Unit`（切换到 `ApplicationTabSelection.NewSession`）。
  - 将 `materializeNewSession(target)` 替换为 `submitNewSession(expectedRevision: Long): PersistedSessionViewModel?`：
    - 对准确草稿 revision 做 admission，零长度、过期或进行中的重复提交无副作用地返回 `null`；不是将重复提交排队后各建一个 Session。
    - 在应用命令串行边界内执行第 1 节正常路径，成功后将返回的 Session 追加到 `tabs` 并设置 `selection = SelectedTab(tabs.lastIndex)`。
    - 真实异常和取消继续传播，不为异常增加返回部分实体、强制发布 Tab 或自动恢复分支。
  - `selectTab(target: PersistedSessionViewModel): Boolean`、`closeTab(target: PersistedSessionViewModel): Boolean`、`openRenameSessionPopup(target: PersistedSessionViewModel)` 均直接接收 `PersistedSessionViewModel`。
  - `closeTab`、`closeAndArchiveSession` 与 `deleteSession` 移除目标 Tab 后：空列表回到 `NewSession`；原来处于 `NewSession` 则保持；选中的 Session 仍存活则保持同一实例并调整下标，只有选中项被删除时才夹取到原位置的后继或末项。
  - `openSettingsPopup(target: PersistedSessionViewModel?, initialPage: SettingsPage)`：当 `target != null` 时校验仍在 `tabs` 中；当 `target == null` 时绑定无目标（若 `target` 被关闭则只关闭绑定了该 `target` 的 Popup）。
  - `ApplicationPopupState.Settings.target` 同步改为可空 persisted target；Login 的 `returnTo` 保留该绑定，`null` 不隐式绑定当时或后来的选中 Session。
  - `openWorkingDirectoryPopup(target: AgentSettingsViewModel)`：允许 `target` 为 `newSession` 或已拥有 Tab 及其当前 `rootAgent`。
  - `openCreatedSessions` 保持只追加、不改变当前选择；`openSession` 继续复用稳定 persisted handle 并选择它。启动与关闭最后一个 Tab 都不重新构造 New Session；shutdown 单独关闭唯一 New Session child，关闭普通 Tab 不 Stop 后端任务。

### 2.4 `app/component/session-tab-bar` 与 `app/view`

- **[`SessionTabBar.kt` (specMain & mosaicMain)](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-tab-bar/view/src/specMain/kotlin/io/github/stream29/kodex/app/sessiontabbar/contract/SessionTabBar.kt#L41-L79)**：
  - `SessionTabBarState(val tabs: List<SessionTabPresentation>, val selected: SessionTabIdentity?)`：`selected == null` 表示当前处于 `New Session` 状态。
  - 右侧 `+` 按钮（[`SessionTabBar.kt#L133-L138`](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-tab-bar/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/sessiontabbar/SessionTabBar.kt#L133-L138)）设置 `selected = state.selected == null`，并使用与选中 Tab 一致的高亮表现；点击调用 `callbacks.selectNewSession()`（原 `createNewSession()`）。
- **[`SessionTreeCliScreen.kt`](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/SessionTreeCliScreen.kt#L143-L645)**：
  - 移除所有 `tabs.filterIsInstance<PersistedSessionViewModel>()` 和 `as? PersistedSessionViewModel` 分支。
  - 主视图根据 `navigation.selection` 分支：
    - `ApplicationTabSelection.NewSession`：渲染 `NewSessionScreen(viewModel = viewModel.newSession, ...)`，点击 `onBrowseWorkingDirectory` 传 `viewModel.newSession`，点击 `onOpenSettings` 传 `viewModel.openSettingsPopup(null, SettingsPage.NewSession)`，提交回调捕获本次输入 revision 并调用 `viewModel.submitNewSession(expectedRevision)`。
    - `ApplicationTabSelection.SelectedTab`：渲染 `selectedSession` 的 `AgentRuntimeScreen`，点击 `onOpenSettings` 传 `viewModel.openSettingsPopup(selectedSession, SettingsPage.CurrentSession)`。

### 2.5 普通创建 RPC 与后端默认设置

- **[GlobalRpc.createSession](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/client/src/specMain/kotlin/io/github/stream29/kodex/rpc/contract/GlobalRpc.kt#L275-L287)**：
  - 普通创建由 `createSession(initialSettings: KodexAgentSettings)` 收窄为 `createSession(cwd: Path): Int`，显式接入既有 `PathAsStringSerializer`，验证新的 RPC 参数可编解码；同步修改 KDoc、代理消费者和测试装配。
  - 不传模型四字段、初始标题或首条消息；后端仍按分配的编号生成 `Session <index>`，保留自动标题生成资格。
- **[BackendGlobalRpc](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/server/src/commonMain/kotlin/io/github/stream29/kodex/rpc/server/BackendGlobalRpc.kt#L60)** 与 **[BackendSessionManagement](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/server/src/commonMain/kotlin/io/github/stream29/kodex/rpc/server/BackendSessionManagement.kt#L49-L51)**：
  - 将后端默认值读取接入实际创建边界，一次捕获后端当前 `newSession`，结合请求 cwd 组装初始 Agent settings，再沿用原创建、校验及初始化流程；不先在前端组装再传回。
  - `createSuggestedSessions(tasks, initialSettings)` 保持显式配置语义；fork、已有 Session 设置与后端运行入口不改成读取全局默认值。
  - 创建和追加仍是两个独立操作，不新增事务式 create-and-submit RPC、幂等键或恢复协议。
  - 该 RPC 参数变化不兼容旧业务协议，将 `BackendBusinessProtocol` 从 `0.4.15` 更新为当前应用已有版本 `0.5.0`；管理协议保持 `1`。验证旧业务协议被拒绝且不停止/替换旧后端；不修改应用版本、历史迁移版本或执行发布。

---

## 3. Checklist 更新清单

- **[cli-session-view-models.md](file:///home/stream/ACodeSpace/push/BuildKodex/checklist/cli-session-view-models.md)**：
  - 更新 `Application` 导航状态描述：`tabs` 只持有 `PersistedSessionViewModel`，选中态为 `NewSession | SelectedTab(index)`，全局唯一 `NewSessionViewModel` 由 `ApplicationViewModel` 持有。
  - 在当前 RPC 边界明确唯一草稿、共享默认值和提交跳转；历史章节保留为旧记录，并显式标注其中多草稿/原位替换规则已失效。
- **[cli-view-model-state.md](file:///home/stream/ACodeSpace/push/BuildKodex/checklist/cli-view-model-state.md)**：
  - 更新 `Application与Session状态` 和 `Agent与NewSession状态` 中关于 `navigation`、`NewSessionViewModel` 与 `Settings` popup 目标的约束。
- **[frontend-application-boundary.md](file:///home/stream/ACodeSpace/push/BuildKodex/checklist/frontend-application-boundary.md)**：
  - 更新 `Application` 命令与 `NewSession` / `SessionSettings` 边界说明，移除 `NewSessionDraft` 数据源描述。
- **[rpc-session.md](file:///home/stream/ACodeSpace/push/BuildKodex/checklist/rpc-session.md#L60)**：
  - 更新普通创建的 cwd 参数、后端默认配置来源、长度判空与异常边界；移除“前端完整草稿设置传入”“可重试内容”的旧要求，保留独立创建/追加和结果未知时不自动重放的规则。
- **[rpc-settings.md](file:///home/stream/ACodeSpace/push/BuildKodex/checklist/rpc-settings.md#L14)**：
  - 明确 New Session 的四字段直接读写共享默认值，仅 cwd 与 composer 是本地草稿；原有 persisted Session 不随默认值变化。
- **[tui-interaction-components.md](file:///home/stream/ACodeSpace/push/BuildKodex/checklist/tui-interaction-components.md)**：
  - 更新顶栏 `+` 按钮选中高亮语义、`New Session` 状态栏模型设置与 `New session` 默认值双向同步、`cwd` 提交重置及 `Settings` 按钮默认打开 `New session` 页签的规则。
- 更新当前有效约束，不把标为历史阶段的旧装配或 `global-settings.md` 单文件模型改写成当前实现。

---

## 4. 验证检查项（Inline Checks）

- 验证应用启动时 `tabs` 为空、`selection` 为 `NewSession`，且不触发任何后端 Session 创建。
- 验证在 `NewSession` 修改 `model` / `reasoningEffort` / `serviceTier` / `requestUserInputMode` 时写入 `BackendSettings.newSession`，且从 `Settings -> New session` 修改时通过共享订阅反映到 `NewSession` 状态栏。
- 验证在 `NewSession` 修改 `cwd` 与输入草稿后切换到已打开的 `PersistedSessionViewModel` Tab 再点 `+` 切回，`cwd` 与草稿保持不变。
- 验证处于 `NewSession` 时顶栏 `+` 按钮为 `selected` 高亮态、普通 Tab 均不高亮；处于 `SelectedTab` 时对应 Tab 高亮、`+` 按钮不高亮。
- 验证零长度输入不创建；纯空格、纯换行和含首尾空白的非零长度输入均创建并原样追加，不裁剪文本。
- 验证同一 revision 的重复/进行中提交与成功后的迟到回调不多建 Session、不消费新草稿；下一次有效草稿仍能正常创建另一个 Session。
- 验证提交成功后创建新 Session 追加到 `tabs`、选中该新 Tab、清空已消费的 `NewSession.composer` 并将 `cwd` 重置为初始启动目录；创建后、追加前不得提前重置。
- 验证前端默认值投影有意落后时，普通创建使用后端实际持有的四字段和请求 cwd；不等待该前端投影追平，且不影响建议子任务显式配置或已有 Session。
- 验证创建/打开/追加失败走原异常出口，不自动重发、转移草稿或删除已创建实体；不将“结果未知”当作“没有发生”。不验收失败续交、强制跳转或重试不重复创建。
- 验证 `resume()` 的异步运行失败不回滚已成功提交的输入、不恢复已清空草稿；成功不等待模型输出。
- 验证切页/卸载 renderer 不取消已接受提交，输入与目录的迟到完成不覆盖后续编辑。
- 验证关闭最后一个 `PersistedSessionViewModel` Tab 时，`tabs` 变为空且 `selection` 自动回到 `NewSession`。
- 验证关闭/删除选中项前后的后台 Tab 时保持同一选中实例；在 New Session 状态移除后台 Tab 不切页、不丢草稿；批量打开已创建 Session 不改变当前选择。
- 验证在 `NewSession` 视图打开 `Settings` 默认进入 `SettingsPage.NewSession`，切换到 `SettingsPage.CurrentSession` 显示 `"No selected session"`（`SessionSettingsState.Unavailable`）。
- 验证无目标 Settings 与 Login returnTo 不被后来选择的 Session 重新绑定，关闭无关 Tab 不关闭它；shutdown 正确释放单例与 popup。
- 实现阶段运行变更所涉 RPC client/server、New Session、Application、Settings、Tab Bar 单元测试及现有 `integration-test` 跨模块测试；补充顶栏空列表、选择高亮和窄终端交互验收。检查项已编排，不表示已经执行或通过。

## 5. 设计审阅结论（2026-10-19）

- 用户已确认三项边界：异常路径不做专门重试/兜底；仅零长度不提交，空白原文有效；普通创建只传 cwd，由后端读取默认配置。
- 已消除原计划中“单例每次新建却保证失败续交不重复创建”“非空白/trim 与用户输入口径不符”“前端完整快照与后端默认值真源冲突”及 cwd 提前重置的矛盾。
- 已补充提交去重、准确草稿保护、导航移除规则、nullable Settings/Login 绑定、单例释放及普通创建 RPC 的改动与检查范围。
- 当前未发现需要继续拍板的产品阻塞；提交 admission 与 owner 规则是实现约束，不扩展为失败恢复能力。实现已获本轮用户授权，执行状态见 Task Tree 和下面的交接记录。

## 6. 实现编排与接口冻结

### 6.1 依赖图与唯一写入者

| 波次 | 子任务 | 文件归属 | 依赖与交付 |
| --- | --- | --- | --- |
| 1A | [Implement backend defaults creation](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-19-new-session-backend-defaults.md#L1) | `rpc/client`、`rpc/server`；backend-daemon 的业务协议常量及其针对性测试 | 独立实施；交付 cwd 创建接口、后端默认值读取、序列化和协议兼容测试 |
| 1B | [Implement reusable singleton draft](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-19-new-session-singleton-draft.md#L1) | `app/component/new-session/viewmodel`、`app/rpc/RpcSessionDraft.kt`、`app/rpc` 全部 commonTest、`app/test-support/rpc` | 按冻结接口实施；交付可复用单例、共享配置、提交/清空、真实 RPC fixture |
| 1C | [Implement persisted Session and Settings boundaries](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-19-new-session-persisted-settings.md#L1) | `app/session/contract`、`app/session`、`app/component/{session-settings,settings}`、`app/rpc/RpcSessionSettings.kt` | 独立实施；交付纯 persisted 目标、无目标 Settings 数据源与组件测试 |
| 2 | [Implement Application navigation and production assembly](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-19-new-session-application-navigation.md#L1) | `app/application`（含 spec、popup、装配、单元测试） | 1A/1B/1C 经主会话静态验收后派发；交付联合导航、单例 owner 和无草稿伪装的生产装配 |
| 3A | [Implement singleton TUI rendering](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-19-new-session-tui-rendering.md#L1) | `app/view`、`app/component/new-session/view`、`app/component/session-tab-bar/view` | 波次 2 静态验收后派发；交付顶栏、根 screen、焦点/菜单/终端标题及 renderer 测试迁移 |
| 3B | [Implement cross-module regression coverage](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-19-new-session-integration-coverage.md#L1) | `integration-test` | 与 3A 独立实施，消费冻结的 renderer 接口；交付真实 backend/JSON/frontend/renderer 回归 |
| 主会话 | 编排、规范更新、集成修正、验收 | 本父任务、`checklist/`、本任务证据目录；其他文件仅在 worker 交还后接手 | 不与活动 worker 同改文件；独占组织 Gradle/Native/PTY |

- 上表目录以 `Kodex/` 为根；`app/rpc/RpcSessionDraft.kt` 等简称指其现有 commonMain 包路径。
- 每个 worker 只修改自己的任务文件和分配范围，不修改父任务、兄弟记录、checklist 或他人的文件，不提交/推送、不创建新子代理。
- `app/rpc` 是共享物理目录，不是共享写权限：1B 只写 Draft 与 commonTest，1C 只写 SessionSettings。装配 `Application.kt` 只有波次 2 可写。
- 跨范围遗漏在自己的报告列出准确路径及原因，由主会话重新指派；不为了临时编译通过添加旧接口转发、可选双协议或测试专用生产开关。

### 6.2 冻结接口

```kotlin
// 1A: ordinary creation only; suggested Sessions retain explicit settings.
suspend fun GlobalRpc.createSession(cwd: Path): Int

// 1B: framework-free contract, same package as today's NewSessionViewModel.
interface NewSessionViewModel : AgentSettingsViewModel, AutoCloseable {
    val composer: ComposerViewModel
    val runtimeConfiguration: RuntimeConfigurationViewModel
    suspend fun submit(expectedRevision: Long): PersistedSessionViewModel?
}
data class NewSessionViewModelArguments(val initialWorkingDirectory: Path)
// Factory port stays create(arguments); implementation construction becomes:
DefaultNewSessionViewModelFactory(views, sessions, global: RpcGlobalSettings, scope)

// 1C: PersistedSession absorbs name/rename/settings/close; SessionViewModel deleted.
data class SettingsViewModelArguments(
    val target: PersistedSessionViewModel?,
    val initialPage: SettingsPage,
)
// app/rpc supplies this stateless source, usable by Application's assembly.
object UnavailableSessionSettingsDataSource : SessionSettingsDataSource
// state always Unavailable; updates/rename return false; close is a no-op.

// 2: selection type/validation as in section 2.3.
val ApplicationViewModel.newSession: NewSessionViewModel
suspend fun ApplicationViewModel.selectNewSession(): Unit
suspend fun ApplicationViewModel.submitNewSession(expectedRevision: Long): PersistedSessionViewModel?
// createApplicationViewModel retains its existing factory ports;
// newSessionArguments is one NewSessionViewModelArguments, not an ordinal lambda.

// 3A: existing NewSessionScreen/NewSessionContent callback becomes:
onSubmit: (expectedRevision: Long) -> Unit
fun SessionTabBarCallbacks.selectNewSession(): Unit
```

- 以上是签名级伪代码，不将 extension 写法照抄为新 API 层。其余既有参数/公开 state 不为此机械包装；新增所需依赖只进对应真实 owner 构造。
- `RpcGlobalSettings` 借用给单例，不由单例 close；四字段投影不再复制为可独立编辑的完整本地 settings。cwd/输入仍在前端。
- 提交 gate 在进入 Application 串行等待前拒绝进行中重复命令；revision 在真实 mutation 边界再次校验。child 的同一调用返回前完成创建、追加、resume admission 和准确草稿重置；父级只负责成功结果入列及选择。
- 已接受编排归存活的前端 owner，调用方/renderer 取消只结束等待；保留唯一错误出口，不能因无人等待丢失失败，也不能由 worker 自行添加第二套通知。实际实现选择在波次 2 静态验收时检查。
- 本次不改变普通 persisted Agent Composer 的 trim/空白语义；原文有效的要求针对 New Session 首次提交链路，不能顺带重做所有输入入口。

### 6.3 派发、回报与返工

- 新 Session 不继承本会话历史；派发 prompt 必须包含仓库根、父子任务绝对路径、工作范围、冻结接口和已确认产品约束。
- 首批只派发 1A/1B/1C；工具接受只表示 worker 已启动，不代表代码、测试或验收完成。返回的 Session ID 和 storage URI 记在本节。
- worker 在自己的 `Details` 下回报实际文件、实现决策、静态检查、编写的测试、未跑测试、缺陷/依赖和待父级检查项；将活动语句停在 `Await parent acceptance`，不自行移入 done。
- 主会话逐文件检查范围、接口和语义；每波合格后才派发下一波。若接口有必要改变，由主会话先修订此冻结表，再通知所有受影响任务，不能让兄弟代码默默适应新形状。
- 每轮返工记录问题、负责人、修复文件及复验结果；不以 worker 自报“完成”取代主会话验收，不因接口尚在分波迁移而删除正确断言。

### 6.4 主会话验收门禁

1. **G0 静态与范围**：保留既存工作树改动；检查 diff/未跟踪文件、旧 API 消费者、依赖方向、KDoc、序列化接线和新协议常量。旧多草稿/原位替换测试迁移为新断言，其他行为不得静默丢覆盖。
2. **G1 编译与基础单测**：串行运行 `:rpc-client:jvmTest`、`:rpc-server:jvmTest`、`:app-backend-daemon:jvmTest`、`:app-backend-host:jvmTest`、`:app-rpc:jvmTest`、`:app-session-contract:jvmTest`、`:app-session:jvmTest`、`:app-component-new-session-viewmodel:jvmTest`、`:app-component-session-settings-viewmodel:jvmTest`、`:app-component-settings-viewmodel:jvmTest`、`:app-application:jvmTest`。真实 task 名先据当前模块/任务表核实，不执行旧 `impl` 坐标。
3. **G2 Renderer 与装配**：运行 `:app-component-{new-session,session-tab-bar,session-settings,settings}-view:jvmTest`、`:app-view:jvmTest` 及 `:integration-test:jvmTest` 中本任务相关测试；必须覆盖第 4 节，不启用真实账号、线上模型、录屏脚本或默认禁用的 real-IO/perf 测试。
4. **G3 平台与交互**：验证协议客户端/服务端及 CLI 的适用 Native 编译；在独立 PTY/临时 Home 下使用 mock 或离线安全装配验收空 tabs、`+` 高亮、切换保留、原文发送、成功重置、Settings 无目标及窄终端。未能跑真实交互时明确标为缺口，不拿快照测试冒充。
5. **G4 基线与收尾**：已存在或疑似既有失败用本轮冻结基线对照；记录测试数、失败名、命令、JVM、报告位置和排除范围。必须区分本任务回归、已证实既有失败及尚未验证。释放本任务资源后才交付；未完成门禁不移 done。

- 重型执行由主会话统一调度，worker 禁止启动 Gradle、Native、IDE Sync、外部 API 或控制设备。主会话优先使用 remote MacBook 私有隔离副本，启动前确认 SSH、设备空闲、现有重型锁和 Gradle daemon JVM；不能复用可用 daemon 是构建阻塞，不能绕过锁或自行杀进程。
- 记录基线 manifest 后同步显式允许的源码改动；保留设备已有凭据/缓存，不复制/展示密钥，不操作生产 Home/正在使用的 Kodex。具体资源占用和本轮参数在验收开始时检查记录，不把旧任务观测当作当前事实。

### 6.5 当前交接状态

- 首批 Session 已由用户接受并异步启动：
  - 1A：Session 953，[后端默认值创建与协议](file:///home/stream/.kodex/sessions/953)。
  - 1B：Session 954，[可复用单例草稿与共享配置](file:///home/stream/.kodex/sessions/954)。
  - 1C：Session 955，[真实 Session 与 Settings 边界](file:///home/stream/.kodex/sessions/955)。
- 六个 worker 均已完成父级验收；第一波接口保持冻结设计，后续测试及 Native/PTY 的实际证据见 §6.7–6.9。各波“尚未执行”等说明保留其交付时点，不替代本节最终状态。
- 波次 2：Session 959 [Application 导航与生产集成](file:///home/stream/.kodex/sessions/959) 已交付。主会话完成审查并修复测试问题，六模块复跑 134 项通过，允许第三波消费当前接口。
- 波次 3 已由用户接受并完成交付：
  - E：[Session 961 — TUI 渲染与交互](file:///home/stream/.kodex/sessions/961)，独占根 View 与 NewSession/TabBar renderer。
  - F：[Session 962 — 跨模块回归与验收覆盖](file:///home/stream/.kodex/sessions/962)，独占 `integration-test`。
  - 两者已交付源码/静态检查，由父级审查及实际运行；worker 没有启动构建或控制设备。
- 主会话：G1 的 551 项与 G2 的 133 项均通过，共 **684 项，0 失败、0 跳过**，仅代表所列门禁，不代表全仓。相邻 Agent 组件另有 22 项通过、1 项冻结基线同样失败，不混入全绿计数。macOS Arm64 Native 与独立 PTY 均通过，G0–G4 完成；未提交/推送或发布。

### 6.6 第一波父级静态验收

- 已核查 A 的 cwd 参数/显式 serializer 依赖、后端创建边界 supplier、真实 composition root、默认标题与协议 `0.5.0`；建议批次/fork 不读新建默认值。新 descriptor/JSON/compatibility 测试尚未编译或执行。
- 已核查 B 的原文长度判断、tryLock admission、revision 与后续输入/cwd 保护、共享 defaults CAS、可重复创建、借用资源和 fixture 收尾；child 仍直接挂起，整个 accepted-work/导航/唯一异常出口必须由 D 接管，不能因 B 的直接调用测试替代实际 Application 验收。
- 已核查 C 的接口合并/删除、nullable Settings、静态 unavailable source 与原 persisted CAS/生命周期保留；renderer 原有 unavailable 分支未被改写为可编辑状态。
- 已审阅实际变更范围与新增文件，`git diff --check` 通过；派发前 `gradle.properties` SHA-256 验证一致。未看到旧接口兼容转发或额外恢复机制。
- 主会话接手并补齐：
  - `BackendHostTest.kt` 两处创建、共享 fixture 的后端默认模型；`BackendReconnectTest.kt` 一处创建。
  - `ComposerHostRepairTest.kt` fixture 的后端默认模型与一处创建。
  - AgentSettingsViewModel 的 New Session KDoc，以及 backend-daemon checklist 中 Home `0.4.15` / business `0.5.0` / management `1` 的区别。
- 上述是子任务写范围之外的集成遗漏，不回滚他们正确保留的边界。D/E/F 内的旧消费者按原波次迁移；当前全仓尚不可宣称可编译。
- G1 可先验证不依赖 Application 迁移的 RPC 子集；G1 其余、G2–G4 待后续集成。所有失败必须保留命令/报告并区分源码缺陷与环境阻塞。

### 6.7 基础 RPC 验证执行记录

- 已冻结本轮 G1 输入 manifest/源码包，位于 [本地证据目录](file:///home/stream/ACodeSpace/demo/kodex-new-session-948/)；远端自有副本 `/Users/stream/ACodeSpace/local/kodex-new-session-948/candidate`。该轮仅选择 `:rpc-client:jvmTest :rpc-server:jvmTest`，不编译仍在迁移的 Application/View 消费者。
- remote MacBook 正式预检时无 Gradle/Kotlin daemon、swap 为 0；使用已安装 JBR 25.0.4。APFS clone 旧实验的 dependency/wrapper/native 缓存到本任务私有 Gradle Home，没有复制凭据；只读复用既有 secret-free package-access init。
- 持有既有 `device-heavy.lock` 的本任务 guard 才启动构建；显式 JAVA_HOME / org.gradle.java.home，single-use Gradle、4 GiB heap、单 worker、禁并行，Kotlin in-process 避免留下独立编译 daemon。测试 action 禁止 up-to-date/build-cache，编译缓存保留。
- guard 的 30 分钟总预算、内存/512 MiB swap 守卫和自有进程组清理见 [run-gate.pl](file:///home/stream/ACodeSpace/demo/kodex-new-session-948/run-gate.pl#L1)。停止只针对本轮进程，不抢占他人 daemon 或设备。
- 第一轮 exec `78174812` 已退出 0，90 秒，Gradle 295 tasks executed；`rpc-client` 151 项、`rpc-server` 142 项，共 **293 项，0 失败、0 跳过**。已核实新增 descriptor/Path serializer 和后端默认值测试在报告内，不只是编译成功。
- 日志、status 和 XML 已收集到 [G1 RPC 证据](file:///home/stream/ACodeSpace/demo/kodex-new-session-948/results/)。结束后检查共享锁无持有者，Gradle/Kotlin daemon 均无残留，swap 使用仍为 0。
- 第二轮沿用同一已冻结候选输入，选取 `:app-rpc:jvmTest :app-backend-daemon:jvmTest`，exec `658771016` 已退出 0，42 秒。`app-rpc` 107 项、`app-backend-daemon` 17 项，共 **124 项，0 失败、0 跳过**；日志/XML 已收集到同一本地证据目录，名称 `g1-rpc-draft*`。
- 两轮共 **417 项，0 失败、0 跳过**。覆盖本轮 raw text/revision/重复提交/cwd 保护/共享 defaults/异常不重放等 RPC 边界用例与业务协议常量；不包括尚待 D 的实际 Application owner/导航、NewSession VM 组件、Settings 组件、backend-host 兼容连接、根 renderer、Native 或真实交互。默认禁用的 real-IO/perf/录屏项未启用，不将其计作通过。
- 第二轮结束后再次确认共享重型锁无持有者、Gradle/Kotlin daemon 无残留、swap 使用为 0。仅保留本任务私有源码/缓存与报告供下一波验证，无存活构建资源。

### 6.8 Application 父级验收与返工

- 已审查 D 的生产装配、全部契约变化、提交 gate/owner/唯一异常出口、选择保持、nullable Settings/Login、cwd popup owner 捕获及取消后 join 顺序；与冻结接口一致。生产 NewSession child 仍由 D 的 command owner 编排，未使用 NonCancellable 保活正常提交。
- 本轮冻结 `g1-application-source*`，在同一私有 MacBook 副本使用相同 JBR/锁/单 worker/测试强制执行规则，运行：
  - `:app-application:jvmTest`
  - `:app-component-new-session-viewmodel:jvmTest`
  - `:app-component-session-settings-viewmodel:jvmTest`
  - `:app-component-settings-viewmodel:jvmTest`
  - `:app-session:jvmTest`
  - `:app-backend-host:jvmTest`
- 首跑 `g1-application` 退出 1（36 秒）：D 新测试四处把 `OpenAiLoginViewModel.isActive(attemptId)` 方法当布尔属性，导致 Application test compilation 失败；父级上轮修改的两个 backend-host 断言错误地要求目录首个模型等于 fixture 默认模型，产生两项断言失败。这些是已定位的测试问题，不能计作测试通过或未经证实归因于生产缺陷。
- 主会话修复仅涉及两个测试文件：
  - `ApplicationNewSessionTest.kt` 使用公开的 effects Flow，验证无 OAuth 的 Login effect channel 在关闭无关 tab 时仍开放、dismiss/shutdown 后终止；不删除生命周期断言，不新增生产测试开关。
  - `BackendHostTest.kt` 在 cwd-only 创建前以 CAS 明确写入当次选中的后端模型，不再假设模型目录首项与 fixture 默认值相等。
- 保留首次失败日志/XML、原始输入及重试差异。`g1-application-retry1` 退出 0（38 秒），六模块报告分别为 **67 / 5 / 17 / 15 / 13 / 17**，共 **134 项，0 失败、0 跳过**。报告含真实 Application 提交/取消、孤立异常、shutdown、配置双向同步、无目标 Settings/Login、Session binding 及旧业务协议拒绝用例。
- 结合先前四模块 417 项，当前 10 个模块共 **551 项通过**；不是把失败首跑或复跑重复计数。日志/status/XML 与输入 manifest 位于 [本轮证据](file:///home/stream/ACodeSpace/demo/kodex-new-session-948/results/)，文件前缀 `g1-application*`。
- 已确认本轮锁和自有 Gradle/Kotlin daemon 释放、swap 为 0，原有 `gradle.properties` SHA-256 保持。D 允许进入 E/F 集成，尚不代表 Renderer/Native/整仓验收通过。
- 本波文案经 bounded agy 辅助审阅，采用编译/断言和门禁范围措辞建议；§6.2 冻结接口保持。原始输出见 `results/agy-d-document-review.json`。

### 6.9 TUI、集成与基线父级验收

- 已审阅 E/F 的生产 diff、依赖与真实 fixture；旧 API 扫描为空，录屏源码迁移保留，仅编译、不执行。G2 仍使用 §6.7 的隔离副本、JBR、资源锁和内存守卫。
- 父级修正测试编排，保留原产品断言：
  - `SettingsLoginLifetimeTest.kt` 的异型 Throwable 列表比较改为数量和同一异常实例断言；真实文件 I/O 改用 RealTime compartment 与单线程 dispatcher，避免虚拟时间误超时及 Compose 并发。
  - 发送事件后等待业务结果时持续推进 TestMosaic 帧；点击切页后先等待新主体呈现，再触发其 Settings。`NewSessionIntegrationTest.kt` 的三个 gate 等待同样推进帧，不能只等待未被处理的输入事件。
- 实际产品问题：点击 `+` 后原顶栏目标仍存活，Mosaic 的 autoFocus 不会主动抢占它，第二次 Enter 没有进入输入框。`NewSessionContent` 挂载时请求其输入目标焦点，只在尚未绑定时等一帧，不在普通重组重复抢焦点。真实根界面第二次创建及 cursor 保留用例验证此修复。
- `g2-components`、`g2-root*`、`g2-focused-retry1`、`g2-retry` 保留首次错误与复验 XML/日志。初次完整 integration 因已识别的无帧 gate 等待而由父级终止（exit 125，182 秒），不计作完整测试结果。
- 最终 `g2-final` 退出 0（52 秒），真实发现/执行 **133 项，0 失败、0 跳过**：
  - new-session/view 8；session-tab-bar/view 14；session-settings/view 7；settings/view 18。
  - app/view 的 11 个安全 suite 共 55；integration 的 ApplicationRootBoundary 8＋NewSessionIntegration 23。
  - session-contract 的 jvmTest 为 NO-SOURCE，不伪造测试数量。未选择 Docs recording、线上模型、realIO/perf suite；精确 task/filter 可从同名日志与 guard 参数还原。
- 相邻 Agent 组件 `g2-components` 实际 23 项中 22 通过；`real running host restores original steer hint and Enter never duplicates pending preview` 在等待 pending steer 数量时 40 秒超时。冻结原始源码的独立 `baseline-validation` 只选该用例，同样 40 秒、相同等待点失败（`baseline-steer` exit 1，69 秒）。已证实为既有失败，不删除/跳过它，也不越界重做 persisted Composer。
- 本节输入 manifest、原始/修正源码包及 XML 位于 [验收证据](file:///home/stream/ACodeSpace/demo/kodex-new-session-948/)；最终两处额外修改 hash 见 `g2-final-sha256.txt`。原有 gradle.properties hash 不变。
- G3 的 macOS Arm64 RPC client/server 编译和 CLI debug executable 链接已通过（`g3-native-retry1`，exit 0，81 秒）。首跑 3 秒失败来自辅助 Gradle init 被应用到 build-logic 时误查产品 project，修复只涉及实验脚本的 project 存在性判断。
- Native 产物为 Mach-O arm64，SHA-256 `6efff47782eef00127b8e76418154ae6a78c9e501e10448390868f512be5f2a1`；不是四平台构建/发布结论。
- 独立 PTY 实际运行真实 Native frontend，连接临时 Home 内真实本地 TCP/kRPC/JVM backend，只有外部模型/Login 显式 mock。使用已有 runBackendHost factory seam，辅助启动器仅在隔离副本的测试源码与自有 init 中，不引入生产开关，不接触用户 Home、账号或现有 backend。
- 最终 PTY `1792362631-484062`：**22 项检查全部通过**，20.3 秒、SSH exit 0。涵盖空启动/零长度不创建、带空白首条发送、两次有效创建、`+` 互斥高亮、切页文本和物理光标保留、cwd picker/保留/成功重置、Settings 默认 New session 及 Current session unavailable、32/8 列缩放与恢复、前后端退出。原文逐字比对另由真实集成的三种空白用例证明，不把终端画面当存储字节检查。
- 早期 PTY receipt/frame/transcript 全部保留。仅修正辅助脚本的 settings.yml 扩展名、路径省略显示期待、cursor 错误捕获点，以及关闭 Settings 后未重新聚焦输入框的事件编排。
- 一次 UI 检查已完成但 JVM 启动器自然退出等待失败（exit 143），未算正常收尾。最终启动器仅在 runBackendHost 返回且 discover 确认 instance 锁释放后显式退出；日志包含 `KODEX_VALIDATION_HOST_RETURNED_AND_RELEASED`，backend stop accepted、进程 exit 0。未修改生产 backend。

### 6.10 最终证据与收尾

- [本地证据根](file:///home/stream/ACodeSpace/demo/kodex-new-session-948/) 保留基线与最终源码包/manifest、所有失败/通过日志/XML、PTY receipts/frames/raw transcript、辅助启动源码/hash，以及约 25 MiB 的 Native 压缩包。最终 manifest 的 1,306 个文件在远端逐项校验通过，见 `results/final-remote/results/final-source-verification.log`。
- 已收回远端日志及产物，再在重型锁下移除整个自有 `/Users/stream/ACodeSpace/local/kodex-new-session-948`，包括候选副本、基线副本、私有 Gradle/Native cache 和辅助入口。所有临时 PTY Home 已删除，前后端/JVM/daemon 与共享锁均无残留，swap 为 0；见 `results/final-resource-cleanup.log`。不删除其他任务目录或共享锁文件。
- 既存 gradle.properties SHA-256 保持；无提交、推送、安装替换或 release。没有执行整仓所有测试、Docs recording/线上模型或其他 Native 平台，不把相邻 Agent 的已证实基线失败称为通过。
- 文档已完成删减、意图对照与全文一致性复查，并用 bounded agy 辅助审阅；采用交付状态和全绿范围措辞修正。最终无阻塞性文档问题，输出见 `results/agy-closeout-document-review.json`。
- 父任务及六个 scoped 子任务移入 done，更新入链；后续若处理既有 steer 超时，应另获该范围授权，不在本任务继续扩展。

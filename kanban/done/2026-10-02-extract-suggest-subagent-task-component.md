# Task Tree

- `Review Suggest Subagent Task ports and side-effect sequence`()
- `Move suggestion state, configuration, commands, and factory into spec`()
- `Extract editing, submission, and complete configuration renderer`()
- `Hand host and Working Directory integration to the batch integrator`()
- `Write accept, reject, partial failure, and obsolete callback tests`()
- `Run centralized component and host builds`()
- `Record batch acceptance and commit`()

# Details

## Scope and source map

- 属于 [四组件双线批次](2026-10-02-extract-pending-tool-and-resource-components.md)，
  工具线两个组件 implementation、集中构建及用户验收已完成。
- 本批提交为 `7310a445`，进入 done；工具线 Session 未运行构建，由主 Session 集中验证。
- 目标：`app/component/suggest-subagent-task/{spec,impl/viewmodel,impl/view}`。
- 从 `app/contract/agent/.../AgentViewModel.kt` 移走
  SuggestedSessionConfiguration、state/result、SuggestSubagentTaskViewModel；
  从 `app/viewmodel/agent/.../SuggestSubagentTaskViewModel.kt` 抽出实现。
- 从 `AgentRuntimeView.kt` 移出任务面板；从 Application
  `AgentRuntimeScreen.kt` 和 `SessionTreeCliScreen.kt` 收拢建议配置专有交互。
  仅移动 suggestion 专用部分，共用 RuntimeConfiguration/全局菜单保持宿主职责。
- 覆盖接受、拒绝/反馈、模型/推理强度/service tier/request-user-input mode/cwd，
  不把 configurationContent 留成没有交互契约的任意黑盒。

## Implemented spec and host boundaries

- `SuggestSubagentTaskDependencies` 提供绑定源 Agent 的 pending Flow、
  当前默认配置读取、模型选项只读源、批量创建 Sessions、完成原工具事件、
  恢复源 Agent 的 capability。工厂另收 owner scope。
- 默认配置在新调用进入时采样，用户编辑草稿由组件持有；不是创建时再读取当前
  选中 tab。模型目录可继续更新，规范未在目录中的既有配置值如何保留展示。
- 宿主 adapter 将配置投影到已有 initial settings，并调用原批量创建入口；
  `onCreated`、Session 打开/登记和应用导航留宿主。不传 ApplicationViewModel
  给组件，也不在组件内创建 Session repository。
- cwd 使用 Working Directory 组件的声明过的选择契约；打开/dismiss popup 的
  应用路由仍由宿主负责，端口绑定 exact callId，提交中/过期选择被拒绝。
  不将此能力绑定到源 Session cwd，也不新增组件间 impl 依赖。
- spec 说明 Idle/Pending、configuration、feedback、revision、submitting，
  edit/submit 的 Stale/Busy/Failed/Submitted 与 `@throws` cancellation。
  反馈原字符串语义不擅自修剪；现实现只将非空拒绝反馈写入结果。

## Interaction and side effects

- 草稿编辑以 exact callId 准入；submit 再验证 expectedRevision，
  原子锁定配置/反馈快照。接受：创建 Sessions → 完成 Accepted 工具结果 →
  resume；拒绝：完成 Rejected 工具结果 → resume，不创建 Sessions。
- 不能把工具完成、Session 创建和 resume 宣称为一笔事务。创建成功后完成失败、
  回执丢失、完成后恢复失败必须分别记录；不自动重试创建或删除已创建 Session。
- 既有失败返回恢复可编辑状态，renderer 的呈现按当前行为基线测试；
  不借拆分新增产品错误提示或成功确认。取消和调用替换按父任务门槛复核。
- scope/close 遵守源 Agent 所有权；关闭交互组件不能关闭已创建 Sessions，
  不能把取消等待解释为取消后端已接受批量创建。
- 接受确认、拒绝编辑模式和反馈草稿是业务交互状态，spec 必须表达；
  renderer 只保留滚动/焦点，不保留另一份拒绝模式 authority。

## Checks

- 迁移 Feedback/Panel/建议配置测试，并增加 fake-port 事件顺序断言；
  无配置差异、全部配置字段、拒绝空/非空反馈、同 call 编辑、
  revision 变化、提交中重复操作、替换调用、owner close 和取消均覆盖。
- 对“创建已发生但完成失败”测试已有行为和副作用边界，不能用全成功 mock 代替。
  若要引入缓存结果/幂等 key/自动补偿，必须另行评审，不在迁移中默加。
- Working Directory 的所有建议调用门禁与最新配置合并测试继续通过；
  真实 RPC 批量创建/完成顺序和 Application onCreated/navigation 不回归。

## Integration handoff checkpoint — implementation complete

- **完整组件 implementation、配置 renderer、拒绝模式及测试完成；
  集中构建与运行测试待办**。不等待主 Session 编译后才交接。
- 未运行 Gradle/操作共享 IDE/提交/推送/切换分支；保留 Working Directory、
  设置工作线及宿主文件的各自 owner 修改。
- 三个真实项目沿 convention 继承 targets/source sets：
  `:app-component-suggest-subagent-task-spec` (`kodex.kmp-cli`)、
  `:app-component-suggest-subagent-task-impl-viewmodel` (`kodex.kmp-viewmodel`)、
  `:app-component-suggest-subagent-task-impl-view` (`kodex.kmp-view`)。
  spec 直接依赖 clean-models、multi-agent tool spec、OpenAI models、coroutines/io；
  无 Agent contract、Application、RPC impl 或 repository 依赖。

### Exact assembly API

- 保持 `io.github.stream29.kodex.app.agent.contract.` 前缀：
  `SuggestedSessionConfiguration`、`SuggestSubagentTaskState`、
  `SuggestSubagentTaskSubmissionResult`、`SuggestSubagentTaskViewModel`。
  仅移走原 AgentViewModel.kt 的 suggestion declarations/专属 imports；
  其他父级声明保留，没有双份类型或 factory compatibility wrapper。
- 同包新增 `SuggestSubagentTaskDependencies`：
  - `val pending: Flow<PendingSuggestSubagentTaskToolEvent?>`
  - `val models: StateFlow<List<ModelInfo>>`
  - `fun defaultConfiguration(): SuggestedSessionConfiguration`
  - `suspend fun createSessions(arguments: SuggestSubagentTaskArgs, configuration: SuggestedSessionConfiguration): List<SuggestedSessionMeta>`
  - `suspend fun completeToolCall(completed: StableCleanEvent.CompletedTool): Int`
  - `fun resumeRuntime(): Unit`
- 同包新增 `SuggestSubagentTaskViewModelFactory`：
  `fun create(dependencies: SuggestSubagentTaskDependencies, ownerScope: CoroutineScope): SuggestSubagentTaskViewModel`。
- 实现原函数 FQCN 保持，**只提供新签名**：
  `io.github.stream29.kodex.cli.agent.createSuggestSubagentTaskViewModel(dependencies: SuggestSubagentTaskDependencies, ownerScope: CoroutineScope): SuggestSubagentTaskViewModel`；
  同包 `suggestSubagentTaskViewModelFactory: SuggestSubagentTaskViewModelFactory`。
- ViewModel 新增 `val models: StateFlow<List<ModelInfo>>`（只读 catalog handle）；
  原 state/updateFeedback/updateConfiguration/submit/close 保持，新增：
  - `setRejecting(callId: String, rejecting: Boolean): Boolean`
  - `updateModelConfiguration(callId: String, model: OpenAiModelId, reasoningEffort: ReasoningEffort, serviceTier: ServiceTier): Boolean`
  - `updateRequestUserInputMode(callId: String, mode: RequestUserInputMode): Boolean`
  - `updateWorkingDirectory(callId: String, directory: Path): Boolean`
  - 原 `submit(callId: String, expectedRevision: Long, accepted: Boolean): SuggestSubagentTaskSubmissionResult` 不变。
- 字段命令在最新配置上只改对应字段，全量 updateConfiguration 仍为真实 CAS 编辑。
  Working Directory 的 host selection port 可调用 updateWorkingDirectory；
  popup 路由、child owner、onCreated/navigation 始终留宿主。

### Complete renderer API — no configurationContent slot

全部位于保留的 `io.github.stream29.kodex.cli.agent` 包：

- `SuggestSubagentTaskDropdowns.remember(owner: Any?, callId: String?): SuggestSubagentTaskDropdowns`，
  仅 renderer-only menu/anchor handle，无业务 state。
- `SuggestSubagentTaskPanel(viewModel: SuggestSubagentTaskViewModel, state: SuggestSubagentTaskState.Pending, columns: Int, rows: Int, dropdowns: SuggestSubagentTaskDropdowns, onBrowseWorkingDirectory: (String) -> Unit)`。
  Panel 自己绘制 model/effort/tier、question mode、cwd、决定与反馈控件。
- `BoxScope.SuggestSubagentTaskConfigurationMenus(viewModel: SuggestSubagentTaskViewModel, state: SuggestSubagentTaskState.Pending, dropdowns: SuggestSubagentTaskDropdowns)`。
  宿主在 TuiPopupHost 内、persistent content 之后调用，与 Panel 共享 exact owner/call
  handle。菜单自己观察 viewModel.models，选择直接调用 spec 字段命令。
- 主 Session 删除宿主旧 suggestion 专用 trigger/配置副本更新 glue，
  更新 AgentRuntimeScreen/SessionTreeCliScreen/host Gradle。
  通用 RuntimeConfiguration 与其他 Agent/NewSession 菜单不迁入本组件。

### Defaults and preserved side effects

- 新 call 采样一次 defaults；`feedback = ""`、`revision = 0`、
  `submitting = false`、新增 `rejecting = false`；同 callId 完全去重。
- Reject 只进入组件拒绝编辑模式，不完成工具；Accept 本身即用户确认。
  相同 rejection mode 为 true/no-op，改变 mode 增加 revision；
  其他 admitted edit 沿原规则增加 revision，包括相同配置/反馈。
- 编辑在 closed/Idle/stale-call/submitting 时 false；
  submit 检查 call/revision → Busy → CAS；保持 unchecked Long revision 增量。
- 接受 create → complete Accepted → resume；拒绝 complete Rejected → resume。
  feedback 仅拒绝且非 blank 时传入，保留原字符串不 trim。
  Session 登记/打开/onCreated/navigation 位于依赖适配器。
- close 取消 observer 并 Idle，不取消 owner/已接受提交/已创建 Sessions；
  caller 等待取消不撤销已接受工作。
- 保留且测试 creation 成功、completion 失败后新 Sessions 仍存在，
  显式 retry 可以再创建一批；没有自动 retry、补偿或 exactly-once。
  迟到结果不覆盖 replacement state，成功仍清除内部 pendingEvent 并 resume；
  resume failure 时已是 Idle；取消保留 Submitting 直到投影替换/close。
- catalog 缺失的配置仍显示、不自动改写；菜单保留当前 model，
  effort 使用 catalog 或当前 effort，tier 使用 catalog/default。

### Tests and validation checkpoint

- `suggestSubagentTaskFeedbackTest` 移入组件并改为 dependency-only fake ports。
  原 repository/runtime delegate 仅捕获完成事件，已移除；
  不需要组件依赖宿主 fixture 或公开 internal impl。
- 覆盖 accept/reject、空/blank/原始反馈、全部配置字段、rejecting mode、
  same-call/default/catalog、stale/Busy、freeze、create failure、
  creation 成功后 complete failure/显式 retry、两阶段 cancellation、
  caller cancel/close 排空、owner 终止、replacement 迟到结果、resume failure。
- `suggestSubagentTaskPanelTest` 迁入 mosaicTest，fake 增加新命令，
  拒绝模式读取组件 state；保留长任务滚动、Accept、Reject、空反馈、键入测试。
- 新增 `suggestSubagentTaskConfigurationTest`：全字段标签/catalog fallback、
  cwd call identity、question-mode 键盘菜单、model/effort/tier submenu、
  Submitting 关闭菜单。共享纯 renderer helper 由两组件 renderer 回归覆盖。
- 宿主 SuggestedConfigurationTest、Working Directory adapter/ownership fixtures、
  DocsRecording、RPC 测试适配由主 Session 处理；本 Session 未编辑它们。
- 静态：内外层 git diff --check 通过；spec 无 impl/Agent contract 边，
  原拥有路径无重复声明。
- 集中验证已完成：组件 ViewModel 18 项、View 8 项 JVM 测试通过；
  配置菜单异步 dismissal 断言改为等待其实际 effect，不修改业务行为。
  docs fixtures/宿主/Working Directory 回归通过，批次合计 392 项；
  Native/JS 编译、CLI smoke 与 docs 录制导出未运行，本批已验收并提交为 `7310a445`。
- 事实来源：
  [spec](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/suggest-subagent-task/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/agent/contract/SuggestSubagentTaskViewModel.kt#L1)、
  [factory/implementation](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/suggest-subagent-task/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/cli/agent/SuggestSubagentTaskViewModel.kt#L1)、
  [complete renderer](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/suggest-subagent-task/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/agent/SuggestSubagentTaskPanel.kt#L1)、
  [dependency tests](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/suggest-subagent-task/impl/viewmodel/src/commonTest/kotlin/io/github/stream29/kodex/cli/agent/SuggestSubagentTaskFeedbackTest.kt#L1)。

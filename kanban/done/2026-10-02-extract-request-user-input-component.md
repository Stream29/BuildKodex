# Task Tree

- `Review Request User Input ports and lifecycle contract`()
- `Move state, drafts, commands, and factory into component spec`()
- `Extract dependency-only ViewModel and rendering implementation`()
- `Hand host integration changes to the batch integrator`()
- `Write dependency-only regressions and review side-effect boundaries`()
- `Run centralized component and host builds`()
- `Record batch acceptance and commit`()

# Details

## Scope and source map

- 子任务属于 [四组件双线批次](2026-10-02-extract-pending-tool-and-resource-components.md)，
  工具线 Session 457 implementation、集中构建及用户验收已完成；
  本批随 `7310a445` 提交，进入 done。
- 目标：`app/component/request-user-input/{spec,impl/viewmodel,impl/view}`。
- 原契约：`app/contract/agent/.../RequestUserInputViewModel.kt`。
  移入 draft answer、submission state/result、Idle/Pending、ViewModel 和纯验证规则；
  维持现有 Kotlin 包名，新增 dependency/factory 的 KDoc 与接口。
- 原实现：`app/viewmodel/agent/.../RequestUserInputViewModel.kt`；
  原 View：`app/view/agent/.../AgentRuntimeView.kt` 的问答面板及专有 helper。
  `AgentRuntimeStatus` 留原模块；获明确授权的跨工具纯 renderer helper
  由工具线放入现有 `app/view/components`。
- 覆盖 AgentRuntimeScreen 的实际入口及 docs/test consumers。History 的已完成答案
  继续用原 stable 数据模型，不重构整个 History，只处理移动声明后的引用。

## Implemented spec

- `RequestUserInputDependencies` 提供绑定当前 Agent 的 pending Flow、
  完成原工具结果的 suspend capability、恢复同一 Agent 的 capability。
  保留原完成 index 的语义；不暴露 runtime、repository 或 GlobalRpc。
- 工厂显式接收 dependencies 与 owner scope，创建独立 child。Agent 只发布准确
  child；renderer 不再次订阅 runtime state，也不镜像 answers。
- command 保持 `callId`、`questionId` 与提交 `expectedRevision`。
  选项必须属于当前问题；有选项时 Other 可用，无选项时直接自由输入，
  非空校验和 `user_note:` 协议投影沿现行实现。
- state 完整描述 Idle 与 Pending；Pending 暴露问题顺序、全部答案、revision、
  Editing/Submitting/Failed 和 canSubmit。Failed 留草稿，明确重试与编辑清错行为。
- KDoc 逐命令声明 false/结果联合的准入顺序；无效事件不是异常。
  写明 revision 耗尽等已暴露异常与 cancellation 的 `@throws`，不泛化为
  “任何异常都抛出”而忽略已有 Failed 返回。

## Submission and ownership

- 先校验 exact call/revision 与完整答案，再原子进入 Submitting；
  完成原 StableRequestUserInputToolEvent 后才请求 resume，不新增后端自动 resume。
- 固定本次答案快照；Pending 更换或关闭后，迟到前端结果不能覆盖新调用状态。
  同 callId 的既有去重策略与 Flow 生命周期写清，不能借迁移改为新队列。
- owner scope 持有提交；取消 renderer 等待与终止 owner 是不同事件。
  close 幂等，释放组件自有观察并使后续编辑失效，不关闭 Agent 或清理后端 pending。
- 面板不显示/切换 tab 不调用 child close；草稿及已接受提交仍属于原 Agent，
  只有明确组件 disposal/Agent 生命周期结束才走关闭契约。
- 取消/投影替换发生在副作用之前与之后分别测试；不承诺提交回滚。
  若实际 cancel/迟到状态行为不满足拟定安全契约，按父任务门槛单独评审。

## Renderer and checks

- Idle 不显示阻塞问答；Pending 按问题顺序渲染选项、Other 与输入。
  Submitting 禁用编辑/重复提交；Incomplete 保持编辑；
  Failed 呈现原错误分支并保留草稿；保留 questionId、callId 绑定。
- 保留选项到下一问题/提交的焦点迁移、自由输入提交键、滚动和窄窗口行为。
  renderer 不从历史/通知恢复草稿，不在 composition 自动 submit。
- 移入现有 ViewModel、Options renderer 测试；补全 stale call/revision、
  Busy/Incomplete、失败重试、owner close、取消、pending 替换与迟到完成。
- fake dependencies 验证完成事件答案、顺序与 resume 次数；RPC 集成验证真实完成
  事件 serializer 与准入。使用 in-memory Session 的集成测试可保留在宿主，
  不使生产组件依赖持久化实现。

## Integration handoff checkpoint — implementation complete

- **组件 implementation、renderer 与迁入/新增测试完成；集中构建与运行测试待办**。
  本 Session 未运行 Gradle、使用共享 IDE、提交、推送或切换分支。
- 保持内层 `refactor/spec`；未改写 Working Directory 原未提交改动或其他工作线文件。
  宿主接线、宿主 Gradle、统一构建与父任务文档仍由主 Session 负责。
- 三个真实项目沿 convention 继承 targets/source sets：
  `:app-component-request-user-input-spec` (`kodex.kmp-cli`)、
  `:app-component-request-user-input-impl-viewmodel` (`kodex.kmp-viewmodel`)、
  `:app-component-request-user-input-impl-view` (`kodex.kmp-view`)。
  spec 直接依赖 clean-models、request-user-input tool spec、coroutines；
  ViewModel 只依赖组件 spec，renderer 只依赖组件 spec 与通用 renderer 库。

### Exact assembly API

- 原声明保持 `io.github.stream29.kodex.app.agent.contract.` 前缀：
  `RequestUserInputDraftAnswer`、`RequestUserInputSubmissionState`、
  `RequestUserInputState`、`RequestUserInputSubmissionResult`、
  `RequestUserInputViewModel`、`allowsOtherAnswer`。
  原文件已移走，没有 wrapper 或重复声明。
- 同包新增 `RequestUserInputDependencies`：
  - `val pending: Flow<PendingRequestUserInputToolEvent?>`
  - `suspend fun completeToolCall(completed: StableCleanEvent.CompletedTool): Int`
  - `fun resumeRuntime(): Unit`（同步请求原 continuation，不等待整个运行）
- 同包新增 `RequestUserInputViewModelFactory`：
  `fun create(dependencies: RequestUserInputDependencies, ownerScope: CoroutineScope): RequestUserInputViewModel`。
- 实现入口原 FQCN 保持，**只提供新签名**：
  `io.github.stream29.kodex.cli.agent.createRequestUserInputViewModel(dependencies: RequestUserInputDependencies, ownerScope: CoroutineScope): RequestUserInputViewModel`。
  同包 `requestUserInputViewModelFactory: RequestUserInputViewModelFactory` 供 typed assembly。
- renderer 原 FQCN/签名保持：
  `io.github.stream29.kodex.cli.agent.RequestUserInputPanel(viewModel: RequestUserInputViewModel, state: RequestUserInputState.Pending, columns: Int, rows: Int)`。
  `AgentRuntimeStatus` 留原 `app/view/agent`。
- 主 Session 为 Agent contract 添加 spec 依赖，为 assembly 添加 impl-viewmodel，
  为实际 renderer consumers 添加 impl-view；History DTO/wire serializer 不变。

### Defaults and preserved boundaries

- 默认 Idle；新 call 为 `answers = emptyMap()`、`revision = 0`、`submission = Editing`。
  同 callId 投影完全去重，不更新参数。
- 保持 option/Other/free-form 准入与 `user_note: ` + trimmed text。
  Submit 顺序：exact call → revision → Busy → completeness → CAS。
  编辑清除 Failed；相同 free-form Editing 文本不增加 revision。
- accepted complete → resume 属于 ownerScope；renderer 等待取消不撤销工作。
  close 幂等并取消组件 observer，不取消 owner/已接受提交或 backend pending。
- 保留并测试既有边界：cancellation 保留 Submitting；
  迟到结果不覆盖 replacement state，但成功仍清除内部 pendingEvent 并 resume
  （因此 replacement 可能返回 StaleCall）；resume 失败发生于 Idle 后；
  revision 耗尽抛异常；blank failure message 无法构造 Failed，沿原异常传播。
  未增加业务修复、队列或 exactly-once。
- 共用 framework-only helpers 放入 `app/view/components/.../InteractionInput.kt`：
  `InteractionOption`、`InteractionFreeForm`、`InteractionText`。
  没有组件 state 类型或组件间 impl 依赖。

### Tests and validation checkpoint

- `requestUserInputViewModelTest` 移入组件 impl-viewmodel 并改为纯 fake-port 测试。
  原 in-memory repository 只提供无关 runtime capability，已去除，
  不再需要 host fixture 或公开 internal 实现。
- 保留 selection/replacement；补 invalid edit、Other、直接文本、完成事件/顺序、
  StaleCall/StaleRevision/Incomplete/Busy、失败编辑与显式 retry、取消、
  owner 终止、close observer 释放、替换迟到成功/失败、
  resume failure、revision exhaustion、blank failure、Idle。
- `requestUserInputOptionsTest` 已移入 impl-view 的 mosaicTest，
  原选项、Other、键盘/焦点测试保留，补 Failed/Submitting 展示。
- RPC serializer、DocsRecording 与其他宿主 fixtures/consumer adaptations
  留主 Session，本 Session 未编辑它们。
- 静态：内外层 `git diff --check` 通过；本组件 spec 无 Agent contract/renderer/impl
  依赖，原拥有路径无重复声明。
- 集中验证已完成：组件 ViewModel 15 项、View 6 项 JVM 测试通过；
  宿主/serializer/Working Directory 回归见批次主任务，共计 392 项通过。
  Native/JS 编译、CLI smoke 与 docs 录制导出未运行；本批已验收并提交为 `7310a445`。
- 事实来源：
  [spec](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/request-user-input/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/agent/contract/RequestUserInputViewModel.kt#L1)、
  [factory/implementation](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/request-user-input/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/cli/agent/RequestUserInputViewModel.kt#L1)、
  [dependency tests](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/request-user-input/impl/viewmodel/src/commonTest/kotlin/io/github/stream29/kodex/cli/agent/RequestUserInputViewModelTest.kt#L1)。

# Task Tree

- `Inventory quick-config triggers, nested menus, and Agent/Draft commands`()
- `Specify stable target ports and runtime capability projection`()
- `Review atomic selection and caller cancellation compatibility`()
- `Implement component spec, ViewModel, and complete quick-config renderer`()
- `Integrate both targets and status-bar layout without moving runtime controls`()
- `Validate capabilities, selection identity, atomic writes, and lifecycle`()
- `Record acceptance and archive component task`

# Details

- 实施/集中验证完成；本组件 19 项 JVM 测试通过，原子三字段及真实
  Agent/Draft、替换/关闭、窄布局和调用等待验证见主任务。用户已验收，
  内层代码随批次提交为 `721ac0fe`。

## Scope and source map

- 属于 [五组件批次](2026-10-03-extract-session-configuration-and-browsing-components.md)，
  用户已审阅并授权实现，当前 executable。
- 目标 `app/component/runtime-configuration/{spec,impl/viewmodel,impl/view}`。
- 源 RuntimeStatusBar 的 RuntimeConfiguration/Dropdowns、两种 trigger 布局、
  RuntimeConfigurationMenus，SessionTreeCliScreen 的 settingsOwner 接线。
  已迁入 [组件菜单及命令接线](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/runtime-configuration/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/runtimeconfiguration/RuntimeConfiguration.kt#L127)。
- 完整覆盖 materialized Agent 与虚拟 New Session Draft；不迁移 status bar
  的运行/停止/压缩、token、cwd button、Settings button 或整体排版。

## Proposed spec and dependencies

- 新建真实 RuntimeConfigurationViewModel、typed factory/dependencies；
  依赖只暴露准确 owner 的 configuration/models 和两种写能力：
  `updateModelConfiguration(model, effort, tier)`、`updateRequestUserInputMode(mode)`。
- domain 原 field commands 仍由 Agent/Draft infrastructure adapter 处理；
  新 UI child 不依赖完整 AgentViewModel/Session contract，也不保留一套全局 selected-owner 查询。
- state 给当前四字段、模型有序选项与各层 capability 选项/展示状态；
  借用 canonical OpenAI 类型，不把 settings 整份可写对象送给 renderer。
- Settings owner 父级发布稳定 child；Agent/Draft 切换更换准确 child，
  callback 必须绑定旧 owner/selection 而非解析当前 selected Session。
- UI dropdown、三级 submenu focus/anchor 和 popup position 留 renderer；
  spec 写明全部选择、能力 fallback、错误/取消与关闭语义。

## Baseline behavior and review gates

- modelOptions 为目录顺序加当前 model 去重。efforts 按所选 ModelInfo 支持列表；
  无能力时保留当前 effort。tiers 沿 availableServiceTiers，空时 Default。
- model → effort → tier 三级选择一次更新三个字段；不可拆成三个 CAS。
  Mode 独立更新；adapter 使用最新完整 settings 保留 cwd/thread 等无关字段。
- 不自动更正保存的 Custom effort/不可用 model/tier，不把 Settings 固定选项
  规则套到 Runtime，也不新增运行中禁止编辑的全局锁。
- 当前 renderer 的 coroutineScope 持有 suspend 调用等待，unmount 会取消等待。
  新 VM 命令应保留 caller cancellation；如提议改为 owner-scope 接受后继续执行，
  须单列行为差异复核，不借 factory 调整静默延长命令生命周期。
- close 只结束本组件观察/本地等待，不关闭借用 binding/Agent/Draft 或回滚
  后端已接受更新；菜单隐藏不等于 domain owner 关闭。
- 明确报告由现有 host/adapter 负责，不复制业务 exception 文本到 UI；
  有可观察 validation/cancellation 需在 spec 逐方法补 `@throws`。

## Checks and handoff

- fake deps：空/重复目录、当前缺失模型、各 effort/tier 能力与 fallback、
  四字段投影、原子一次更新、mode 独立更新、迟到旧 owner、取消/close。
- renderer：Agent/Draft 两入口、trigger label、三级键盘选择/Escape、
  selected focus、model catalog 更新、窄布局与状态栏其他 controls 共存。
- 保留 RuntimeStatusBarTest 中的快速配置/菜单测试及
  RpcSessionViewsTest/NewSession state 原子修改、无关字段保持和 materialization 回归。
- 整合负责人独占 RuntimeStatusBar/SessionTreeCliScreen、Agent/Draft
  parent contract/工厂/RPC adapters；组件线只拥有新三项目。
- 不为共用 selector 制造通用 configuration core；能共用的纯样式放现有公共 renderer。

## Integration handoff

- 组件线实现完成，静态检查完成；**中央编译/测试及宿主整合待执行**。
  本线仅新增 `Kodex/app/component/runtime-configuration/{spec,impl/viewmodel,impl/view}`
  与本节，未改旧 RuntimeStatusBar、Agent/Draft contract/adapter、共享测试、
  settings.gradle 或消费方依赖；未执行构建、提交、推送、分支或 IDE 操作。
- 新项目使用 `kodex.kmp-cli` / `kodex.kmp-viewmodel` / `kodex.kmp-view`；
  现有 module-tree discovery 会发现以下三个路径：
  `:app-component-runtime-configuration-spec`、
  `:app-component-runtime-configuration-impl-viewmodel`、
  `:app-component-runtime-configuration-impl-view`。
  spec 仅依赖 `:openai-spec-models` 与 coroutines；view 复用
  `:app-view-components` 的 Tui helpers、Theme、PopupMenuBackground，
  不依赖 `:app-view-application`。

### Exact public contract and factory

- 原 RuntimeConfiguration 是 internal；新公开声明统一归
  `io.github.stream29.kodex.app.runtimeconfiguration`，
  [规范 KDoc 与 API](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/runtime-configuration/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/runtimeconfiguration/RuntimeConfigurationViewModel.kt#L1)。
- 精确签名（全部 canonical OpenAI 类型，无 Agent/RPC/global store）：

```kotlin
data class RuntimeConfiguration(
    val model: OpenAiModelId,
    val reasoning: ReasoningEffort,
    val tier: ServiceTier,
    val requestUserInputMode: RequestUserInputMode,
)
data class RuntimeConfigurationModelOption(
    val model: OpenAiModelId,
    val efforts: List<ReasoningEffort>,
    val tiers: List<ServiceTier>,
)
data class RuntimeConfigurationState(
    val configuration: RuntimeConfiguration,
    val modelOptions: List<RuntimeConfigurationModelOption>,
    val closed: Boolean = false,
)
interface RuntimeConfigurationDependencies {
    val configuration: StateFlow<RuntimeConfiguration>
    val models: StateFlow<List<ModelInfo>>
    suspend fun updateModelConfiguration(model: OpenAiModelId, effort: ReasoningEffort, tier: ServiceTier): Unit
    suspend fun updateRequestUserInputMode(mode: RequestUserInputMode): Unit
}
interface RuntimeConfigurationViewModel : AutoCloseable {
    val state: StateFlow<RuntimeConfigurationState>
    suspend fun updateModelConfiguration(model: OpenAiModelId, effort: ReasoningEffort, tier: ServiceTier): Unit
    suspend fun updateRequestUserInputMode(mode: RequestUserInputMode): Unit
    override fun close(): Unit
}
fun interface RuntimeConfigurationViewModelFactory {
    fun create(dependencies: RuntimeConfigurationDependencies, ownerScope: CoroutineScope): RuntimeConfigurationViewModel
}
```

- [实现 factory](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/runtime-configuration/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/app/runtimeconfiguration/RuntimeConfigurationViewModel.kt#L21)：
  同 package 的 `DefaultRuntimeConfigurationViewModelFactory`，
  以及 `createRuntimeConfigurationViewModel(dependencies: RuntimeConfigurationDependencies,
  ownerScope: CoroutineScope): RuntimeConfigurationViewModel`。
  稳定 exact-owner child 只建一次；父 owner 关闭时关闭 child，不在 renderer frame 重建。
- `modelOptions` 是能力对象列表，不是原始 slug 列表：
  catalog slugs + 当前 model 按首次出现去重；同 slug 取第一份 metadata。
  efforts 保留 metadata 顺序，无 metadata/空列表则当前 effort；
  tiers 使用 `ModelInfo.availableServiceTiers()`，无 metadata 则 Default。
  原配置原样展示，不自动改 saved effort/model/tier，也不增加 running lock。
- suspend command 在 caller 的局部 coroutineScope 调用一次 port，
  无 owner-scope launch/持久提交队列/新重试策略。
  close 冻结 state、设 closed、取消本地观察和命令等待；closed 新命令 no-op。
  不关闭 borrowed port/binding/Agent/Draft/父 scope，不承诺回滚 backend 已接受更新。
  普通异常和 CancellationException 原样传播，报告留原宿主/adapter。

### Exact renderer and layout APIs

- Renderer package：`io.github.stream29.kodex.cli.runtimeconfiguration`。
  [完整 triggers、三级菜单与 label](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/runtime-configuration/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/runtimeconfiguration/RuntimeConfiguration.kt#L31)。
- `RuntimeConfigurationDropdowns.remember(owner: Any?): RuntimeConfigurationDropdowns`
  为 composable；传稳定 exact `RuntimeConfigurationViewModel`，Agent/Draft 切换更换 handles。
  两个公开 handles 为 `model: TuiDropdownState`、
  `requestUserInputMode: TuiDropdownState`。
- 下列均为 composable；两个 trigger API 都有
  `RuntimeConfigurationViewModel` 与 `RuntimeConfigurationState` 两个 overload，
  其余参数相同：

```kotlin
fun RuntimeConfigurationTriggers(
    viewModel: RuntimeConfigurationViewModel,
    dropdowns: RuntimeConfigurationDropdowns,
    enabled: Boolean = true,
): Unit
fun RuntimeConfigurationTriggers(
    state: RuntimeConfigurationState,
    dropdowns: RuntimeConfigurationDropdowns,
    enabled: Boolean = true,
): Unit
fun RuntimeConfigurationStatusItemsWithoutSpacing(
    viewModel: RuntimeConfigurationViewModel,
    dropdowns: RuntimeConfigurationDropdowns,
    enabled: Boolean = true,
): Unit
fun RuntimeConfigurationStatusItemsWithoutSpacing(
    state: RuntimeConfigurationState,
    dropdowns: RuntimeConfigurationDropdowns,
    enabled: Boolean = true,
): Unit
fun BoxScope.RuntimeConfigurationMenus(
    viewModel: RuntimeConfigurationViewModel,
    dropdowns: RuntimeConfigurationDropdowns,
): Unit
fun BoxScope.RuntimeConfigurationMenus(
    state: RuntimeConfigurationState,
    dropdowns: RuntimeConfigurationDropdowns,
    onConfigurationSelected: (OpenAiModelId, ReasoningEffort, ServiceTier) -> Unit,
    onRequestUserInputModeSelected: (RequestUserInputMode) -> Unit,
): Unit
```

- `Triggers` 是带一格间距的 Row；`StatusItemsWithoutSpacing`
  是**两个直接 measurables，无 Row/spacer**，可直接替换 Main status-bar
  `regularContent` 内两个配置控件。不要包 Row，否则原 wrapping/measurement 变一个控件。
- `BoxScope.RuntimeConfigurationMenus` 放在 `TuiPopupHost` 直接子级、persistent
  content 之后，使用与 triggers 同一份 handles；不放 status-bar 裁剪布局内部。
  VM overload 用 renderer `rememberCoroutineScope` 启动 caller-cancellable 等待；
  unmount 不关闭 child。state overload 供测试/精确 callback 接线。
- 菜单保留 model → effort → tier；只有 tier leaf 提交三字段一次，
  mode 单独提交。selected focus、Escape、fallback 和 enum 顺序沿基线。
  closed 不渲染 trigger 或菜单；disabled 通过 Tui trigger 关闭菜单，不成为运行锁。
- 精确公开 measurement helpers，**宿主直接导入，不复制逻辑**：
  `runtimeConfigurationLabel(model: OpenAiModelId, reasoning: ReasoningEffort,
  tier: ServiceTier): String`；
  `runtimeRequestUserInputModeLabel(mode: RequestUserInputMode): String`。
  tier 仅 Default 省略后缀；questions 文案为 `ask user` / `no question`。

### Main-owned adapter requirements and pending validation

- Agent/Draft 父级公开稳定 child；dependencies 必须绑定原 exact owner，
  `configuration.value` read-through 映射其 `settings.value` 的
  `model` / `reasoning.effort` / `serviceTier` / `requestUserInputMode`，
  collect 映射原 source，不建另一份 writable settings authority。
  `models` 借用其只读目录源。
- Port `updateModelConfiguration(model, effort, tier)` 直接 await 原
  exact-owner `AgentSettingsViewModel.updateModelConfiguration(model, effort, tier)`，
  不拆三次更新，不排入持久 owner/application queue。
  Mode 直接 await 原 exact-owner 单字段 command。
  Adapter 对最新完整 settings 做原更新/CAS，保留 cwd/thread 等字段；
  继续校验原 binding/revision，无全局 active-owner lookup 或迟到回调重定向。
- 本线新增
  [8 个 fake-port VM tests](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/runtime-configuration/impl/viewmodel/src/commonTest/kotlin/io/github/stream29/kodex/app/runtimeconfiguration/RuntimeConfigurationViewModelTest.kt#L28)、
  [11 个 fake-VM/真实 Mosaic menu tests](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/runtime-configuration/impl/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/runtimeconfiguration/RuntimeConfigurationTest.kt#L40)。
  覆盖四字段投影、空/重复/缺失目录、第一份能力、custom effort fallback、
  canonical tiers/未知 tier、atomic once/mode separate、无 optimistic correction、
  caller/close/owner cancellation、borrowed ownership、旧目标及原异常传播；
  真实三级键盘选择、selected focus/Escape、questions、缺失能力、
  两个 direct measurables/其他 status controls、窄宽度、disabled dismissal、
  catalog 更新、owner switch/closed 和 unmount cancellation。
- 集中编译/回归结果见主任务；Main 已迁移旧 RuntimeStatusBar tests 和全部 Agent/Draft
  host calls，检查 original layout measurements 使用共享 label；
  保留 RpcSessionViews/NewSession 原子三字段、无关字段保持、materialization、
  stale binding 与真实两入口生命周期回归。组件 fake tests 不替代宿主接线证明。

# Task Tree

- `Review title settings, fallback, menus, and generated-title owner`()
- `Review component spec and typed setting/model ports`()
- `Extract ViewModel and complete title controls/menus`()
- `Hand default-model and shared queue adapters to the integrator`()
- `Validate fallback, catalog changes, field conflicts, and disabled rendering`()
- `Record six-component batch acceptance`()

# Details

## Scope and source map

- 属于 [六组件批次](2026-10-02-extract-settings-configuration-and-account-components.md)，
  用户已验收通过，随六组件批次归档到 done；不改标题请求协议或后端生成行为。
- 目标：`app/component/session-title-settings/{spec,impl/viewmodel,impl/view}`。
- 原 SettingsPopup 的 SessionTitleSettingsContent/DropdownMenus；
  GlobalSettingsState 的 title/effective model/modelOptions 与三种标题编辑命令。
- SessionTitleSettings 留 `app/shared/settings/contract` 的原类型/serializer；
  `DefaultSessionTitleModel` 在 `app/shared/session-title` 混合实现里，不整体迁入组件。

## Implemented spec and dependency ports

- `SessionTitleSettingsViewModel`、typed dependencies/factory；只读 state 描述
  enabled、nullable configured model、effective model、ordered distinct options、
  reasoning effort、closed；命令覆盖三字段编辑及生命周期。
- deps 提供 settings/model sources、`defaultModel: OpenAiModelId`、
  application-queue admission 和 failure 端口；不提供标题生成器或整个 backend settings。
- adapter 注入现有默认常量；VM 内纯计算 effective/options。
  不复制默认字符串、不使 spec 或 VM 依赖 `app/shared/session-title` 的实现。
- KDoc 区分 model=null 的配置含义与非 null 显示值、目录变化与写入、
  独立 source 非原子性、admission/持久化、异常/取消与渲染 disabled 分支。

## Behavior and rendering baseline

- model=null 沿现有 compiled default；目录按原顺序，加当前 effective model 后去重，
  当前模型缺失仍可见；catalog 更新不改配置。
- reasoning 菜单沿现有 `knownReasoningEfforts`，不改成目录限制或自动换档。
  enabled=false 只禁用模型/推理 renderer controls，保留配置和解释文案。
- UI disabled 与旧 public command 是否拒绝不同：写方法原本仍可被程序调用。
  先写基线测试，再决定 spec 准入，不能凭 disabled 控件新增业务锁。
- 三字段分别捕获当前 baseline 入原共享 queue，合并无关字段；
  关闭 popup 排空已接受写入，不用渲染中的旧全文快照覆盖其他设置。
- 完整迁移控制与菜单，renderer 只持有 anchor/focus/scroll；
  前端标题显示、重命名和生成执行不属本组件。

## Checks

- fake ports：null/default、缺失模型、重复/空目录、目录更新、固定推理菜单、
  enable toggle、无关字段保留、queue admission/failure/cancellation/close。
- renderer：原 section 顺序、三个控件、禁用说明、键盘子菜单与关闭；
  迁入 GlobalSettingsGroupingTest 的标题用例，不复制整套宿主 fixture。
- RPC/后端标题测试继续验证 canonical default、原请求参数和生成生命周期；
  组件迁移不得触发额外标题生成或批量更名。

## Integration handoff

- Component implementation and centralized JVM validation are complete; user accepted the batch.
  No Gradle, commit, push, branch change, IDE operation or shared-source edit was performed.
- Owned tree: `Kodex/app/component/session-title-settings/`; convention projects:
  `:app-component-session-title-settings-spec`,
  `:app-component-session-title-settings-impl-viewmodel`,
  `:app-component-session-title-settings-impl-view`.
- [Authoritative spec/KDoc, lines 15–98](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-title-settings/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/sessiontitlesettings/SessionTitleSettingsViewModel.kt#L15).
  Contract and factory package: `io.github.stream29.kodex.app.sessiontitlesettings`.

### Exact public API

```kotlin
enum class SessionTitleWriteAdmission { Accepted, Rejected }
interface SessionTitleSettingsDependencies {
    val settings: StateFlow<SessionTitleSettings>
    val models: StateFlow<List<ModelInfo>>
    val defaultModel: OpenAiModelId
    val operationFailure: StateFlow<Boolean>
    fun setEnabled(expected: Boolean, enabled: Boolean): SessionTitleWriteAdmission
    fun setModel(expected: OpenAiModelId?, model: OpenAiModelId?): SessionTitleWriteAdmission
    fun setReasoningEffort(expected: ReasoningEffort, reasoningEffort: ReasoningEffort): SessionTitleWriteAdmission
    fun reportFailure(failure: Throwable): Unit
    fun dismissFailure(): Unit
}
data class SessionTitleSettingsState(
    val enabled: Boolean,
    val configuredModel: OpenAiModelId?,
    val effectiveModel: OpenAiModelId,
    val modelOptions: List<OpenAiModelId>,
    val reasoningEffort: ReasoningEffort,
    val operationFailure: Boolean = false,
    val closed: Boolean = false,
) {
    val reasoningOptions: List<ReasoningEffort>
}
interface SessionTitleSettingsViewModel : AutoCloseable {
    val state: StateFlow<SessionTitleSettingsState>
    fun setEnabled(enabled: Boolean): Unit
    fun setModel(model: OpenAiModelId?): Unit
    fun setReasoningEffort(reasoningEffort: ReasoningEffort): Unit
    fun hidePage(): Unit
    fun dismissFailure(): Unit
    override fun close(): Unit
}
fun interface SessionTitleSettingsViewModelFactory {
    fun create(dependencies: SessionTitleSettingsDependencies, ownerScope: CoroutineScope): SessionTitleSettingsViewModel
}
object DefaultSessionTitleSettingsViewModelFactory : SessionTitleSettingsViewModelFactory
fun createSessionTitleSettingsViewModel(
    dependencies: SessionTitleSettingsDependencies,
    ownerScope: CoroutineScope,
): SessionTitleSettingsViewModel
```

- Mosaic package `io.github.stream29.kodex.cli.settings`; remember/root/panel/menu functions are
  `@Composable`, and dropdown handles are renderer-only:

```kotlin
class SessionTitleSettingsDropdowns(val model: TuiDropdownState, val reasoning: TuiDropdownState)
fun rememberSessionTitleSettingsDropdowns(): SessionTitleSettingsDropdowns
fun BoxScope.SessionTitleSettingsComponent(viewModel: SessionTitleSettingsViewModel, showOperationFailure: Boolean = true)
fun SessionTitleSettingsPanel(viewModel: SessionTitleSettingsViewModel, dropdowns: SessionTitleSettingsDropdowns, showOperationFailure: Boolean = true)
fun BoxScope.SessionTitleSettingsDropdownMenus(viewModel: SessionTitleSettingsViewModel, dropdowns: SessionTitleSettingsDropdowns)
```

- [Renderer, lines 16–103](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-title-settings/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/settings/SessionTitleSettingsComponent.kt#L16).
  Use root inside `TuiPopupHost`, or Panel/Menu with the same handles across viewport/host.
  Host retains stable child and owns lifecycle; removing triggers dismisses renderer menus.
  Integrator removes old title fields/projections/commands and title content/menus centrally.

### Adapter requirements and validation

- Inject canonical `DefaultSessionTitleModel` through `defaultModel`; no copied model literal and
  no `app-shared-session-title` dependency in this component. SessionTitleSettings/OpenAI types
  remain in their existing owners; the spec explicitly retains the allowed shared-settings contract
  dependency (its pre-existing mixed dependency closure is not claimed pure).
- Bind settings/models as independent read-only sources. VM's pure projection is ordered distinct
  catalog slugs plus effective model; null uses injected default, absent model remains selectable.
  Catalog refresh never writes settings or triggers title generation.
- Reasoning menu remains None/Minimal/Low/Medium/High/XHigh/Max; a Custom effort displays wireName,
  without auto-correction. Disabled renderer controls preserve configuration and explanatory text;
  public VM commands still admit edits while disabled, matching baseline.
- Each port admits the captured current field baseline/value to the original application queue.
  Merge only that field; preserve unrelated fields and existing conflict behavior. Accepted freezes
  payload, survives hide/close/owner cancellation and is not persistence success; no optimistic state.
- Shared application failure/report/ack remains the sole authority. Rejection/unknown exceptions
  report once generically; cancellation propagates unchanged. Set `showOperationFailure=false` when
  Settings renders the shared acknowledgement; close rejects subsequent edits and stops observation.
- Added **5 dependency-only VM cases** and **5 full-renderer cases**: injected default, empty/duplicate/
  missing catalog, live changes, Custom/fixed reasoning, disabled public edits, current baselines,
  accepted close drain/field merge, rejection/error/cancellation/close; complete section/menus,
  pointer checkbox/disabled triggers, keyboard selection/Escape and shared banner suppression.
- Static review: three convention builds; no app-contract-settings/app-view-settings/RPC/title-generator
  dependencies; complete renderer consumes framework-only components helpers.
- Centralized JVM validation passed: 10 component cases, adapted GlobalSettingsGroupingTest,
  production queue/default-model integration and the backend session-title regression suite.
  Native/JS/CLI validation remains outside the tested target set.

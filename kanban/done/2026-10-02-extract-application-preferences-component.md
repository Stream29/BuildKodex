# Task Tree

- `Review frontend preferences, transient widths, and paired input keys`()
- `Review component spec and separate persistence/runtime ports`()
- `Extract ViewModel and complete General-page renderer`()
- `Hand application-width and shared queue adapters to the integrator`()
- `Validate width lifetime, key pairing, codecs, and host interaction`()
- `Record six-component batch acceptance`()

# Details

## Scope and source map

- 属于 [六组件批次](2026-10-02-extract-settings-configuration-and-account-components.md)，
  用户已验收通过，随六组件批次归档到 done。
- 目标：`app/component/application-preferences/{spec,impl/viewmodel,impl/view}`。
- 覆盖 Settings General 页：左右侧栏宽度、New line key、Submit key、
  SettingsSidebarWidthItem 和 GeneralSettingsDropdownMenus。
- 原 GlobalSettingsState 的 frontend input/width 投影及命令，RpcGlobalEditor 接线，
  RpcGlobalSettings.sidebarWidths 的应用作用域数据。
- 不迁移整个 SessionSidebar、侧栏内容导航、拖拽算法或 composer；
  canonical NewLineKey/SubmitKey/CliSidebarSettings/codec 留原共享所有者。

## Implemented spec and dependency ports

- `ApplicationPreferencesViewModel`、typed dependencies/factory；
  state 有左右可显示宽度、唯一 newline key、派生 submit key、closed/failure。
  命令为左右宽度、输入键选择及 lifecycle。
- deps 分开提供 frontend key source/write admission 和运行期 widths/resize；
  不将这两类操作统一成“保存 Preferences”的整块写入。
- VM 从同一 NewLineKey 派生 SubmitKey；选 SubmitKey 只映射回对应 NewLineKey。
  不产生非法组合、两个可写 flow 或重复落盘字段。
- 宽度来源由应用持有、组件只观察；写一侧应使用依赖当前另一侧，
  不将渲染旧快照中的另一侧回写。关闭 child 不重置应用宽度。

## Behavior and rendering baseline

- 宽度在应用启动时各按 applicationWidth/4 初始化，完全不持久化。
  页面显示遵循 MinimumSidebarWidthColumns；输入控件上下限、clamp 与
  resizeSidebars 接受非负值的差异需写入测试，不能扩大到改变全局布局算法。
- Width 的 +/- 与立即应用、General 两段布局保留；各次命令是否基于显示值
  或最新 dependency 值须固定，防止拿旧 renderer callback 做“双重更新”。
- ShiftEnter newline 对应 Enter submit；Enter newline 对应 CtrlEnter submit，
  沿现有枚举/serial names，键选择通过原 application queue 保存前端文件。
- accepted key write 排空，运行期 resize 不伪装为队列持久化。
  KDoc 覆盖非法宽度的实际 `@throws`、取消/close、未知错误与 failure 渲染；
  不把意料外异常掩盖为已成功更新。

## Checks

- fake ports：宽度初值/最小值/大值、单侧更新保留最新另一侧、
  popup reopen、closed、key 两向映射、queue rejection/accepted close drain。
- renderer：四个完整控件、原 +/- 鼠标/键盘、两个 key 菜单与 anchor，
  disabled/failed 展示，不在 View 维护另一份 preference draft。
- 保留 GlobalSettingsGroupingTest 的宽度用例；本地 codec、Application/sidebar
  回归验证宽度未写盘、sidebars left/right 内容及其他字段不被覆盖。

## Integration handoff

- Component implementation and centralized JVM validation are complete; user accepted the batch.
  No Gradle, commit, push, branch change, IDE operation or shared-source edit was performed.
- Owned tree: `Kodex/app/component/application-preferences/`; convention projects:
  `:app-component-application-preferences-spec`,
  `:app-component-application-preferences-impl-viewmodel`,
  `:app-component-application-preferences-impl-view`.
- [Authoritative spec/KDoc, lines 9–96](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/application-preferences/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/applicationpreferences/ApplicationPreferencesViewModel.kt#L9).
  Contract and factory package: `io.github.stream29.kodex.app.applicationpreferences`.

### Exact public API

```kotlin
enum class PreferencesWriteAdmission { Accepted, Rejected }
interface ApplicationPreferencesDependencies {
    val widths: StateFlow<Pair<Int, Int>>
    val newLineKey: StateFlow<NewLineKey>
    val operationFailure: StateFlow<Boolean>
    fun setLeftWidth(columns: Int): Unit
    fun setRightWidth(columns: Int): Unit
    fun setNewLineKey(newLineKey: NewLineKey): PreferencesWriteAdmission
    fun reportFailure(failure: Throwable): Unit
    fun dismissFailure(): Unit
}
data class ApplicationPreferencesState(
    val leftWidth: Int,
    val rightWidth: Int,
    val newLineKey: NewLineKey,
    val operationFailure: Boolean = false,
    val closed: Boolean = false,
) {
    val submitKey: SubmitKey // derived from newLineKey.submitKey
}
interface ApplicationPreferencesViewModel : AutoCloseable {
    val state: StateFlow<ApplicationPreferencesState>
    fun setLeftWidth(columns: Int): Unit
    fun setRightWidth(columns: Int): Unit
    fun setNewLineKey(newLineKey: NewLineKey): Unit
    fun setSubmitKey(submitKey: SubmitKey): Unit
    fun hidePage(): Unit
    fun dismissFailure(): Unit
    override fun close(): Unit
}
fun interface ApplicationPreferencesViewModelFactory {
    fun create(dependencies: ApplicationPreferencesDependencies, ownerScope: CoroutineScope): ApplicationPreferencesViewModel
}
object DefaultApplicationPreferencesViewModelFactory : ApplicationPreferencesViewModelFactory
fun createApplicationPreferencesViewModel(
    dependencies: ApplicationPreferencesDependencies,
    ownerScope: CoroutineScope,
): ApplicationPreferencesViewModel
```

- Mosaic package `io.github.stream29.kodex.cli.settings`; remember/root/panel/menu functions are
  `@Composable`, and dropdown handles are renderer-only:

```kotlin
class ApplicationPreferencesDropdowns(val newLineKey: TuiDropdownState, val submitKey: TuiDropdownState)
fun rememberApplicationPreferencesDropdowns(): ApplicationPreferencesDropdowns
fun BoxScope.ApplicationPreferencesComponent(viewModel: ApplicationPreferencesViewModel, showOperationFailure: Boolean = true)
fun ApplicationPreferencesPanel(viewModel: ApplicationPreferencesViewModel, dropdowns: ApplicationPreferencesDropdowns, showOperationFailure: Boolean = true)
fun BoxScope.ApplicationPreferencesDropdownMenus(viewModel: ApplicationPreferencesViewModel, dropdowns: ApplicationPreferencesDropdowns)
```

- [Renderer, lines 13–126](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/application-preferences/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/settings/ApplicationPreferencesComponent.kt#L13).
  Use root inside `TuiPopupHost`, or Panel/Menu with the same handles across viewport/host.
  Integrator owns removing old General content, width helper, key menus and parent commands/projections,
  as well as adapting the existing grouping tests rather than copying their shared fixture.

### Adapter requirements and validation

- Supply the application-owned transient `StateFlow<Pair<Int,Int>>` (left/right), not SidebarSettings.
  Initialization remains applicationWidth/4 in its current owner; construction, observation, hide,
  close and reopening never initialize/resize/persist widths.
- Projection clamps display only to canonical MinimumSidebarWidthColumns. **Active explicit commands
  accept any nonnegative width unchanged, including 0..3**, retaining baseline resize behavior.
  Negative active input throws IllegalArgumentException; closed commands are no-ops.
- Renderer +/- captures displayed columns +/-1, with minus disabled at minimum and plus at MAX_VALUE.
  Do not replace commands with refresh-then-increment/decrement. Adapter must use
  `resizeSidebars(columns, widths.value.second)` / `resizeSidebars(widths.value.first, columns)` at
  execution time, preserving current other width even if child display is stale.
- Only canonical NewLineKey is queued/persisted. SubmitKey selection maps through `submitKey.newLineKey`
  once; displayed submit is derived from canonical enum pairing. Never persist a second key/width,
  change codecs/defaults or write a full preferences/settings snapshot.
- Key admission freezes enum payload into the original frontend application queue, preserving other
  fields and draining after child close/owner cancellation. Accepted is not a receipt.
- Bind failure/report/ack to existing application source/port. Rejection/unknown exceptions report
  generically once; cancellation propagates unchanged. `showOperationFailure=false` avoids a duplicate
  banner when Settings already renders the source. Closed child stops observation, not application.
- Added **6 dependency-only VM cases** and **5 full-renderer cases**: min/MAX projection without writes,
  current-other-width/snapshot semantics, explicit 0..3/negative commands, canonical two-way key
  pairing, accepted close drain/reopen, rejection/errors/cancellation/owner close; ordered General
  controls, pointer/keyboard +/-, disabled bounds, both key menus/selection/Escape, shared banner.
- Static review: three convention builds; no app-contract-settings/app-view-settings/RPC dependency;
  canonical shared-settings contract retained with its existing mixed closure, not claimed purified.
- Centralized JVM validation passed: 11 component cases plus production queue/codec,
  Application/sidebar/Settings regressions. RpcSettingsTest checks raw 0..3 widths, current other
  width, no settings-file write, and no mutation after close. Native/JS/CLI remain unverified.

## Integrator baseline clarification

- 原公开宽度写命令接受任意非负值（包括 0..3），仅 renderer/state 投影显示最小值；
  新显式 set-width 命令也应把非负原值传给 adapter，不将 0..3 自动改写为最小值。
  保留负值异常及关闭后 no-op；这与 renderer 的减号最小值禁用是不同约束。

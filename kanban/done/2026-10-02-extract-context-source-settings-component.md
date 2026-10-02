# Task Tree

- `Review Context Source baseline, validation, and environment boundary`()
- `Review component spec, ports, and complete interaction contract`()
- `Extract dependency-only ViewModel and list/add renderer`()
- `Hand shared declarations and queue integration to the integrator`()
- `Validate paths, duplicate sources, stale callbacks, and accepted-write drain`()
- `Record six-component batch acceptance`()

# Details

## Scope and source map

- 属于 [六组件批次](2026-10-02-extract-settings-configuration-and-account-components.md)，
  用户已验收通过，随六组件批次归档到 done。
- 目标：`app/component/context-source-settings/{spec,impl/viewmodel,impl/view}`。
- 原 View：`app/view/settings/.../ContextSourcesSettings.kt` 的列表和新增对话框；
  SettingsPopup 的 `contextSourceAddOpen` 也迁入组件业务状态。
- 原 contract：GlobalSettingsState.contextSources、BuiltInContextSource 与四种命令；
  原 adapter：RpcGlobalEditor 中来源写入；原 helper：
  `app/viewmodel/settings/.../GlobalSettingsViewModel.kt` 的路径/开关函数。
- AgentContextSourceSettings/AgentContextCustomSource 留 agent-context spec。
  BuiltInContextSource 单一声明迁入本 spec，优先保留包名。

## Implemented spec and dependency ports

- `ContextSourceSettingsViewModel`、`ContextSourceSettingsDependencies`、factory；
  state 包含 ordered sources、Hidden/Adding(draft,validation)、closed、
  应用级 operation failure。命令覆盖开关、打开/编辑/确认/取消新增、删除、hide/close。
- 输入每次同步到 VM draft；已知验证返回结构化失败或 safe message，
  不用 renderer `remember` 保存第二份 path/error。
- deps 提供只读 source、环境相关比较 policy、typed write admission、
  failure report/acknowledgement；不暴露 GlobalRpc、store、raw CAS 或文件系统。
- policy 将前端 home/静态来源比较留 adapter；VM 负责判断并呈现验证。
  意料外环境失败与非法输入区别写明，实际异常在对应 KDoc `@throws` 声明。
- 新增/取消回调带 exact interaction identity，晚到 callback 不关闭替换的新对话框；
  具体 token/result 签名在主计划评审后冻结。

## Behavior and rendering baseline

- 五个内置开关：Agents/Kodex/Codex home、Git root、cwd；不可编辑/删除其路径。
- 输入 trim 后允许 absolute path、`~`、`~/...`，拒绝 blank、含 `$` 和其他相对路径。
  比较使用现有前端 home；最终保存原 trimmed 路径文本，不把前端绝对展开值写给后端。
- 静态 home 重复拒绝；同规范化自定义路径只启用既有条目，不追加或改写其文本。
  新条目追加，已有顺序保持；启停/删除沿当前 exact stored path 和字段 baseline。
- 保留 Add validation 文案与按钮/焦点；空列表与内置说明完整渲染。
  accepted admission 关闭新增，不声称保存成功；异步失败仍由应用 flag 呈现。
- hide/close 丢弃未接受草稿、关闭 child 观察，不取消原应用 queue 已接受写入。
  取消不转业务失败，不执行上下文扫描或读取用户文件来“验证”新增路径。

## Checks and review gates

- 测试 blank/relative/`$`/绝对路径/`~`、静态重复、等价自定义路径、启用与保序；
  policy 环境失败、取消、replaced draft、double submit、close 后命令。
- 端口写入测试 baseline 冲突、只变目标字段、队列拒绝与 close 后排空。
- renderer 覆盖输入/validation/Cancel/Add、Escape、exact callback 与列表开关/删除。
- 当前前端 home 比较可能与后端解释不同；仅记录并保持基线，
  跨设备路径策略属于另行设计，不新增 directory RPC。
- 共享源码/宿主与现有 Settings 测试由整合负责人接线；不顺带迁移整个 agent-context。

## Integration handoff

- Component implementation and centralized JVM validation are complete; user accepted the batch.
  No Gradle, commit, push, branch change, IDE operation or shared-source edit was performed.
- Owned tree: `Kodex/app/component/context-source-settings/`; three convention projects:
  `:app-component-context-source-settings-spec`,
  `:app-component-context-source-settings-impl-viewmodel`,
  `:app-component-context-source-settings-impl-view`.
- Canonical contract/KDoc:
  [spec, lines 15–148](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/context-source-settings/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/contextsourcesettings/ContextSourceSettingsViewModel.kt#L15).
  Types/factory below use package `io.github.stream29.kodex.app.contextsourcesettings`.

### Exact public API

```kotlin
sealed interface ContextSourceWriteAdmission {
    data object Accepted : ContextSourceWriteAdmission
    data class Rejected(val message: String) : ContextSourceWriteAdmission
}
interface ContextSourcePathPolicy {
    val builtInNormalizedPaths: Set<String>
    fun normalize(path: String): String?
}
interface ContextSourceSettingsDependencies {
    val sources: StateFlow<AgentContextSourceSettings>
    val pathPolicy: ContextSourcePathPolicy
    val operationFailure: StateFlow<Boolean>
    fun setBuiltInEnabled(source: BuiltInContextSource, expected: Boolean, enabled: Boolean): ContextSourceWriteAdmission
    fun replaceCustomSources(expected: List<AgentContextCustomSource>, updated: List<AgentContextCustomSource>): ContextSourceWriteAdmission
    fun setCustomEnabled(original: AgentContextCustomSource, enabled: Boolean): ContextSourceWriteAdmission
    fun removeCustom(original: AgentContextCustomSource): ContextSourceWriteAdmission
    fun reportFailure(failure: Throwable): Unit
    fun dismissFailure(): Unit
}
class ContextSourceDialogToken
sealed interface ContextSourceSettingsDialog {
    data object Hidden : ContextSourceSettingsDialog
    data class Adding(
        val token: ContextSourceDialogToken,
        val draft: String = "",
        val error: String? = null,
    ) : ContextSourceSettingsDialog
}
data class ContextSourceSettingsState(
    val sources: AgentContextSourceSettings,
    val dialog: ContextSourceSettingsDialog = ContextSourceSettingsDialog.Hidden,
    val operationFailure: Boolean = false,
    val closed: Boolean = false,
)
interface ContextSourceSettingsViewModel : AutoCloseable {
    val state: StateFlow<ContextSourceSettingsState>
    fun add(): Unit
    fun updateDraft(token: ContextSourceDialogToken, path: String): Unit
    fun save(token: ContextSourceDialogToken): Unit
    fun setBuiltInEnabled(source: BuiltInContextSource, enabled: Boolean): Unit
    fun setCustomEnabled(path: String, enabled: Boolean): Unit
    fun removeCustom(path: String): Unit
    fun dismiss(token: ContextSourceDialogToken): Unit
    fun hidePage(): Unit
    fun dismissFailure(): Unit
    override fun close(): Unit
}
fun interface ContextSourceSettingsViewModelFactory {
    fun create(dependencies: ContextSourceSettingsDependencies, ownerScope: CoroutineScope): ContextSourceSettingsViewModel
}
object DefaultContextSourceSettingsViewModelFactory : ContextSourceSettingsViewModelFactory
fun createContextSourceSettingsViewModel(
    dependencies: ContextSourceSettingsDependencies,
    ownerScope: CoroutineScope,
): ContextSourceSettingsViewModel
```

- `BuiltInContextSource` is copied verbatim to this spec with original FQCN
  `io.github.stream29.kodex.app.settings.contract.BuiltInContextSource`.
  [Declaration, lines 1–10](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/context-source-settings/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/settings/contract/BuiltInContextSource.kt#L1).
  Integrator owns removing the old declaration before compilation; canonical AgentContext types stay put.
- Mosaic package: `io.github.stream29.kodex.cli.settings`; all entrypoints are `@Composable`:

```kotlin
fun BoxScope.ContextSourceSettingsComponent(viewModel: ContextSourceSettingsViewModel, showOperationFailure: Boolean = true)
fun ContextSourceSettingsPanel(viewModel: ContextSourceSettingsViewModel, showOperationFailure: Boolean = true)
fun BoxScope.ContextSourceSettingsDialogs(viewModel: ContextSourceSettingsViewModel)
```

- [Renderer, lines 29–145](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/context-source-settings/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/settings/ContextSourceSettingsComponent.kt#L29).
  Use full root inside `TuiPopupHost`, or Panel in the viewport and Dialogs at the host.
  Host supplies stable child, calls `hidePage()` on navigation and `close()` on disposal.
  Delete host `contextSourceAddOpen`, old add draft/error and old renderer/commands centrally.

### Adapter requirements and validation

- Supply the existing read-only context source projection and frontend-home comparison policy.
  Policy returns null for ordinary invalid input; unexpected environment errors throw and are reported.
  It must not scan/read context files, change backend path interpretation or normalize saved spelling.
- Every admission is synchronous and copies exact baseline/payload into the original app queue.
  Add uses the whole custom-list baseline; VM freezes replacement list and first equivalent enabled
  entry in order. Toggle/delete use exact original path/value; unrelated backend fields remain merged.
  Queue ownership must outlive child close/owner cancellation; Accepted is not a save receipt.
- Bind failure/report/ack to the existing application source/port. State only projects that flag.
  Pass `showOperationFailure=false` when Settings already renders its acknowledgement.
  Cancellation propagates unchanged; no raw exception details are shown.
- Exact-token draft validation, double/reentrant save guard and stale replacement callbacks are in VM.
  Renderer stores only synchronized input buffer/cursor/undo; it owns no error or add-open authority.
- Added **11 dependency-only VM cases** and **5 full-renderer cases** in the new trees:
  invalid/static/equivalent paths, trim/order, live baselines, rejection, environment errors,
  cancellation, stale/reentrant callbacks, accepted close drain, observations, list controls,
  keyboard typing/Add/Escape, Cancel, programmatic draft updates and shared banner suppression.
- Static review: three convention builds; no settings-contract/view-settings/RPC/environment-I/O
  dependencies; rendering uses the framework-only helpers already in `app/view/components`.
- Centralized JVM validation passed: 16 component cases plus Settings/Application/RPC
  regressions, including production queue field merging and accepted-write drain.
  Native/JS/CLI and real environment/filesystem integration are not claimed by fake-policy tests.

# Task Tree

- `Review usage wire values, fallback, navigation refresh, and reset availability`()
- `Review observation/command ports and full rendering semantics`()
- `Extract dependency-only ViewModel and usage summary/actions renderer`()
- `Hand navigation and one-way reset intent wiring to the integrator`()
- `Validate all usage branches, command cancellation, and wire compatibility`()
- `Record six-component batch acceptance`()

# Details

## Scope and source map

- 属于 [六组件批次](2026-10-02-extract-settings-configuration-and-account-components.md)，
  用户已验收通过，随六组件批次归档到 done。
- 目标：`app/component/account-usage/{spec,impl/viewmodel,impl/view}`。
- 原 Global Settings 的 SettingsAccountUsageState、snapshotOrNull、
  usage observation/refresh/reset 入口；CodexAccountUsageSettings.kt 的 summary、
  formatters 与动作部分。重置 dialog/消费属于独立 Usage Reset 组件。
- 实际 usage source 仍由 RpcGlobalSettings 共享观察/后端 store 提供，
  不复制 store 或 BackendSettings 字段，不再让 renderer 直接消费整个 Global VM。

## Implemented spec and dependency ports

- `AccountUsageViewModel`、typed dependencies/factory；
  read-only state 保留 usage 联合及 derived action availability/closed，
  command 覆盖 refresh 与 requestReset，不持有重置选择/确认副本。
- 现有 Serializable SettingsAccountUsageState 和 snapshotOrNull 迁入本 spec，
  保持包名、嵌套类型、payload/default/nullability/serializer descriptor。
  Snapshot/credit/outcome 复用 OpenAI account-usage spec，禁止复制 DTO。
- deps 为 usage source、suspend refresh command、typed requestReset port、
  应用 failure reporting；Account Usage spec 不引用 usage-reset spec/impl。
  宿主接通 reset.show；reset spec 可单向引用本 spec 的用量 DTO。
- KDoc 将刷新准入、RPC 返回与 source publication 分开；
  account change/fallback 依后端语义，前端不推导原子 auth+usage 组合。

## Behavior and rendering baseline

- 全覆盖 Unavailable、Loading(previous?)、Available、Failed(previous?)、Redeeming。
  同账号 previous 可继续展示；没有 previous 时不可装成空/零。
- lifetime tokens、reset count、windows、日期和 unavailable sections 保留差异；
  safe failure message 与 loading/redeeming 状态有明确渲染，不渲染凭据。
- 原 UI 在 Loading/Redeeming 禁用 Refresh、Unavailable 隐藏动作；
  旧 refresh 命令本身并未用此 UI 条件做全局去重。新 command guard 先测基线。
- 原首次进入/切回 OpenAI 页都刷新，由 Settings 导航接线继续触发；
  新 VM constructor 不额外自动刷新，用户 Refresh 也不产生双重命令。
- Use reset 在 Available 且计数/明细显示可能可用时开启，入口仅请求选择，
  不直接消费。只有计数而无明细可能允许进入错误提示，不能默认选择券。
- 旧文案称“backend will choose a reset”与现行协议冲突，提案改为明细不可用、
  刷新后重试；不新增后端自动选券。null expiry 的未知说明由 reset renderer 负责。
- close 取消本组件观察/等待，不关闭共享 source；失败统一报告，
  取消不报告，Unit 返回不当作最新快照或 Available 的保证。

## Checks

- dependency-only 测试全部联合/fallback、null/zero/empty、account replacement、
  action availability、manual/enter refresh、重复/取消/close 与 requestReset-only。
- renderer 覆盖各 branch、tokens/count/date 文案、按钮禁用/空明细提示；
  迁入 CodexAccountUsageSettingsTest，补真正 VM+fake ports 的交互测试。
- RPC AccountUsageValuesTest、backend usage 与 auth isolation 回归；
  comparison fixtures 验证 descriptor/序列化未变，取消观察不关闭共享后端 store。

## Integration handoff checkpoints

- Component implementation and centralized JVM validation are complete; user accepted the batch.
  Only this new component tree, its sibling Authentication tree and their handoff sections
  were changed by this work unit. Shared renderer/host/tests/Gradle were left to the integrator.
- Three projects use the required conventions:
  `:app-component-account-usage-spec` (`kodex.kmp-cli`, serialization),
  `:app-component-account-usage-impl-viewmodel` (`kodex.kmp-viewmodel`),
  `:app-component-account-usage-impl-view` (`kodex.kmp-view`).
  Spec depends only on `openai-spec-account-usage`, coroutines and serialization.
  No Global Settings, RPC, `app-view-settings`, or Usage Reset dependency.
- The original `io.github.stream29.kodex.app.settings.contract.SettingsAccountUsageState`
  and `snapshotOrNull()` are copied into
  [the component spec](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/account-usage/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/settings/contract/SettingsAccountUsageState.kt#L1).
  Original nested names, serializer annotations, payload/order/defaults/nullability and
  validation remain unchanged. Snapshot/credit values retain their existing OpenAI owner.
  Usage Reset/Session consumers can import the unchanged DTO names by depending on this spec.
  Integrator owns removing old declarations and updating RPC/downstream dependencies.

### Exact public integration API

- Spec and factory package: `io.github.stream29.kodex.app.accountusage`.
  [Canonical ports and state/render semantics](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/account-usage/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/accountusage/AccountUsageViewModel.kt#L14).

```kotlin
interface AccountUsageDependencies {
    val usage: StateFlow<SettingsAccountUsageState>
    val operationFailure: StateFlow<Boolean>
    suspend fun refresh(): Unit
    fun requestReset(): Unit
    fun reportFailure(failure: Throwable): Unit
    fun dismissFailure(): Unit
}
fun createAccountUsageViewModel(
    dependencies: AccountUsageDependencies,
    ownerScope: CoroutineScope,
): AccountUsageViewModel
// Also: DefaultAccountUsageViewModelFactory : AccountUsageViewModelFactory

data class AccountUsageState(
    val usage: SettingsAccountUsageState,
    val operationFailure: Boolean = false,
    val closed: Boolean = false,
) {
    val actionsVisible: Boolean // Derived, presentation only.
    val refreshEnabled: Boolean // Derived, presentation only.
    val resetEnabled: Boolean   // Available + possible count/detail credit.
}
interface AccountUsageViewModel : AutoCloseable {
    val state: StateFlow<AccountUsageState>
    fun refresh(): Unit
    fun requestReset(): Unit
    fun dismissFailure(): Unit
    override fun close(): Unit
}
```

- Renderer package: `io.github.stream29.kodex.cli.accountusage`.
  [Complete summary/actions renderer](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/account-usage/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/accountusage/AccountUsageComponent.kt#L30).

```kotlin
@Composable fun AccountUsageComponent(
    viewModel: AccountUsageViewModel, showOperationFailure: Boolean = true,
)
@Composable fun AccountUsageContent(
    state: AccountUsageState,
    onRefresh: () -> Unit,
    onUseReset: () -> Unit,
    onDismissFailure: () -> Unit,
    showOperationFailure: Boolean = true,
)
```

### Adapter and validation checkpoints

- Supply the existing shared usage and operation-failure flows. `refresh()` delegates to
  `refreshAccountUsage`; Unit completion is not an Available/current-cache guarantee.
  `requestReset()` is a one-way Unit intent to host-owned `usageReset.show()`, with no credit
  argument or reset child/state/consumption dependency. Host/reset owner validates the request.
- Keep first-entry/re-entry OpenAI refresh in Settings navigation; no constructor/composition
  refresh was added. Every active explicit `refresh()` launches a call, including overlapping
  Loading/Redeeming/Unavailable calls. Renderer disablement is not command-global dedupe.
- Render Unavailable without actions; Loading/Failed use only source-supplied same-account
  fallback; Available renders full windows/tokens/count/actions; Redeeming retains snapshot
  and shows progress. No old-account cache, inferred auth+usage atomicity, or reset dialog exists.
- Preserve possible-credit reset enablement, including count-only snapshots. They only open
  the reset owner's unavailable-details path; no automatic selection/consumption is introduced.
  Text now says details unavailable and suggests refresh, never that the backend chooses a
  credit. Unknown token/count/details remain distinct from zero and known empty lists.
- Compose AccountUsageComponent in the Account section with `showOperationFailure = false`
  when the host already renders the application failure. Host closes the stable child at
  Settings end, not on page unmount. Close cancels only child observations/refresh waits,
  not the shared usage store or host-owned reset.
- New dependency-only suite:
  [AccountUsageViewModelTest](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/account-usage/impl/viewmodel/src/commonTest/kotlin/io/github/stream29/kodex/app/accountusage/AccountUsageViewModelTest.kt#L1).
  Covers every union/fallback and action rule, null/zero/empty detail matrix, account replacement,
  no constructor refresh, overlapping commands in every state, Unit/source separation,
  reset-only intent, source/command failure distinction, cancellation and close/owner lifetime.
- New renderer suite:
  [AccountUsageComponentTest](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/account-usage/impl/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/accountusage/AccountUsageComponentTest.kt#L1).
  Covers all branches/fallbacks, rate windows/durations/delays/exhaustion, grouped counts/tokens,
  unavailable versus zero, corrected details text, disabled keyboard actions, real child refresh
  and reset intents, safe shared failure/dismissal, delegation and closed rendering.
- Integrator removed the old combined usage/reset renderer and completed Settings/RPC adapters.
  Centralized JVM validation passed: 46 component cases, migrated usage renderer tests,
  frozen RPC descriptor/JSON checks and navigation-triggered refresh regression.
  Native/JS/CLI and live provider usage behavior remain unverified.

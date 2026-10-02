# Task Tree

- `Review source selection, wire summary, login routing, and logout timing`()
- `Review typed ports, confirmation semantics, and wording corrections`()
- `Extract authentication state/operation owner and complete renderer`()
- `Hand RPC DTO dependencies, queue, and Login routing to the integrator`()
- `Validate selected-source effects, stale callbacks, close, and serializers`()
- `Record six-component batch acceptance`()

# Details

## Scope and source map

- 属于 [六组件批次](2026-10-02-extract-settings-configuration-and-account-components.md)，
  用户已验收通过，随六组件批次归档到 done；与 Account Usage 同工作单元，Usage Reset 独立处理。
- 目标：`app/component/authentication-settings/{spec,impl/viewmodel,impl/view}`。
- 原 SettingsPopup 的 AuthenticationSettingsContent、来源菜单、
  OpenAiLogoutConfirmationDialog 与 renderer-local logout boolean；
  Global Settings 的 authentication/operation/effect 和相关命令。
- RpcGlobalEditor 保留 adapter 和 queue，不把 GlobalRpc 注入 component spec/VM。
  原 OpenAI Login 组件及 RpcOAuth 协议不重复实现。

## Implemented spec and dependency ports

- `AuthenticationSettingsViewModel`、typed dependencies/factory；
  state 暴露 selected source、safe summary、Idle/SigningOut/Failed 操作、
  VM-owned confirmation、closed。来源摘要不包含 token 或 reset 身份。
- 现有 Serializable SettingsAuthenticationState 迁入本 spec，FQCN/serializer
  形状不变；SettingsAuthenticationOperation/State 单一声明也迁入。
- deps 提供 independent source/summary observations、source write admission、
  `remove(source)` 与 typed login-opening port、failure handling。
  接入现有应用 Login child，浏览器 URL 不经新的全局广播 channel。
- 来源切换清理未消费 reset 选择由宿主协调；组件不依赖 reset impl。
  原 shared write queue 与短期 logout command owner 保持不同生命周期。
- KDoc 明确 flow 非联合原子、点击/确认/调用准入捕获的目标、重复命令准入、
  CancellationException 和普通失败、closed、各认证分支与安全渲染。

## Baseline and explicit review gates

- 两种 source 都由后端管理，均支持 login/relogin/remove；不引入 Codex 只读限制、
  Reload 操作、远程 revoke 或删除另一个来源。
- 来源选择走 settings CAS；登录目标在实际创建 Login 时绑定，成功不隐式切来源。
  协调 effect/port 形态须保留该时点，不暗改为更早的页面投影目标。
- 注销目前在 `removeAuthentication()` 命令开始时读取并固定 source，
  而不是打开确认时；进行中抑制重复 remove，结果不直接伪造 auth summary。
- 旧注销文案固定为 Kodex，和 source=Codex 的真实操作冲突。
  本计划提案按来源展示真实目标；是否把确认目标改为打开时绑定需另列差异测试
  并在评审中决定，不借 UI token 宣称已有“旧来源不可能被改”的保证。
- 旧 renderer 在 SigningOut 禁用来源与登录；旧程序调用 source-write
  不一定被拒绝。新 spec 的 guard 先对照基线，不能默默加全局操作锁。
- page hide 丢弃未确认 dialog；close 释放短期 operation/观察，不关闭 shared auth
  store，不取消已接受 settings write。取消等待不证明后端尚未删除凭据。

## Checks

- fake ports：两种来源、各 typed unavailable reason、摘要隐私、Sign in/relogin、
  selected-source removal、双确认、错误/取消、late token、来源变化与 close。
- 确认文案两来源分别测试；登录继续使用已有 OpenAI Login lifecycle 回归，
  不以模型刷新或 usage summary 推断某次登录已完成。
- 迁入 AuthenticationSettingsTest；保留 RPC AuthenticationValuesTest、
  RpcOAuthTest 和 backend authentication tests，逐项比较序列化 descriptor/JSON。
- runtime metadata/file 格式、续期/回存/防迟到恢复规则不在本批重写范围。

## Integration handoff checkpoints

- Component implementation and centralized JVM validation are complete; user accepted the batch.
  This work unit changed only the new authentication/account-usage trees and these two
  handoff sections. No build, commit, push, shared-source deletion, or host-test migration ran here.
- Three projects use the required conventions:
  `:app-component-authentication-settings-spec` (`kodex.kmp-cli`, serialization),
  `:app-component-authentication-settings-impl-viewmodel` (`kodex.kmp-viewmodel`),
  `:app-component-authentication-settings-impl-view` (`kodex.kmp-view`).
  Spec depends on `app-shared-settings-contract` for the original `KodexAuthSource`,
  `openai-spec-models`, coroutines and serialization; no Global Settings/RPC/view dependency.
  The shared-settings module's pre-existing mixed closure remains outside this unit.
- The original `io.github.stream29.kodex.app.settings.contract.SettingsAuthenticationState`,
  `SettingsAuthenticationOperation`, and `SettingsAuthenticationOperationState` are copied into
  [the component spec](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/authentication-settings/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/settings/contract/SettingsAuthenticationState.kt#L1).
  Original nested type names, serializer annotations, fields/order/defaults/nullability,
  validation and enum values are unchanged. Integrator owns deleting old declarations and
  retargeting RPC spec dependencies before compilation.

### Exact public integration API

- Spec and factory package: `io.github.stream29.kodex.app.authenticationsettings`.
  [Canonical ports and lifecycle KDoc](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/authentication-settings/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/authenticationsettings/AuthenticationSettingsViewModel.kt#L15).

```kotlin
interface AuthenticationSettingsDependencies {
    val selectedSource: StateFlow<KodexAuthSource>
    val authentication: StateFlow<SettingsAuthenticationState>
    val operationFailure: StateFlow<Boolean>
    fun updateSource(source: KodexAuthSource): Boolean
    suspend fun remove(source: KodexAuthSource): Unit
    fun openLogin(): Unit
    fun reportFailure(failure: Throwable): Unit
    fun dismissFailure(): Unit
}
fun createAuthenticationSettingsViewModel(
    dependencies: AuthenticationSettingsDependencies,
    ownerScope: CoroutineScope,
): AuthenticationSettingsViewModel
// Also: DefaultAuthenticationSettingsViewModelFactory : AuthenticationSettingsViewModelFactory

class AuthenticationLogoutConfirmation // Exact reference identity; no source binding.
data class AuthenticationSettingsState(
    val selectedSource: KodexAuthSource,
    val authentication: SettingsAuthenticationState,
    val operation: SettingsAuthenticationOperationState = SettingsAuthenticationOperationState.Idle,
    val confirmation: AuthenticationLogoutConfirmation? = null,
    val operationFailure: Boolean = false,
    val closed: Boolean = false,
)
interface AuthenticationSettingsViewModel : AutoCloseable {
    val state: StateFlow<AuthenticationSettingsState>
    fun updateSource(source: KodexAuthSource): Boolean
    fun requestLogin(): Unit
    fun requestLogout(): Unit
    fun confirmLogout(expected: AuthenticationLogoutConfirmation): Unit
    fun cancelLogout(expected: AuthenticationLogoutConfirmation): Unit
    fun hidePage(): Unit
    fun dismissFailure(): Unit
    override fun close(): Unit
}
```

- Renderer package: `io.github.stream29.kodex.cli.authenticationsettings`.
  [Complete renderer](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/authentication-settings/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/authenticationsettings/AuthenticationSettingsComponent.kt#L37).

```kotlin
@Composable fun BoxScope.AuthenticationSettingsComponent(
    viewModel: AuthenticationSettingsViewModel, showOperationFailure: Boolean = true,
)
@Composable fun AuthenticationSettingsPanel(
    viewModel: AuthenticationSettingsViewModel,
    sourceDropdown: TuiDropdownState,
    showOperationFailure: Boolean = true,
)
@Composable fun AuthenticationSettingsContent(
    state: AuthenticationSettingsState,
    sourceDropdown: TuiDropdownState,
    onOpenLogin: () -> Unit,
    onRequestLogout: () -> Unit,
    onDismissFailure: () -> Unit,
    showOperationFailure: Boolean = true,
)
@Composable fun BoxScope.AuthenticationSettingsOverlays(
    viewModel: AuthenticationSettingsViewModel, sourceDropdown: TuiDropdownState,
)
@Composable fun BoxScope.AuthenticationLogoutDialog(
    source: KodexAuthSource, onDismiss: () -> Unit, onConfirm: () -> Unit,
)
```

### Adapter and validation checkpoints

- Provide read-only projected source, safe authentication summary and the application failure
  flow. `updateSource` returns existing queue admission, not a save receipt; host preserves CAS
  baseline, queue drain and reset dismissal. No optimistic source/summary writes occur in child.
- `openLogin()` delivers only the existing buffered Unit opening intent. Keep actual
  `createLogin` source capture in the host; do not pass a prebound source through this port.
  `remove(source)` delegates to existing RPC; reporter/dismiss delegate to the shared authority.
- Confirmation target is **not** captured on open. Dialog text tracks live selected source;
  `confirmLogout(expected)` captures `dependencies.selectedSource.value` synchronously before
  coroutine launch. Exact-object confirm/cancel guards reject old/replaced/hidden dialogs.
  No global source lock: source-write and login programmatic calls remain admitted during
  SigningOut; only renderer controls and duplicate removal are guarded.
- Compose Panel in the host's scrolling Account section; compose Overlays as a direct child
  of `TuiPopupHost` after persistent content, passing the same renderer-owned dropdown.
  Use `showOperationFailure = false` when Settings already displays the shared banner.
  Parent publishes the stable child, calls `hidePage()` when leaving OpenAI and `close()` at
  Settings end. Unmounting the renderer does not close the stable child.
- Both source-specific local-file logout warnings are implemented. Unknown logout failure no
  longer claims credentials were kept. No Reload, revoke, token exposure or OAuth/store copy.
- New dependency-only suite:
  [AuthenticationSettingsViewModelTest](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/authentication-settings/impl/viewmodel/src/commonTest/kotlin/io/github/stream29/kodex/app/authenticationsettings/AuthenticationSettingsViewModelTest.kt#L1).
  Covers source/summary independence, every unavailable reason, admission/rejection,
  command-start capture, stale identities, duplicate removal, programmatic writes/login during
  SigningOut, error/cancellation, hide, close and owner cancellation.
- New renderer suite:
  [AuthenticationSettingsComponentTest](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/authentication-settings/impl/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/authenticationsettings/AuthenticationSettingsComponentTest.kt#L1).
  Covers safe identity/plan, unavailable reasons, disabled keyboard controls, dismiss,
  both warnings/Cancel, live-source keyboard confirmation, menu/login ports and closed state.
- Centralized JVM validation passed: 27 component cases, migrated authentication renderer
  tests, frozen RPC descriptor/JSON checks and OpenAI Login/host regressions.
  Logout text is source-specific and split into short lines to avoid popup truncation.
  Native/JS/CLI and real OAuth/filesystem credential effects remain unverified.

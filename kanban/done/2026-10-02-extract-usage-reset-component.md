# Task Tree

- `Review reset workflow, exact confirmation, RPC semantics, and baseline tests`()
- `Review complete spec and behavior/wording compatibility gates`()
- `Extract dependency-only reset state machine and complete dialog renderer`()
- `Hand account/page lifecycle and RPC adapters to the integrator`()
- `Validate confirmation, unknown outcomes, cancellation, and late responses through central builds`()
- `Record six-component batch acceptance`()

# Details

## Scope and source map

- 属于 [六组件批次](2026-10-02-extract-settings-configuration-and-account-components.md)，
  用户已验收通过，随六组件批次归档到 done；这是独立复杂工作单元，不同时承担其他复杂状态机。
- 目标：`app/component/usage-reset/{spec,impl/viewmodel,impl/view}`。
- 原 UsageResetOption/Request/State 在 GlobalSettingsViewModel.kt；
  原 VM 为 RpcUsageReset；完整 dialog host/branches 在 CodexAccountUsageSettings.kt。
  前端选择/确认移入组件；GlobalRpc、供应商幂等数据、usage store 留 adapter/backend。

## Implemented spec and dependency ports

- `UsageResetViewModel`、typed dependencies/factory；
  明确 Hidden、Choosing、Confirming、Consuming、ConsumeFailed、
  PreparationFailed、Completed 的意义、动作与渲染。
- deps 提供 Account Usage spec 的 usage source、`consume(creditId)`、
  `refreshUsage()`。不提供原始 credentials、backend attempt 或自动选券接口。
- commands 覆盖 show/选择准确 creditId/confirm expected/返回列表/
  显式重新选择 retry/dismiss/close；不让父级再次读取最新 confirming 后替代
  renderer 捕获的 confirmation identity。
- 现有 Option/Request/State 非 wire，单一声明迁移且优先保留包名；
  CodexRateLimitResetOutcome/credits/snapshot 复用现有 OpenAI spec。
- 初始静态核对确认 Preparing 只有旧声明和 renderer，没有生产路径发布；
  新非 wire state 已移除该遗留分支，不增加 prepare RPC；旧声明删除由整合方负责。
- KDoc 覆盖选择来源、二次确认准入、消费/刷新顺序、未知结果、取消与 close、
  所有状态 renderer 行为，以及公开的 require/cancellation/依赖异常。

## Behavior and review gates

- show 只从已有 snapshot 的具体明细生成选择；availableCount 不能作为
  无 creditId 消费的许可。选项无 title/description 沿现有 fallback，
  expiry=null 明确“未知”，不省略到让用户误认永久有效。
- confirm 只准入准确对象/句柄和非空 creditId；伪造等值对象、替换选择、
  double click 或 Consuming 中动作不再次消费。
- 原内部 RpcUsageReset 已有对象身份校验，旧父接口/renderer 未传 expected。
  新完整 renderer 传递准确对象，copied/stale/exact 与替换确认测试记录这个
  已授权差异，不伪称旧父接口已有 exact guarantee。
- Consume 按后端调用准入时当前账号校验券；不增加确认期 account 参数/锁、
  prepare/attempt/result-replay RPC，也不将多个独立流当作原子快照。
- Reset/NothingToReset/NoCredit/AlreadyRedeemed 完整返回；
  确定结果不能因后续刷新失败被改成未知结果。
- 普通消费异常 => ConsumeFailed，并沿现行路径刷新一次；
  刷新不是未消费证明。用户 retry 回到重新选券/确认，不自动重发消费，
  不补偿、不提供 exactly-once 保证。
- 原 page leave/source switch 调用 dismiss，Consuming 时 dismiss 无效；
  close 取消前端 owner 并隐藏。后端已接受命令不保证随等待取消而撤销。
  cancellation、late/noncooperative response 的现存行为须定向测试，
  有差异时单独评审，不能把隐藏界面当成后端回滚。

## Checks

- fake deps：无明细/已知空/计数未知、可选具体券、unknown expiry、
  无匹配 id、exact/伪造/过期确认、double confirm、四种 outcome。
- 未知结果刷新一次、刷新也失败、确定结果保留、retry 不直接消费、
  owner/command cancellation、close 与迟到响应、source/page 切换。
- renderer：列表说明/日期、cancel-first/back-first focus、确认、进行中、
  失败/Completed 全分支，所有 callback 带正确 snapshot identity。
- 迁入/改写 RpcSettingsTest 的 reset 三个基线测试为 dependency-only 测试，
  同时保留 RPC backend 实际消费与 account validation 回归；
  不能仅靠 mock outcome 证明端到端幂等或消费安全。

## Integrator baseline clarification

- 原 `RpcUsageReset.confirm` 在正常返回 outcome 时不额外调用 refresh；
  只有普通消费异常才调用一次 `refreshAccountUsage`。本次保持该调用次数，
  不因“结果应不被刷新失败抹掉”的约束引入新的成功路径刷新。
- `refreshChoices` 是无明细时显式刷新入口，不得作为消费成功的隐式跟随操作。

## Integration handoff

### Delivery status and ownership

- 组件实现、集中 JVM 编译/测试及宿主整合已完成；用户已验收六组件批次。
- 本 lane 只写新 `app/component/usage-reset` 树及本 child executable；
  未修改旧 source、shared host/tests、central Gradle、其他 component 或 master docs。
- 新树含三个 convention 项目：`app-component-usage-reset-spec`、
  `app-component-usage-reset-impl-viewmodel`、`app-component-usage-reset-impl-view`。
  插件分别为 `kodex.kmp-cli`、`kodex.kmp-viewmodel`、`kodex.kmp-view`。
- 未运行 build/test、commit/push、branch 操作或 IDE/共享设备控制；没有临时文件或后台资源。

### Exact public API

- DTO 保留 `io.github.stream29.kodex.app.settings.contract` FQCN：
  `UsageResetOption(creditId: String?, title: String, description: String, expiresAt: Instant?)`、
  `UsageResetRequest(availableCount: Long, options: List<UsageResetOption>)`、
  `UsageResetState`。既有字段、nullable 语义、校验和 `Completed.selectedCredit` 保留，
  唯一删掉的 branch 为无 producer 的 `Preparing`，不是 wire 变更。
  规范见 [DTO/state](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/usage-reset/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/settings/contract/UsageResetState.kt#L6)。
- Ports / VM / typed factory 位于 `io.github.stream29.kodex.app.usagereset.contract`：
  [UsageResetDependencies](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/usage-reset/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/usagereset/contract/UsageResetViewModel.kt#L15)、
  [UsageResetViewModelFactory](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/usage-reset/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/usagereset/contract/UsageResetViewModel.kt#L54)、
  [UsageResetViewModel](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/usage-reset/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/usagereset/contract/UsageResetViewModel.kt#L86)。

```kotlin
interface UsageResetDependencies {
    val usage: StateFlow<SettingsAccountUsageState>
    suspend fun consume(creditId: String): CodexRateLimitResetOutcome
    suspend fun refreshUsage(): Unit
}
fun interface UsageResetViewModelFactory {
    fun create(dependencies: UsageResetDependencies, ownerScope: CoroutineScope): UsageResetViewModel
}
interface UsageResetViewModel : AutoCloseable {
    val state: StateFlow<UsageResetState>
    fun show()
    fun select(creditId: String)
    fun confirm(expected: UsageResetState.Confirming)
    fun back()
    fun retry()
    fun refreshChoices()
    fun dismiss()
    override fun close()
}
```

- Implementation 位于 `io.github.stream29.kodex.app.usagereset`：
  [DefaultUsageResetViewModelFactory](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/usage-reset/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/app/usagereset/UsageResetViewModel.kt#L21) 实现 typed factory；
  [createUsageResetViewModel(dependencies, ownerScope)](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/usage-reset/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/app/usagereset/UsageResetViewModel.kt#L33)
  返回 spec `UsageResetViewModel`，无构造 I/O。
- Mosaic entrypoints 位于 `io.github.stream29.kodex.cli.usagereset`：
  [BoxScope.UsageResetDialogHost(viewModel: UsageResetViewModel)](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/usage-reset/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/usagereset/UsageResetDialog.kt#L39)；
  [BoxScope.UsageResetDialog](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/usage-reset/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/usagereset/UsageResetDialog.kt#L59)
  的参数准确为 `state: UsageResetState`、`onSelect: (String) -> Unit`、
  `onConfirm: (UsageResetState.Confirming) -> Unit`、`onBack: () -> Unit`、
  `onRetry: () -> Unit`、`onRefresh: () -> Unit`、`onDismiss: () -> Unit`。
- Host 放在 `TuiPopupHost` 内；renderer 全部 visual helpers 来自 `app-view-components`，
  不依赖 `app-view-settings`。卸载 renderer 不 close 可复用 child；最终 close 由 owner 宿主调用。

### Adapter requirements and preserved semantics

- Account lane 独占 `SettingsAccountUsageState` 与 `snapshotOrNull` 的迁入；
  reset spec 直接依赖 `app-component-account-usage-spec` 和原 `openai-spec-account-usage`，
  不声明这两个 usage 类型，不依赖 Global Settings contract 或 RPC。
- Adapter 提供 shared account-isolated usage flow，
  `consume(creditId) = GlobalRpc.consumeUsageReset(creditId)`、
  `refreshUsage() = GlobalRpc.refreshAccountUsage()`；保持后端 command-start 当前账号准入。
- 不传确认期 account、attempt、prepare、结果重放或隐含默认券。
  不把认证/usage 的独立 flow 当作原子快照；账号变化清除旧 fallback 仍是 source/host 责任。
- Account Usage 的 reset intent 调用准确 child 的 `show()`。
  Page/source 离开调用 `dismiss()`，active consume 与其失败后刷新期间无效；
  关闭 Settings 时调用 `close()`，取消 local owner 并立即隐藏，不关闭 shared store。
- 必须让 renderer 的 `confirm(expected)` 直接到 child；不可在父级重读最新
  Confirming 后替代 expected。Back/retry 只建立新选择，不自动 consume。
- 四种正常 outcome 原样 Completed，`selectedCredit=true`，**零额外 frontend refresh**；
  普通消费异常先 ConsumeFailed 后 refresh **一次**。刷新失败/cancellation 保留未知 failure，
  不伪装为已取得最新状态；取消 consume wait 不触发刷新或自动 retry。
- 单独 consume wait cancellation 沿旧语义留下最后状态，job unwind 后外层可
  show/dismiss 恢复；close 则立即 Hidden。Cancellation 不是未消费或远端回滚证明。
- Shared declarations、旧 RpcUsageReset 与旧 dialog branches 删除、全部 hosts/adapters、
  原 RPC/host tests 迁移及 central project registration/builds 均由 main lane 处理。

### Intentional differences and tests

- 全链路 exact confirming：等值 copy、旧 selection、dismiss/reopen 与 duplicate
  callbacks 均不可消费；正常 exact object 只消费一次。
  [VM identity cases](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/usage-reset/impl/viewmodel/src/commonTest/kotlin/io/github/stream29/kodex/app/usagereset/UsageResetViewModelTest.kt#L165) 与
  [renderer identity/expiry/focus cases](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/usage-reset/impl/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/usagereset/UsageResetDialogTest.kt#L108)。
- Picker 与 confirmation 均明确显示 `Expiry unknown`，并显示 description；
  picker 改为 Cancel-first，confirmation 保持 Go back-first。
  无明细/已知空/计数未知均不能 autochoose，PreparationFailed 提供显式 Refresh。
- 防御 malformed 非 wire 输入：blank IDs 从 choices 排除；blank title/description
  使用既有 fallback；重复 ID 仍不可 select。原 DTO 构造校验没有放宽。
- Local lifecycle strengthening：消费 response 后、发布前及失败 follow-up 前检查
  owner 与当前 job；取消/close 后迟到甚至 noncooperative response 不发布 Completed/
  ConsumeFailed，也不发起 follow-up refresh。Choice refresh 使用本地 generation，
  dismiss/reopen 后旧 refresh 不能重新打开 dialog。generation 不是 backend attempt/account token。
- [Noncooperative response tests](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/usage-reset/impl/viewmodel/src/commonTest/kotlin/io/github/stream29/kodex/app/usagereset/UsageResetViewModelTest.kt#L487)
  用 unconfined collector 检查 close 后连瞬间的 stale publication 都没有；
  fake remote acceptance 可以仍发生，明确不声称 remote rollback。
- 两组 suite：`usageResetViewModelTest` 和 `usageResetDialogTest`。
  覆盖 actual details/count/fallback、选券、所有 outcome、失败一次刷新、
  cancel/close/owner 生命周期及 renderer 全 branch、键盘/Escape、准确 callback。
  [Full renderer/real child tests](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/usage-reset/impl/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/usagereset/UsageResetDialogTest.kt#L300)
  包含正常结果零额外刷新、unknown outcome refresh 失败后仍保留 failure，以及 retry 只返回 fresh picker。

### Validation boundary

- 已静态核对新项目依赖方向、公开签名、所有 state renderer branch、KDoc 顺序/
  异常/取消语义及新文件 whitespace；文件级 `git diff --no-index --check` 无诊断。
- 集中 JVM 验证通过：65 项组件测试及共享宿主/RPC/backend/serializer 回归。
  RpcSettingsTest 覆盖准确二次确认、无明细不得消费和未知结果刷新一次；
  Settings 导航测试覆盖稳定 child、离页清理和关闭。Native/JS/CLI 未验证。
- fake dependency tests 不证明供应商 exactly-once、backend account validation 或真实网络行为；
  不将 fake 测试通过描述为真实供应商协议保证。

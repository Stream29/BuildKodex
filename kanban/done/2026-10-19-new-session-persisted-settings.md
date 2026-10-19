# Task Tree

- `Await first-wave Session approval`()
- `Inspect persisted contracts and Settings target consumers`()
- `Remove draft masquerading and provide unavailable source`()
- `Migrate owned Settings component tests`()
- `Review scoped diff and report evidence`()
- `Receive parent acceptance`()

# Details

## 父级最终验收

- 已通过契约/实现审查、Settings/Session 的 JVM 回归、集成及 Native/PTY 的无目标 Settings 检查，任务移入 done；实际证据见父任务 §6.7–6.10。
- 后文“未执行/待验收”为 worker 交付时点，不替代父级最终结果。

## 上下文与授权

- 父任务：[singleton New Session](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-18-singleton-new-session-navigation.md#L1)，重点第 2.1/2.2/6 节；用户授权实现，主会话验收。
- 仓库根 `/home/stream/ACodeSpace/push/BuildKodex`，产品在 `Kodex/`。加载根指导、相关技能、Draft/父任务及 frontend、RPC 设置/状态和 Session checklist。
- 第一波 C，与 A/B 并行但写入不重叠。不提交/推送、不派 worker、不运行 Gradle/Native/IDE Sync 或控制设备；保留他人修改。

## 唯一写范围

- `Kodex/app/session/contract/**` 和 `Kodex/app/session/**`。
- `Kodex/app/component/session-settings/**`、`Kodex/app/component/settings/**`（spec、实现、view、build 与各自测试）。
- `Kodex/app/rpc/src/commonMain/kotlin/io/github/stream29/kodex/cli/rpc/RpcSessionSettings.kt`；可在同包新增独立 unavailable source 文件。
- `app/rpc` commonTest 全归 B；`Application.kt`/popup spec 归 D；根 View 归 E；integration-test 归 F。遇到范围外消费者只报告路径。

## 冻结接口与约束

- 将 name/rename/AgentSettingsViewModel/AutoCloseable 及其 KDoc 合入 `PersistedSessionViewModel`，删除无共同消费者的 `SessionViewModel`。
- 保留 persisted ready binding、rootAgent 重绑定、CAS、close/release 不 Stop backend 的既有契约，不重写 Session registry 生命周期。
- `SettingsViewModelArguments(target: PersistedSessionViewModel?, initialPage: SettingsPage)`；null 表示没有选中真实 Session，不表示回查当前 tab。
- 删除 `SessionSettingsTargetKind.NewSessionDraft` 及单分支枚举，移除 snapshot/UI/tests 中冗余 targetKind。
- 删除 `RpcDraftSettingsSource`；提供同包 `public object UnavailableSessionSettingsDataSource : SessionSettingsDataSource`，只读 state 恒为 Unavailable，tryUpdate/rename 返回 false，close 无操作。
- `SessionSettingsState.Unavailable` 仍展示 `No selected session`；New Session 的 Settings 默认页由 D/E 选择为 NewSession，本任务不改 Application。
- 保留 Settings 默认值、Title generation、OAuth、即时持久化/队列与 exact target 编辑的原语义；默认值没有复制为新的 Settings 状态真源。

## 检查项

- 迁移范围内所有 SessionViewModel/targetKind/draft-source 消费者与测试 fake；不能留下兼容父接口或旧枚举。
- 覆盖 null target 的 unavailable source、不接受编辑和幂等 close；原真实 Session 的 rename、cwd、模型配置、revision/CAS 与关闭取消测试仍有效。
- Settings renderer 不再显示草稿专属命名/配置分支，不把 null 变成可编辑默认 Session。
- 静态检查依赖方向和 diff；测试执行交主会话统一组织。跨目录旧测试即使暂时无法编译也不能越界修改。

## 交付报告（2026-10-19，第一波 C）

- Session ID：955；[storage](file:///home/stream/.kodex/sessions/955)。
- 已按用户本轮实现授权执行；没有冻结接口或写范围阻塞。父会话尚未验收，任务停在 `Await parent acceptance`，未移动 done。

### 实际变更与接口删除

- [PersistedSessionViewModel.kt](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/session/contract/src/specMain/kotlin/io/github/stream29/kodex/app/session/contract/PersistedSessionViewModel.kt#L7)：直接继承 `AgentSettingsViewModel, AutoCloseable`，合入 `name`、`rename`、`close` 及 exact-target、取消与本地释放 KDoc。删除原 `SessionViewModel.kt`，不保留别名、父接口或转发层。
- [SettingsViewModel.kt](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/settings/viewmodel/src/specMain/kotlin/io/github/stream29/kodex/app/settings/contract/SettingsViewModel.kt#L94)：`SettingsViewModelArguments.target` 为 `PersistedSessionViewModel?`；明确 null 永远无真实目标，不能解析后来的选中 tab。初始页仍显式由宿主提供。
- [SessionSettingsViewModel.kt](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-settings/viewmodel/src/specMain/kotlin/io/github/stream29/kodex/app/settings/contract/SessionSettingsViewModel.kt#L11)：删除整个 `SessionSettingsTargetKind`（含 `NewSessionDraft`）与 snapshot 的 `targetKind`；source 契约只描述准确 persisted binding 或无目标。
- [RpcSessionSettings.kt](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/rpc/src/commonMain/kotlin/io/github/stream29/kodex/cli/rpc/RpcSessionSettings.kt#L20)：删除 `RpcDraftSettingsSource`；真实 source 仅移除 snapshot 冗余 kind，保留原 binding 捕获、revision、字段基线/CAS、rename 与取消路径。
- 新增 [UnavailableSessionSettingsDataSource.kt](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/rpc/src/commonMain/kotlin/io/github/stream29/kodex/cli/rpc/UnavailableSessionSettingsDataSource.kt#L10)：public object，同包可供 D 装配。只读 `StateFlow` 恒为 Unavailable，配置更新/rename（包括空名）恒返回 false，close 无操作。
- C 范围内所有 snapshot fake/测试已迁移。生产 renderer 本来未按 kind 分支，无须改写；`No selected session` 原实现保留。未改 Application、popup spec、根 View 或 integration-test。
- 真实 Session 的生产 registry/lifecycle/rootAgent 实现、SessionSettings 生产队列、Settings 根切页/关闭及两个生产 renderer 均未修改。未引入新 defaults authority，也未修改 Title、OAuth 或持久化队列装配。

### 编写和保留的测试（均未执行）

- 新增 [UnavailableSessionSettingsTest.kt](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/settings/viewmodel/src/commonTest/kotlin/io/github/stream29/kodex/app/settings/UnavailableSessionSettingsTest.kt#L17)，三个测试：
  - 无目标 source 对不同 revision 的配置/rename 全部拒绝；state handle 稳定，反复 close 不改变状态。
  - 两个 Settings child 共享静态 source，不创建 Rename/Browse child、不接受四字段编辑；关闭一个不影响另一个，catalog 更新不能将其变为可编辑状态。
  - null arguments + 真正 Settings children 下 CurrentSession 保持 Unavailable；随后打开真实 Session 不改变绑定，New Session defaults 仍经既有全局 authority 更新，Title 与已有 Session 设置不受影响。本测试手动装配 children，不声称验证了尚待 D 修改的 Application factory。
- 迁移 [SessionSettingsComponentViewModelTest.kt](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-settings/viewmodel/src/commonTest/kotlin/io/github/stream29/kodex/app/settings/SessionSettingsComponentViewModelTest.kt#L15) 与 [SessionSettingsContractTest.kt](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-settings/viewmodel/src/commonTest/kotlin/io/github/stream29/kodex/app/settings/contract/SessionSettingsContractTest.kt#L12)：去掉两类目标循环和 kind 参数，保留 catalog/revision、FIFO、exact child、rename、cwd、取消与关闭断言。
- 迁移 [WorkingDirectoryOwnershipTest.kt](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/settings/viewmodel/src/commonTest/kotlin/io/github/stream29/kodex/app/settings/WorkingDirectoryOwnershipTest.kt#L151) 与 [SessionRenameAdapterTest.kt](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/settings/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/settings/SessionRenameAdapterTest.kt#L61) 的 snapshot fake，不改变既有行为覆盖。
- 迁移 [SessionSettingsComponentTest.kt](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-settings/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/sessionsettings/SessionSettingsComponentTest.kt#L28)：仅测试 persisted 可编辑面板；加强 Unavailable 下无 Identity/Model/Rename/Browse、无写入的断言，保留后续可用/关闭的 renderer 覆盖。
- [SessionBindingOwnershipTest.kt](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/session/src/commonTest/kotlin/io/github/stream29/kodex/cli/session/SessionBindingOwnershipTest.kt#L112) 与 [SessionRepositoryViewModelTest.kt](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/session/src/commonTest/kotlin/io/github/stream29/kodex/cli/session/SessionRepositoryViewModelTest.kt#L13) 迁移四处普通创建为 cwd 参数；前者 fixture 在后端 defaults 中选取 mock catalog 的 `test-model`，不再传前端完整 settings。原重绑定、取消 opener、release race、rename 和不 Stop 后端断言保留。
- 其余现有真实 Settings/CAS、关闭取消及 cwd 测试未删减；`app/rpc` commonTest 按分工留给 B。

### 已做静态检查

- 对 C 范围执行 `git diff --check`，无 whitespace 错误；两个新增 Kotlin 文件额外检查尾空白与末尾换行，无问题。
- `rg -n '\b(SessionViewModel|SessionSettingsTargetKind|RpcDraftSettingsSource|NewSessionDraft|targetKind)\b'` 扫描 C 源文件，结果为空。snapshot/Settings arguments 全部调用位置已检查。
- 用 `git show HEAD:<path> | cmp - <path>` 核实五个生产文件逐字节未变：Session `SessionViewModels.kt`、SessionSettings/Settings 的 commonMain VM、`SessionSettingsComponent.kt` 与 `SettingsPopup.kt`。真实 CAS source 的 diff 只有删除 draft class、kind import 和 snapshot 参数。
- 阅读现有模块依赖，Settings spec 继续只依赖准确 persisted contract；Unavailable source 留在 `app/rpc`。未改 build 文件或引入反向生产依赖。
- 只写授权范围和本任务；没有提交/推送、worker、Gradle/Native/IDE Sync、GUI/远端或真实账号/模型操作，没有修改既存 `gradle.properties`。本会话无遗留临时文件或持有资源。

### 未运行验证与范围外消费者

- **所有编译、单测、renderer 测试、Native 与交互验收均未运行**，由父会话统一执行；以上静态检查不是测试通过证明。当前完整工作树仍在分波迁移，不能宣称全库可编译。
- D 尚需迁移的准确文件：
  - [Application.kt:118](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/application/src/commonMain/kotlin/io/github/stream29/kodex/cli/app/Application.kt#L118)：仍装配旧 draft source；null 应直接装配本次提供的 Unavailable object。
  - [ApplicationViewModel.kt（实现）:26](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/application/src/commonMain/kotlin/io/github/stream29/kodex/cli/app/ApplicationViewModel.kt#L26)：仍导入/使用共同父接口，Settings arguments 创建点约在第 240 行。
  - [ApplicationViewModel.kt（契约）:7](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/application/src/specMain/kotlin/io/github/stream29/kodex/app/application/contract/ApplicationViewModel.kt#L7)、[ApplicationNavigationState.kt:3](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/application/src/specMain/kotlin/io/github/stream29/kodex/app/application/contract/ApplicationNavigationState.kt#L3)、[ApplicationPopupState.kt:5](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/application/src/specMain/kotlin/io/github/stream29/kodex/app/application/contract/ApplicationPopupState.kt#L5)：仍为旧目标类型。
- E 尚需迁移的共同父接口消费者：
  - [HistoryRenderState.kt:9](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/HistoryRenderState.kt#L9)、[SessionTabRenderState.kt:9](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/SessionTabRenderState.kt#L9)、[SessionTreeCliScreen.kt:59](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/SessionTreeCliScreen.kt#L59)。
  - [SessionTabBarTest.kt:20](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/app/SessionTabBarTest.kt#L20)、[SessionTabBarTestAdapter.kt:6](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/app/SessionTabBarTestAdapter.kt#L6)、[SettingsLoginLifetimeTest.kt:19](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/app/SettingsLoginLifetimeTest.kt#L19)、[DocsWorkspaceRecordingTest.kt:21](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/view/src/jvmTest/kotlin/io/github/stream29/kodex/cli/app/DocsWorkspaceRecordingTest.kt#L21)。仅需迁移受影响源码，不意味着获准执行录屏/真实场景。
- B 的旧 draft-source 测试尚有 [DraftAndSettingsBoundaryTest.kt:84](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/rpc/src/commonTest/kotlin/io/github/stream29/kodex/cli/rpc/DraftAndSettingsBoundaryTest.kt#L84)（另有 179/268/318 行）和 [RpcSettingsTest.kt:575](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/rpc/src/commonTest/kotlin/io/github/stream29/kodex/cli/rpc/RpcSettingsTest.kt#L575)。这是本次静态扫描时的共享工作树快照，不代表 B 最终交付状态；C 未越界修改。

### 父级检查点

- 主会话已核查本波接口、unavailable source 与测试迁移；允许下游 Application 采用本波接口。编译/单测仍待统一执行，不能移 done。
- B 最终交付已迁移上述旧 draft-source 测试；D/E 的旧共同父接口消费者仍按依赖计划迁移。

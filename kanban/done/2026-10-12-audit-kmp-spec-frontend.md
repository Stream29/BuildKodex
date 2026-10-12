# Task Tree

- `Inventory and review every frontend spec project`()
- `Propose consolidation mapping and identify concrete hazards`()
- `Record source evidence and validation requirements`()

# Details

- Parent: [KMP source-set migration plan](../executable/2026-10-12-discuss-kmp-spec-impl-source-sets.md).
- Scope: all `app` projects in `Kodex/`, including components, roots, migration,
  persistence, views/ViewModels, test support, and CLI consumers.
- Baseline: `4388871137c56bd1b73d5b739cc98f036a790f3c`; verify HEAD before/after review.
- Read [confirmed design](../../checklist/kmp-spec-impl-source-sets.md).
  Target is platform/intermediate -> commonMain (impl) -> specMain.
- Static audit only. No build, product edits, commits, device/IDE control, or new Sessions.
- Only this report is writable by this review line.
- Deliver a complete per-spec inventory, concrete old->new project/source mapping,
  mechanical moves vs decisions, local and cross-domain edges, and file/line evidence.
- Review View vs ViewModel split, dependency cycles, Koin/Mosaic plugins and generated
  code, platforms/source-set hierarchies, tests/resources and task owners.
- Distinguish observed blockers, plausible unverified risks, and checks needed.
  Do not claim compilation or performance results from source inspection.

## Review result

### 结论与证据边界

- 开始核对的 **Kodex 子仓库** HEAD：
  `4388871137c56bd1b73d5b739cc98f036a790f3c`。外层 BuildKodex HEAD 是
  `c0a7d34c4d8a5a7e69538e68d80b56d5fd0bc671`，不是本次产品基线。
- 完整清单是 **91 个 app 构建项目、31 个 spec 项目、72 个 spec commonMain
  Kotlin 文件、12 个 spec commonTest Kotlin 文件**：
  27 个组件 spec，另有 application、session、migration、settings persistence。
  不把本地 `build/` 缓存或生成文件计入源码清单。
- 建议先做 **24 组 spec + ViewModel**，View 暂留独立；另合并 tab-bar
  spec + View、application spec + sidebar spec + application impl、
  persistence spec + filesystem impl。这覆盖 28 个 spec 的候选迁移。
  **agent、session 保留共享 spec；migration 等平台决策后再迁移。**
- 这里的映射、批次和保留建议均为待审方案，不是新确认的设计或实施授权。
  唯一确认的源集形状仍是 `platform/intermediate -> commonMain(impl) -> specMain`；
  不另建 `implMain`，不要求消费者只获得 spec 产物。
- 只读取文件、Git 状态及运行离线文本统计/显式依赖图分析。
  **没有运行 Gradle、编译、测试、IDE Sync、远程操作或性能验证**。
  下文“无环”仅指指定映射下的静态生产项目边，不是构建通过。

### 1. 全部 spec 与旧到新映射

#### 路径与源集约定

- `S(c)` = `:app-component-<c>-spec`，目录 `Kodex/app/component/<c>/spec`。
- `VM(c)` = `:app-component-<c>-impl-viewmodel`，目录
  `Kodex/app/component/<c>/impl/viewmodel`；
  `V(c)` = 同组件的 `:app-component-<c>-impl-view`。
- 下表 `→ VM(c)` 表示：保留这个现有 impl 项目 ID 和目录，将 S(c) 的
  **全部** `src/commonMain/kotlin/**` 移至其 `src/specMain/kotlin/**`，
  package/KDoc/声明不改；VM 的 `commonMain` 原位不动。
  S(c) 原 `commonTest` 合入 VM 的 `commonTest`，测试不搬到 `specMain`。
- 原 S(c) 的 `commonMain.dependencies` 整块迁到目标 `specMain.dependencies`；
  原实现依赖保留在 `commonMain`，自指 `project(S(c))` 删除。
  View 与其他消费者把旧 S(c) 项目引用改为该唯一目标，仍可合法使用实现。
- 表中“文件”完整列出该 spec 的生产 Kotlin 文件名，统一省略 `.kt`；
  保留文件原 package 路径，而不是依据名称重建路径。
  每行 spec 构建链接提供原项目依赖和插件证据。
  不同时安排目录/项目 ID 美化，避免把重命名混入源集合并。

| # | 旧 spec（构建证据） | 全部契约文件 | 唯一候选目标 / 特别事项 |
| --- | --- | --- | --- |
| 1 | [account-usage](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/account-usage/spec/build.gradle.kts#L1) | AccountUsageViewModel；SettingsAccountUsageState | → VM(account-usage)；保留 usage 序列化 DTO |
| 2 | [agent](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/agent/spec/build.gradle.kts#L1) | AgentState；AgentViewModel | 保留 S(agent)；真正 VM 在 app-impl-rpc，不在 V(agent) |
| 3 | [application-preferences](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/application-preferences/spec/build.gradle.kts#L1) | ApplicationPreferencesViewModel；ApplicationPreferencesValues | → VM(application-preferences)；1 个契约测试随迁 |
| 4 | [authentication-settings](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/authentication-settings/spec/build.gradle.kts#L1) | AuthenticationSettingsViewModel；SettingsAuthenticationState；KodexAuthSource | → VM(authentication-settings)；1 个契约测试随迁 |
| 5 | [composer](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/composer/spec/build.gradle.kts#L1) | ComposerViewModel | → VM(composer)；保留 ports、owner ID、提交状态 |
| 6 | [context-source-settings](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/context-source-settings/spec/build.gradle.kts#L1) | ContextSourceSettingsViewModel；BuiltInContextSource | → VM(context-source-settings) |
| 7 | [history](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/spec/build.gradle.kts#L1) | AgentHistoryViewModel；AgentHistorySource；ContextCompactionHistoryItemViewModel；HistoryItemViewModel；MessageHistoryItemViewModel；PatchHistoryItemViewModel；PlanUpdateHistoryItemViewModel；ReasoningHistoryItemViewModel；RequestUserInputHistoryItemViewModel；SuggestSubagentTaskHistoryItemViewModel；ToolHistoryItemViewModel；WorkGroupHistoryItemViewModel | → VM(history)；2 个契约测试随迁；不并 V(history) |
| 8 | [history-index](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history-index/spec/build.gradle.kts#L1) | HistoryIndexViewModel | → VM(history-index)；保留每消费者 read-handle 生命周期 |
| 9 | [hook-settings](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/hook-settings/spec/build.gradle.kts#L1) | HookSettingsViewModel | → VM(hook-settings)；edit handle 仍由适配层提供 |
| 10 | [mcp-settings](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/mcp-settings/spec/build.gradle.kts#L1) | McpSettingsViewModel；McpServerSettingsState | → VM(mcp-settings)；不移动真实 MCP/OAuth 资源所有权 |
| 11 | [new-session](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/new-session/spec/build.gradle.kts#L1) | NewSessionViewModel | → VM(new-session)；继续依赖保留的 app-spec-session |
| 12 | [new-session-defaults](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/new-session-defaults/spec/build.gradle.kts#L1) | NewSessionSettingsViewModel；KodexNewSessionSettings | → VM(new-session-defaults)；2 个契约测试随迁 |
| 13 | [openai-login](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/openai-login/spec/build.gradle.kts#L1) | OpenAiLoginDependencies；OpenAiLoginViewModel；KodexAuthLoginAttempt | → VM(openai-login)；1 个契约测试随迁 |
| 14 | [path-picker](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/path-picker/spec/build.gradle.kts#L1) | DirectoryPickerDependencies；DirectoryPickerEffect；DirectoryPickerState；DirectoryPickerViewModel | → VM(path-picker)；保留 Koin 模块与 linuxX64Test |
| 15 | [request-user-input](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/request-user-input/spec/build.gradle.kts#L1) | RequestUserInputViewModel | → VM(request-user-input)；保留 pending-tool/wire 类型引用 |
| 16 | [runtime-configuration](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/runtime-configuration/spec/build.gradle.kts#L1) | RuntimeConfigurationViewModel | → VM(runtime-configuration) |
| 17 | [session-catalog](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-catalog/spec/build.gradle.kts#L1) | SessionCatalogDependencies；SessionCatalogEntry；SessionCatalogInteractions；SessionCatalogViewModel | → VM(session-catalog)；1 个契约测试随迁 |
| 18 | [session-delete](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-delete/spec/build.gradle.kts#L1) | SessionDeleteViewModel | → VM(session-delete)；普通函数式依赖，不新造 DI 层 |
| 19 | [session-rename](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-rename/spec/build.gradle.kts#L1) | SessionRenameViewModel | → VM(session-rename) |
| 20 | [session-settings](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-settings/spec/build.gradle.kts#L1) | SessionSettingsViewModel | → VM(session-settings)；1 个契约测试随迁 |
| 21 | [session-sidebar](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-sidebar/spec/build.gradle.kts#L1) | SidebarSettingsViewModel | → `:app-impl-application` 的 `src/specMain/kotlin`，与 application 契约同一 owner；V(session-sidebar) 保留 |
| 22 | [session-tab-bar](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-tab-bar/spec/build.gradle.kts#L1) | SessionTabBar | → V(session-tab-bar)/specMain；无独立 VM；1 个契约测试随迁 |
| 23 | [session-title-settings](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-title-settings/spec/build.gradle.kts#L1) | SessionTitleSettingsViewModel；DefaultSessionTitleModel；SessionTitleSettings | → VM(session-title-settings)；1 个契约测试随迁 |
| 24 | [settings](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/settings/spec/build.gradle.kts#L1) | GlobalSettingsViewModel；SettingsViewModel | → VM(settings)；Global 的实现仍在 app-impl-rpc，共享契约只存一份 |
| 25 | [suggest-subagent-task](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/suggest-subagent-task/spec/build.gradle.kts#L1) | SuggestSubagentTaskViewModel | → VM(suggest-subagent-task) |
| 26 | [usage-reset](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/usage-reset/spec/build.gradle.kts#L1) | UsageResetState；UsageResetViewModel | → VM(usage-reset)；依赖唯一 account-usage 契约 |
| 27 | [working-directory](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/working-directory/spec/build.gradle.kts#L1) | WorkingDirectoryViewModel | → VM(working-directory)；与 path-picker 普通组合不合并所有权 |
| 28 | [app-spec-application](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/spec/application/build.gradle.kts#L1) | ApplicationNavigationState；ApplicationPopupState；ApplicationViewModel | → `:app-impl-application`，`Kodex/app/impl/application/src/specMain/kotlin` |
| 29 | [app-spec-session](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/spec/session/build.gradle.kts#L1) | PersistedSessionState；PersistedSessionViewModel；PersistedSessionViewModelRegistry；SessionViewModel | 暂留整个共享项目；不复制到 new-session/session/rpc |
| 30 | [app-migration-spec](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/migration/spec/build.gradle.kts#L1) | KodexHomeHandle；KodexHomeInUseException；KodexHomeVersionException；Migration；MigrationVersion | 条件目标 `:app-migration-impl` 的 specMain；先解决 JS 覆盖；1 个契约测试随迁 |
| 31 | [app-settings-spec-persistence](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/settings/spec/persistence/build.gradle.kts#L1) | BackendSettingsStore；CliFrontendSettingsStore | → `:app-settings-impl-filesystem`，`Kodex/app/settings/impl/filesystem/src/specMain/kotlin`；两个接口共同保留 |

### 2. 全部旧 spec 的显式消费边

- 下表是逐个 spec 的**全部直接项目消费者**，来源于 203 个非外部仓库、
  非 build-logic 的版本化 `build.gradle.kts` 文本扫描；不是仅列 CLI 可达节点。
- `S/VM/V(name)` 沿用上节；`A=app-impl-application`、`R=app-impl-rpc`、
  `P=app-impl-session`、`U=app-impl-view`、`AS=app-spec-application`、
  `PS=app-spec-session`。无前缀的 rpc 项目名按完整 Gradle ID 理解。
- 默认边来自 `commonMain`；`m` 表示 `mosaicMain`，`t` 表示 `commonTest`，
  `mt` 表示 `mosaicTest`，`jt` 表示 `jvmTest`。
  同组件自带 V/VM 的边也列出，以便删除自指而不漏改 View。

| 旧 spec | 全部直接消费者（迁移时重定向） |
| --- | --- |
| account-usage | V(account-usage)、VM(account-usage)、S(settings)、S(usage-reset)、rpc-spec-contract |
| agent | V(agent)、V(history)、VM(new-session)、V(session-sidebar)、R、U(m)、AS、PS |
| application-preferences | V(agent)(m)、V(application-preferences)、VM(application-preferences)、V(composer)(m)、V(new-session)(m)、S(session-sidebar)、S(settings)、U(t,m)、AS、rpc-spec-models |
| authentication-settings | V(authentication-settings)、VM(authentication-settings)、S(settings)、rpc-spec-contract、rpc-spec-models |
| composer | S(agent)、V(composer)、VM(composer)、S(new-session) |
| context-source-settings | V(context-source-settings)、VM(context-source-settings)、S(settings) |
| history | S(agent)、V(history)、VM(history)、U(m) |
| history-index | S(agent)、V(history-index)、VM(history-index) |
| hook-settings | V(hook-settings)、VM(hook-settings)、S(settings) |
| mcp-settings | V(mcp-settings)、VM(mcp-settings)、S(settings)、R |
| new-session | V(new-session)、VM(new-session)、AS |
| new-session-defaults | V(new-session-defaults)、VM(new-session-defaults)、S(settings)、rpc-spec-models |
| openai-login | V(openai-login)、VM(openai-login)、R、AS |
| path-picker | V(path-picker)、VM(path-picker)、S(session-settings)、S(settings)、S(working-directory)、U(m)、AS |
| request-user-input | S(agent)、V(request-user-input)、VM(request-user-input) |
| runtime-configuration | S(agent)、S(new-session)、V(runtime-configuration)、VM(runtime-configuration) |
| session-catalog | V(session-catalog)、VM(session-catalog)、A、P、U(m)、AS、integration-test(jt)、rpc-spec-contract |
| session-delete | S(session-catalog)、V(session-delete)、VM(session-delete)、AS |
| session-rename | V(session-rename)、VM(session-rename)、S(session-settings)、AS |
| session-settings | V(session-settings)、VM(session-settings)、S(settings) |
| session-sidebar | V(session-sidebar)、AS |
| session-tab-bar | V(session-tab-bar) |
| session-title-settings | V(session-title-settings)、VM(session-title-settings)、S(settings)、R、rpc-spec-models |
| settings | V(settings)、VM(settings)、A、R、U(m)、AS、integration-test(jt) |
| suggest-subagent-task | S(agent)、V(suggest-subagent-task)、VM(suggest-subagent-task) |
| usage-reset | S(settings)、V(usage-reset)、VM(usage-reset) |
| working-directory | S(session-settings)、S(settings)、V(working-directory)、VM(working-directory)、AS |
| app-spec-application | A、U(m)、integration-test(jt) |
| app-spec-session | V(agent)(mt)、S(new-session)、S(settings)、A、P、U(m)、AS、integration-test(jt) |
| app-migration-spec | app-migration-impl |
| app-settings-spec-persistence | R、app-settings-impl-filesystem、rpc-impl-server |

#### 不直接引用旧 spec、但必须纳入迁移验收的消费者

- **CLI**：`app-cli` 的 mosaicMain 直接使用 migration impl、root View、
  application impl，见 [CLI 依赖](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/cli/build.gradle.kts#L14)。
  [Main](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/cli/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/Main.kt#L6)
  使用真实 Home preparation；不是新增第二套 frontend 启动链。
- **RPC 适配层 R**：不移动它创建的 child adapters 到契约。
  [R 依赖表](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/rpc/build.gradle.kts#L9)
  消费多数 child VM，公开返回 Agent、GlobalSettings、OAuth 等契约。
- **P、A、U**：
  [P](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/session/build.gradle.kts#L9)、
  [A](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/application/build.gradle.kts#L9)、
  [U](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/view/build.gradle.kts#L20)
  保持现有实现组合；U 不与 A 整体折叠，避免把 root View 测试与应用后端混成一个项目。
- **测试支撑**：
  [app-test-support-rpc](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/test-support/rpc/build.gradle.kts#L6)
  在自己的 commonMain 导出 R、P、VM(new-session)，但本身只供测试消费。
  不因 commonMain 目录名就将它接入生产依赖。
- **共享 UI**：
  [app-view-components](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/view/components/build.gradle.kts#L8)
  独立保留；它的 commonMain 也有 Mosaic runtime API，不是可以随意移入
  全部纯契约项目的工具库。
- **跨领域入口**：
  [rpc-spec-contract:13–15](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/spec/contract/build.gradle.kts#L13)
  使用 auth/usage/catalog DTO；
  [rpc-spec-models:11–14](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/spec/models/build.gradle.kts#L11)
  使用 auth/defaults/title/preferences；
  [rpc-impl-server:15–16](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/server/build.gradle.kts#L15)
  同时使用 persistence spec/impl。
  [integration-test:53–69](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/integration-test/build.gradle.kts#L53)
  的前端消费在 jvmTest，不能误升到 commonTest（该项目另有 JS target）。

### 3. 生产依赖折叠：具体阻塞与可行边界

#### B1：Agent + History 全量组件合并会成环（已证实的候选阻塞）

- 原无环链：`S(agent) -> S(history)`，同时 `V(history) -> S(agent)`。
  前者来自 [Agent spec:14](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/agent/spec/build.gradle.kts#L14)，
  后者来自 [History View:9–10](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/impl/view/build.gradle.kts#L9)。
- 若各组件的 S/VM/V 全折叠，得到 **Agent -> History -> Agent**。
  即便 Agent 仍留独立 spec，只合并 History 的 S/VM/V 也会得到
  **S(agent) -> History combined -> S(agent)**。
- 这不是多余 Gradle 边的纯猜测：
  [AgentViewModel.history](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/agent/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/agent/contract/AgentViewModel.kt#L124)
  真正暴露 History 契约；
  [AgentHistoryView:30、95–98](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/history/AgentHistoryView.kt#L95)
  真正使用 `AgentShellSessionRegistry`。
  不能只删除依赖行而不迁移类型所有权。
- 建议 **S(history)+VM(history)**，View 保留。
  若未来强求 View 合并，需要先决定 shell-session 契约的独立归属，
  或选择更大合并单元；这是额外设计，不能伪装成机械移文件。

#### B2：Session 共享根不能整体塞入 persisted impl（已证实的候选阻塞）

- 原链：`P -> R -> S(settings) -> PS`。
  证据依次是 [P:10–11](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/session/build.gradle.kts#L10)、
  [R:33](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/rpc/build.gradle.kts#L33)、
  [S(settings):21](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/settings/spec/build.gradle.kts#L21)。
  将全部 PS 合入 P 后，变为 **P -> R -> Settings -> P**。
- New Session 的
  [契约](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/new-session/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/session/contract/NewSessionViewModel.kt#L18)
  同时继承 `SessionViewModel`、返回 `PersistedSessionViewModel`；
  [实现](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/new-session/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/cli/newsession/NewSessionViewModel.kt#L14)
  接收 `PersistedSessionViewModelRegistry`。
  它与 [RpcPersistedSessionViewModel](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/session/src/commonMain/kotlin/io/github/stream29/kodex/cli/session/SessionViewModels.kt#L58)
  是共同消费根契约，不是两套可以复制粘贴的 spec。
- 建议 PS 暂留。若后续拆分，要逐声明评估共同 Session、persisted factory/
  registry、New Session 返回类型与 Settings arguments，重跑图后另行决策。
  不把“拆个公共 base”当成已证明足够消环的结论。

#### B3：Agent spec 直接合入 R 同样成环（已证实的候选阻塞）

- 真实 Agent VM 在
  [RpcAgentViewModel:61–78](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/rpc/src/commonMain/kotlin/io/github/stream29/kodex/cli/rpc/RpcAgentViewModel.kt#L61)，
  V(agent) 是 renderer，不是 VM owner。
- R 依赖 S(settings)，后者依赖 PS，PS 依赖 S(agent)：
  [R:31–35](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/rpc/build.gradle.kts#L31)、
  [Settings:21](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/settings/spec/build.gradle.kts#L21)、
  [PS:8](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/spec/session/build.gradle.kts#L8)。
  因此把 S(agent) 全部放进 R 会得到 **R -> Settings -> PS -> R**。
- 暂留 S(agent) 比把 R、Settings、PS、Agent 全并为超大项目更小且明确。
  保留 spec 不是要求所有消费者只依赖 spec；R 和组件 VM 的实现依赖照常合法。

#### 没有阻塞但须保留真实 owner 的情形

- **Sidebar**：真实实现是 A 的
  [SidebarSettingsViewModel:12–23](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/application/src/commonMain/kotlin/io/github/stream29/kodex/cli/app/SidebarSettingsViewModel.kt#L12)，
  绑定 `RpcGlobalSettings`；V(session-sidebar) 没有对应 VM 项目。
  S(sidebar) 与 AS 一起进 A 可去掉内部项目边，View 仍只借用状态。
  不创建空 VM 或把这个应用级对象搬入 renderer。
- **Settings**：VM(settings) 的
  [createSettingsViewModel](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/settings/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/app/settings/SettingsViewModel.kt#L8)
  只组合与关闭 child；同一个 S(settings) 里的 `GlobalSettingsViewModel`
  由 [RpcGlobalEditor:24–40](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/rpc/src/commonMain/kotlin/io/github/stream29/kodex/cli/rpc/RpcGlobalEditor.kt#L24)
  实现。契约统一放 VM(settings)/specMain，R 引用一次即可，不给每个实现复制接口。
- **Defaults**：普通 VM 与
  [RpcNewSessionSettings 的委托包装](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/rpc/src/commonMain/kotlin/io/github/stream29/kodex/cli/rpc/RpcNewSessionSettings.kt#L16)
  共享 `NewSessionSettingsViewModel`。迁移不删除其资源包装，不将其误识别为另一个 spec。
- **组合依赖**：catalog VM 直接组合 delete VM，
  session-settings VM 组合 working-directory/rename VM，都是原来就存在的合法边：
  [catalog:9–11](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-catalog/impl/viewmodel/build.gradle.kts#L9)、
  [session-settings:6–10](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-settings/impl/viewmodel/build.gradle.kts#L6)。
  它们各自 S+VM 合并后仍单向，不必为了“纯 spec”禁止这些组合。

#### 静态图结果与局限

- 对 203 个版本化构建脚本提取 literal `project(":...")`，按 Main/Test
  所在块分开；使用 settings 的扁平 ID 规则（目录 `/` 替换为 `-`），见
  [注册逻辑](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/settings.gradle.kts#L76)。
  对候选 owner 做顶点重定向，删除合并产生的自边，再检查有向环。
- 基线显式生产图无环；本报告 **28 个 spec 重定向方案无生产环**。
  加上 migration 条件合并的 29 个重定向亦未出现生产环，但仍有平台阻塞。
  其他领域保持原 ID；若其他子线也折叠，必须由协调者重算联合图。
- 若把所有组件 S/VM/V 合并并把 PS 合入 P，可得到 SCC：
  `{agent, history}` 与 `{settings, R, P}`。这些反例不能作为全仓已损坏的证据，
  它们阻塞的是特定未来映射。
- 测试边全并入生产图会出现伪阻塞，例如
  `VM(new-session).commonTest -> app-test-support-rpc.commonMain -> VM(new-session)`，
  见 [new-session:17–25](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/new-session/impl/viewmodel/build.gradle.kts#L17)
  与 [fixture:9–12](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/test-support/rpc/build.gradle.kts#L9)。
  这不是两个生产 compilation 互依；测试具体任务图仍待授权后验证。
- 文本图不包含插件生成的 configuration、variant/metadata、传递库依赖、
  自动层次和真实 Gradle task graph。无环不说明这些都兼容。

### 4. View/ViewModel、插件与生成代码

#### View 与 VM 是否合并

- 本轮建议多数不合并：先减少每个组件的独立 spec 项目，同时保留
  View 的 Mosaic/Compose 编译边界和 VM 的通用 host 编译边界。
  不是声称其他 23 组 S/VM/V 必然不能合并；它们可单独评估，History 是已证实反例。
- V(c) 现在多为 commonMain `api(S(c))`、mosaicMain UI 依赖、
  mosaicTest `implementation(VM(c))`；例如
  [account-usage View:5–13](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/account-usage/impl/view/build.gradle.kts#L5)。
  迁后只把 commonMain 的项目引用改为 VM(c)，测试重复依赖可合并，
  不把 VM 的 commonMain 整块挪入 mosaicMain。
- tab-bar 是例外：spec 是 presentation/callbacks，无独立 VM。
  [契约:15–18、55–64](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-tab-bar/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/sessiontabbar/contract/SessionTabBar.kt#L55)
  把状态权留在应用，因此 spec + View 可直接成为一个有意义的迁移单元。

#### Koin 与显式注入

- [kodex.kmp-viewmodel](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/build-logic/src/main/kotlin/kodex.kmp-viewmodel.gradle.kts#L3)
  引入 Koin **compiler plugin**，只向 commonMain 注入 koin-core、koin-annotations，
  并启用 compileSafety/strictSafety/unsafeDslChecks（21–24）。
  这不是 KSP source-generation convention。
- app 版本化源码里显式 Koin 注解入口是
  [PathPickerViewModelModule](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/path-picker/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/app/pathpicker/PathPickerViewModelKoin.kt#L3)，
  使用 `@Module @Configuration @ComponentScan("io.github.stream29.kodex.app.pathpicker")`。
  它保留 commonMain 和原 package；contracts 仍无需 Koin 注解。
  没有发现该模块的版本化生成文件或源码中的直接调用，
  **不据此推断插件可删除或它不会参与生成**。
- 其他 child 实际使用 typed factory/dependencies/ownerScope；例如
  [AccountUsage:13–24](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/account-usage/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/app/accountusage/AccountUsageViewModel.kt#L13)
  与 [RpcGlobalEditor:40–95](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/rpc/src/commonMain/kotlin/io/github/stream29/kodex/cli/rpc/RpcGlobalEditor.kt#L40)。
  切换项目边不能改成全局 service locator，不能重建第二套 scope/写队列。
- **未证实风险**：Koin compiler 对新增 specMain 的扫描、检查及生成时机；
  如果合并 View，则一个项目同时应用 Compose/Koin 插件的实际编译行为。
  本轮没有运行插件，不能声称其失败或已通过。

#### Mosaic、serialization、版本生成

- [kodex.kmp-view](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/build-logic/src/main/kotlin/kodex.kmp-view.gradle.kts#L1)
  应用 Compose compiler 与显式 Mosaic hierarchy；不会自动向新 specMain
  补齐契约库依赖。[Mosaic 分组](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/build-logic/src/main/kotlin/KodexHostKmp.kt#L35)
  包含 JVM、linuxX64、linuxArm64、macosArm64、mingwX64。
- account-usage、application-preferences、authentication-settings、
  new-session-defaults、session-catalog、session-title-settings、settings 原 spec
  应用了 serialization 插件。移入 VM 项目时必须保留该插件及 specMain 所需库；
  不能因为 VM 原先只应用 Koin 就丢掉 DTO serializers。
- [认证 DTO](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/authentication-settings/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/settings/contract/SettingsAuthenticationState.kt#L7)、
  [usage DTO](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/account-usage/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/settings/contract/SettingsAccountUsageState.kt#L6)
  的包名本身参与现有协议。
  [preferences golden assertions](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/application-preferences/spec/src/commonTest/kotlin/io/github/stream29/kodex/cli/settings/ApplicationPreferencesValuesTest.kt#L39)
  校验 descriptor/wire names；移动源码不得连带改名、改枚举 SerialName。
- migration 的
  [生成任务及目录接线](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/migration/impl/build.gradle.kts#L9)
  将版本源目录只接到 commonMain，并使 `KotlinCompilationTask` 依赖生成任务（49–51）。
  [生成内容](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/build-logic/src/main/kotlin/GenerateKodexVersion.kt#L16)
  是 internal const；[实际使用](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/migration/impl/src/commonMain/kotlin/io/github/stream29/kodex/app/migration/KodexHomeMigration.kt#L39)
  在实现中。保留 generator owner 和 commonMain 位置，不把本地 build 缓存复制成 spec。
- 合并时从 convention 提供真正 specMain 和唯一预期边，并使用已确认的
  `kotlin.internal.suppressGradlePluginErrors=CommonMainOrTestWithDependsOnDiagnostic`；
  不在每个项目复制整套 KMP 配置。
  依照 [已确认的 KGP 例外](file:///home/stream/ACodeSpace/push/BuildKodex/checklist/kmp-spec-impl-source-sets.md#L50)，
  该异常只消除选定诊断，不能代替 hierarchy、依赖或 metadata 验证；
  同 ID 还覆盖 commonTest，必须另查没有引入额外测试边。

### 5. 平台差异与契约依赖落点

- [CLI convention](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/build-logic/src/main/kotlin/kodex.kmp-cli.gradle.kts#L8)
  与 VM/View 共用
  [5 个 host targets](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/build-logic/src/main/kotlin/KodexHostKmp.kt#L22)：
  JVM、linuxX64、linuxArm64、macosArm64、mingwX64。
  app 内除 migration spec 外的 spec 均使用该 CLI convention；
  它们与候选 VM/impl 的 target 集合静态一致，不代表层次布局完全相同。
- **B4，migration 平台不相等**：
  [spec 用 kmp-host](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/migration/spec/build.gradle.kts#L2)，
  [host 额外建 JS/Node](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/build-logic/src/main/kotlin/kodex.kmp-host.gradle.kts#L12)；
  [impl 用 kmp-cli](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/migration/impl/build.gradle.kts#L5)，没有 JS。
  直接把 spec 删掉会丢其 JS 契约覆盖；给 impl 加 JS 则让文件系统/租约/
  serialization 实现及生成任务承担原先没有的 compilation。
  **两种都不是保持原平台的机械迁移。** 当前建议保留 spec；
  扩展实现到 JS 或保留独立共享契约须另决策，不擅自裁 JS。
- app 版本化源码未发现真正的 expect/actual 声明或直接 cinterop 配置；
  不能把文字里的 “actual” 当语言声明。
  平台差异具体集中在上述 JS、Mosaic、中间层和
  [path-picker linuxX64Test](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/path-picker/impl/viewmodel/src/linuxX64Test/kotlin/io/github/stream29/kodex/app/pathpicker/DirectoryPickerBrowserLinuxTest.kt#L1)。
  下游工具/文件系统项目内部层次不在本轮迁移，保持原依赖边与目标覆盖。
- 新 specMain 不会反向继承 commonMain 注入库。逐块搬原 spec 的
  API 依赖，例如 [path-picker 的 coroutines/io](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/path-picker/spec/build.gradle.kts#L7)、
  [history 的 storage/state](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/spec/build.gradle.kts#L8)、
  [persistence 的 rpc models](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/settings/spec/persistence/build.gradle.kts#L5)。
  不将 Koin、Mosaic runtime、filesystem impl、RPC 实例等统一塞入 specMain。
- 反向例子：原 spec 里的
  [AgentHistorySource](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/spec/src/commonMain/kotlin/io/github/stream29/kodex/cli/history/AgentHistorySource.kt#L20)
  是有状态源端口的契约输入，不能凭 `cli` 包名移到实现；
  `DefaultSessionTitleModel` 也随其当前公共契约一起迁移，默认值变更另审。

### 6. 测试、资源与重复声明

- 12 个旧 spec 测试文件全部保留：
  `ApplicationPreferencesValuesTest`、`KodexAuthSourceTest`、
  `ExpandableHistoryItemContractTest`、`WorkGroupHistoryItemContractTest`、
  `NewSessionDefaultsContractTest`、`KodexNewSessionSettingsTest`、
  `OpenAiLoginContractTest`、`SessionCatalogContractTest`、
  `SessionSettingsContractTest`、`SessionTabBarContractTest`、
  `SessionTitleSettingsTest`、`MigrationVersionTest`。
  最后一项等 migration 决策；其余入各目标 commonTest。
- [test convention](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/build-logic/src/main/kotlin/kodex.kmp-tests.gradle.kts#L1)
  给 commonTest 注入 Kotlin test/TestBalloon。
  合并测试时保留原 spec 额外的 serialization-json 依赖；
  保留 View 的 mosaicTest、VM 的 commonTest、linuxX64Test 以及 impl 的 jvmTest，
  不为了统一目录把它们改成 commonTest。
- 对所提映射的源路径做了静态重名检查：未见两个输入要落入同一目标测试/
  资源路径。生产文件即使同名，也分别落 specMain/commonMain，
  例如 account-usage 两侧都叫 `AccountUsageViewModel.kt`。
  不能把两个目录直接摊平为同一 commonMain。
- 对相同 package + filename 的顶层函数/属性做候选检查时，排除了
  `fun interface`（它不等于顶层函数）；未发现推荐映射中的已证实 JVM facade
  重复。此检查不是完整 Kotlin 符号解析，不声称 ABI/编译器检查通过。
  合并后 `internal` 可见范围变大、同包 private/重载查找及 sealed 层次仍应检查。
- History 的
  [sealed item 契约](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/history/contract/item/HistoryItemViewModel.kt#L14)
  与其 12 个契约文件作为整体迁移，不把直接子类型任意分散到其他 compilation。
- app 的唯一版本化非 Kotlin `src` 资源为
  [jvmTest docs/history-index.en.json](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/view/src/jvmTest/resources/docs/history-index.en.json#L1)，
  由 [DocsHistoryRecordingTest:68](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/view/src/jvmTest/kotlin/io/github/stream29/kodex/cli/app/DocsHistoryRecordingTest.kt#L68)
  从 classloader 读取。root View 保留，所以无需挪这个资源；迁移验收无需录制文档或跑性能探针。
- migration 必须保留
  [历史测试源码排除和 test filter](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/migration/impl/build.gradle.kts#L36)，
  不因合并 commonTest 重新激活冻结的旧 combined-store 测试。
  当前替代回归是 `MigrateToV047CurrentStoresTest` 与
  `KodexHomeMigrationCheckpointTest`；不能把未运行或被排除测试写成通过。
- persistence 的
  [BackendSettingsStore CAS 契约](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/settings/spec/persistence/src/commonMain/kotlin/io/github/stream29/kodex/cli/settings/BackendSettingsStore.kt#L67)
  与
  [CliFrontendSettingsStore 的分文件保证](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/settings/spec/persistence/src/commonMain/kotlin/io/github/stream29/kodex/cli/settings/CliFrontendSettingsStore.kt#L9)
  保持不变；沿用 `BackendSettingsCasTest`、`SplitSettingsStoreTest`、
  `SettingsCompatibilityEvidenceTest`、`SettingsStoreInterfaceTest`、jvmTest
  `KodexSettingsPermissionsTest`，不重新整合 backend/frontend 两个 store。

### 7. 待审批次与检查责任

| 批次（未授权执行） | 建议对象 | 静态机械部分 | 必须确认/验证的部分 |
| --- | --- | --- | --- |
| F0 convention | 构建约定 owner | 新 specMain；原契约依赖移层；唯一 commonMain→specMain；精确诊断抑制 | 保持默认/显式层次、五个平台与测试边；不要广泛关闭诊断 |
| F1 小组件 | session-delete、session-rename、runtime-configuration，再 path-picker/openai-login/working-directory | 单组 S+VM，改 V 与调用者项目边 | typed factory、Koin 扫描、io 与平台测试；不能只看 JVM |
| F2 DTO 共享组件 | account-usage、authentication-settings、application-preferences、new-session-defaults、session-title-settings、session-catalog | 保留 serialization 插件、package/wire shape，重定向 rpc-spec-models/contract | 前端与协议子线共同验收；先 leaf delete 再 catalog |
| F3 组合与较大 VM | composer、history-index、request-user-input、suggest-subagent-task、history、context-source-settings、hook-settings、mcp-settings、usage-reset、session-settings、settings、new-session | S+VM，保留 V；按依赖先 leaf 后组合者 | history 不并 View；PS/S(agent) 留存；R/P、fixtures 回归与生命周期 |
| F4 特殊 owner | tab-bar、sidebar+application、persistence+filesystem | 按表唯一 owner 合并，去自指；改 U/CLI/integration/server 消费边 | sidebar 与 application 同批，避免临时 AS↔A 环；跨领域全图重算 |
| F5 阻塞项 | agent、session、migration | 不作机械删除 | 前两者共享契约/环须另设计；migration 先决策 JS 覆盖 |

- F1→F4 是依赖顺序建议而非自动执行脚本；每个批次仍需单独授权。
  不对没有性能测量的数据承诺 Sync/内存改善。
- **前端子线**负责 factory/ownerScope、渲染借用与 Close、全部 V/VM/P/A/U/CLI
  调用者；**协议子线**负责 DTO/kRPC consumer 与 serialization；
  **构建约定 owner/协调者**负责 convention、联合生产图、平台覆盖及测试任务路径。
  persistence/server 与 migration/filesystem 依赖需要和对应子线协同。
- 授权后的验证清单：无抑制/精确抑制诊断对照；契约依赖与生成符号可见性；
  JVM 和原 Native/JS 覆盖；实际 metadata/variant；契约/VM/View/CLI/fixture
  consumer 编译测试；原测试 exclusion/资源路径；无遗留旧项目引用。
  这些都是**待做检查**，不是本报告测试结果。

### 8. 分类汇总与交付核对

- **机械迁移候选**：24 组 S+VM，tab-bar S+View，application+sidebar、
  persistence；按表保留单一契约来源、原 package、依赖所在层和测试范围。
- **需要决策**：是否进一步合并 View；共享 Agent/Session 契约所有权；
  Sidebar 与 application 同批；迁移 convention 的具体落点；migration 的 JS 策略。
- **真实静态阻塞（针对候选，不是现状故障）**：B1 Agent/History 折叠环，
  B2 PS→P 折叠环，B3 S(agent)→R 折叠环，B4 migration spec/impl 平台不等。
- **未证实风险**：Koin/Compose/serialization 在新源集上的编译与生成行为、
  metadata/variant/IDE 识别、internal/ABI 变化、完整测试任务图。
  没有用“可能”替代上面可指认源码边的真实阻塞，也没有把插件风险写成既成故障。
- 收尾复核：Kodex HEAD 仍为
  `4388871137c56bd1b73d5b739cc98f036a790f3c`，其 `git status --short` 为空。
  外层 HEAD 仍为 `c0a7d34c4d8a5a7e69538e68d80b56d5fd0bc671`。
  本子线仅编辑此报告；既有主任务、其他报告和其他工作区变动未触碰。
- 文档核查完成：31 行清单逐行比对实际 72 个契约文件名无遗漏；
  12 个契约测试名核对一致；文件 URI 及所引行号存在；文本重名扫描和
  显式生产图结果已分开陈述。离线脚本通过标准输入运行，无临时脚本/
  输出文件或设备资源需要清理。
- 此处完成的是静态审查子任务。归档到 done 供主任务汇总，不推进产品迁移，
  不把本报告的完成标记当作 F0–F5 执行授权。

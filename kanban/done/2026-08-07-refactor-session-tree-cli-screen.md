# Task Tree

- [done] 完成 CLI 全层级 contract/ViewModel 迁移并收敛 `SessionTreeCliScreen`
  - [done] 重新盘点当前实现
    - [done] 盘点 application、Session、Agent 与 New Session 状态所有权
    - [done] 盘点顶层订阅、直接 runtime 访问与回调链
    - [done] 对照既有 ViewModel、frontend 与 lazy history 约束
    - [done] 确认 IntelliJ 与当前未提交改动
  - [done] 确认迁移范围
    - [done] 采用完整 ViewModel 分层迁移
    - [done] 纳入状态所有权、按需 Agent ViewModel 与 frontend 迁移
    - [done] 保持 runtime、storage 与产品交互语义不变
    - [done] 确定兼容适配后删除旧 API 的实施顺序
  - [done] 确认模块与 DI 边界
    - [done] 按层建立 `app/contract`、`app/viewmodel` 与 `app/view` 领域模块
    - [done] 将三个层级根目录保持为纯目录
    - [done] 让 frontend 只依赖 contract，并将具体实现保持为 internal
    - [done] 纳入全部 ViewModel 与辅助 ViewModel
    - [done] 建立独立 composition root，并让 launcher 只负责 host
    - [done] 采用 Koin Compiler Plugin 与 annotations，不引入已弃用的 KSP processor
  - [done] 修订模块依赖 DAG 与最终 contract
    - [done] [提取共享 ViewModel contracts 并归位应用模块](../done/2026-08-10-extract-viewmodel-contracts.md)
    - [done] [让 frontend 直接消费父子 ViewModel contract](../done/2026-08-11-direct-parent-child-viewmodel-contracts.md)
    - [done] 固定 application、Session、Agent、history、New Session 与 login 的单向 contract 依赖
    - [done] 只在 contract 暴露 owning state、命令、address、稳定 child handle 与 factory port
    - [done] 禁止 contract 依赖 Koin、具体 runtime/repository 实现或 frontend 类型
    - [done] 让动态 child ViewModel 通过注入的 factory contract 创建，禁止父 ViewModel 访问全局 Koin
    - [done] 固定 application、Session、Agent、New Session 与 login 的 scope、identity 和关闭顺序
  - [done] 建立迁移回归基线
    - [done] 固定标签、Session browser、设置、重命名和删除行为
    - [done] 固定 Agent 选择、history revert/fork 与 Shell menu 行为
    - [done] 固定 New Session 物化、cwd、运行指示器与 terminal title 行为
    - [done] 为 ViewModel 创建数和 owning Flow emission 增加可观测断言
  - [done] 建立 compile-safe Koin 基础
    - [done] 在 version catalog 中加入 Koin Compiler Plugin 与 `koin-annotations`
    - [done] 为所有 viewmodel 模块和 composition root 聚合模块应用原生 K2 compiler plugin
    - [done] 用共享 convention plugin 统一 KMP、Koin annotations 与 compiler safety 配置
    - [done] 用 `@Module`、`@ComponentScan` 与 `@Configuration` 声明领域和聚合模块
    - [done] 用 `@Factory`、`@InjectedParam` 与准确 binding 描述当前图定义
    - [done] 不把非 AndroidX 的共享 ViewModel 标注为 `@KoinViewModel`
    - [done] 用 `@KoinApplication` 与 typed `koinApplication<T>()` 建立隔离的应用图
    - [done] 在聚合模块启用完整跨模块 compile safety，并验证 Linux X64 与 JVM 目标
  - [done] 建立领域模块与装配模块
    - [done] [以 KMP renderer source sets 统一前端 view](../done/2026-08-11-share-frontend-view-source-sets.md)
    - [done] 在 `app/contract`、`app/viewmodel` 与 `app/view` 下建立对应领域模块
    - [done] 将 settings/login 合并到同领域的 contract、viewmodel 与 view 模块
    - [done] 保持层级根目录无 `build.gradle.kts`，禁止保留重导出的聚合入口
    - [done] 让 `app/cli` 只保留 launcher，并将 composition root 迁出 frontend
    - [done] 让 UI 生产源码只依赖 contracts，让 composition root 依赖实现与基础设施
  - [done] 迁移全部共享 ViewModel
    - [done] 迁移 `ComposerViewModel` 与 `RequestUserInputViewModel`
    - [done] 迁移 `AgentRuntimeViewModel` 与 `AgentHistoryViewModel`
    - [done] 迁移 `SessionRepositoryViewModel` 与 `RootSessionViewModel`
    - [done] 迁移 `NewSessionViewModel` 与 application ViewModel
    - [done] 迁移 `OpenAiLoginViewModel`
    - [done] 删除未使用的公开 contract factory 与旧顶层创建入口
  - [done] 重构 per-Agent ViewModel
    - [done] 将完整 settings 真源、execution、token、stream、interaction 与 failure 拆为独立 flow
    - [done] 让 Agent ViewModel 持有 composer、request-user-input 与 history ViewModel
    - [done] 暴露 history capability、Shell registry 与精确 Agent 命令
    - [done] 从 CLI 调用面移除对 `session.runtime`、storage 和 client 的直接访问
    - [done] 保留有限 history window、generation 与 storage cache 语义
  - [done] 重构 per-Session ViewModel
    - [done] 只在 Session 发布 summary、topology、lifecycle 与稳定 selected Agent handle
    - [done] 用轻量 topology slot 取代携带完整子 ViewModel 的 tree entry
    - [done] 常驻 root Agent ViewModel 并按 address 复用 materialized registry
    - [done] 展开节点只 materialize direct children，深层选择只 materialize 所需路径
    - [done] 用轻量 runtime observation 更新未 materialize 节点状态
    - [done] 删除 `renderRevision` 与子状态扁平复制
  - [done] 重构 repository、application 与 New Session 边界
    - [done] [让 New Session 直接 materialize persisted child](../done/2026-08-11-direct-new-session-materialization.md)
    - [done] [用列表下标表达 Session 标签页导航](../done/2026-08-11-index-session-tabs.md)
    - [done] [提取惰性 Session Catalog ViewModel](../done/2026-08-11-extract-session-catalog-viewmodel.md)
    - [done] [移除 Application 根级数据源聚合](../done/2026-08-11-remove-application-root-data-sources.md)
    - [done] [删除 Application notification 并修正 popup contract](../done/2026-08-12-remove-application-notification.md)
    - [done] [删除未接入的 Application lifecycle contract](../done/2026-08-12-remove-application-lifecycle-state.md)
    - [done] [将 fork 行为归还具体 Persisted Session](../done/2026-08-12-move-fork-to-session-viewmodel.md)
    - [done] [用 tab index 定位 New Session materialization](../done/2026-08-12-index-new-session-materialization.md)
    - [done] 将惰性 Session catalog child 与 opened Session ViewModel registry 分离
    - [done] 让 application navigation 原子持有 `List<SessionViewModel>` 与 `selectedIndex`
    - [done] 分离 popup-scoped catalog child、navigation、独占 popup 与 shutdown boundary
    - [done] 让 settings、models 与 auth 真源只注入准确 child，不从 Application 暴露
    - [done] 移除 `selectedTree`、`activeNewSession()` 与复制的 New Session 名称
    - [done] 让 New Session 只持有进程内完整 settings 与 composer，并由 `materialize()` 直接返回 persisted child
    - [done] 由 Application 按 `tabIndex` 解析 exact New Session child，并用一次 navigation update 原位替换
    - [done] 将 fork 归准确 Persisted Session，并让 Application 只处理 registry/navigation command
  - [done] 迁移 Mosaic frontend
    - [done] 让 application、Session、Agent 与 New Session composable 直接消费准确 child 并 collect 所需 Flow
    - [done] 让业务动作由准确 owning ViewModel 直接处理
    - [done] 让 frontend 直接渲染 Application popup 中的准确 child ViewModel
    - [done] 将 sidebar expansion、history viewport、focus、dropdown 与 popup anchor 留在 frontend
    - [done] 将 settings、Agent runtime、sidebar、tab 与 history presentation 拆到准确 view 模块
    - [done] 用 Session summary 驱动标签、侧栏、spinner 与 terminal title
    - [done] 让 popup 使用 exact open handle 与显式 owner identity，并在 owner 失效时关闭
  - [done] 删除迁移适配层
    - [done] 删除 aggregate `SessionTreeCliState` 与 `RootSessionViewState`
    - [done] 删除旧 selected-routing wrapper 和重复 capability 推导
    - [done] 删除 frontend 对 shared runtime/storage handle 的依赖
    - [done] 清理失效模型、imports、tests、模块依赖与旧目录
  - [done] 完成分层验证
    - [done] 编译验证每个 contract 不可见具体 ViewModel 和 Koin API
    - [done] 编译验证每个 viewmodel module 的本地图和 composition root 完整图
    - [done] 验证动态 factory 参数、scope identity、复用、关闭与失败清理
    - [done] 验证多 Session selection、draft、registry 与生命周期隔离
    - [done] 验证大量 descendants 只立即创建 root Agent ViewModel 和有限 root history
    - [done] 验证展开、深层选择、切换与关闭时的 materialize/复用/清理
    - [done] 验证高频 composer、stream 与 Agent 状态不发布无关上层 state
    - [done] 验证 history window、revert/fork、request-user-input 与 Shell session 回归
    - [done] 验证关键 composable 可跳过且无整屏高频重组
    - [done] 运行相关 viewmodel、view Linux X64 测试、IDE 检查、CLI 链接与终端 smoke
  - [done] 合并并完成最终修改计划
    - [done] 合并 2026-07-26 与 2026-08-07 的重叠规划
    - [done] 纳入 contract/viewmodel 模块与 compile-safe Koin 改造
    - [done] 完成依赖 DAG、迁移批次、兼容边界与验证命令
  - [done] 获得用户授权进入 executable

# Details

- 状态：`done`；正式 contract 已取代 session-v2、旧 New Session contract 与旧 aggregate API；全部 ViewModel 实现、Mosaic frontend、Koin composition root 和 `app/cli` launcher 已接入。
- 专项可观测性、规模、Compose 编译报告、分层构建和真实 CLI 启动验证均已完成。
- 用户已确认选择完整分层迁移，不采用“仅拆文件”或“只整理当前组合层”的窄方案。
- 本任务已合并并取代原 2026-07-26 多 Session 与 ViewModel 规划；后续只在本文件维护状态，不再保留两份并行规划。
- 本任务重新定基线于当前 `Kodex/app/viewmodel/*` 与 `Kodex/app/view/*` 实现，不沿用旧 `SessionManager`/`CodexCliViewModel` 路径。
- IntelliJ IDEA 正在打开本项目；原活动文件现位于 `Kodex/app/view/application/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/SessionTreeCliScreen.kt`。
- `Kodex/` 与根仓库均有用户未提交改动；实施以当前工作树为基线，不回退或覆盖既有改动。

## 已确认的模块与 DI 决策

- 相关模块统一按代码归属分层：`app/contract/<domain>`、`app/viewmodel/<domain>` 与 `app/view/<domain>`；三个层级根目录不保留 `build.gradle.kts` 或 `src/`。
- 领域 view 使用 KMP `mosaicMain` 与未来的 `desktopMain` 表达 renderer 差异；`app/cli` 与未来的 `app/desktop` 只保留入口和 host 生命周期。
- 迁移范围覆盖已归位至 `app/viewmodel` 的全部 ViewModel，包括 application、Session、Agent、history、New Session、composer、request-user-input 与 settings login。
- Contract 模块只公开接口、不可变 owning state、命令、address、稳定 child handle、factory port 与生命周期 contract，不依赖 Koin 或任何具体 ViewModel。
- Viewmodel 模块依赖自身 contract 和必要的下层 contract；业务实现保持 internal，少量跨实现模块的 composition bridge/factory 为 public，但 frontend 不可见。
- Parent ViewModel 只接收 child contract 和 factory contract；禁止直接依赖 child viewmodel module、直接调用构造函数或通过 `getKoin()` 形成 service locator。
- `KodexApplication` 是唯一 composition root；`app/cli` 只负责 launcher/host，frontend Gradle 生产源码只依赖 contracts。
- Koin scope 只负责依赖装配与对象生命周期；application navigation、registry、selected index 和原子状态迁移仍由对应 ViewModel contract 建模。
- Agent 与 `SessionViewModel` contract 直接暴露 `StateFlow<KodexAgentSettings>`；frontend 只通过按字段命令更新最新快照，不建立 configuration、draft、owner 或 availability 投影。
- New Session 以进程内 `MutableStateFlow<KodexAgentSettings>` 实现相同 contract；`materialize()` 捕获当前 settings/composer 并直接返回 persisted child，不发布额外物化状态。

## Koin 接入结果

- Version catalog 使用 Koin 4.2.2、Koin Compiler Plugin 1.0.1 与 `koin-annotations`。
- `kodex.kmp-viewmodel` 统一应用原生 K2 compiler plugin，并开启 `compileSafety`、`strictSafety` 与 `unsafeDslChecks`。
- Contract 模块不应用 Koin；每个 ViewModel library module用 `@Module`、`@Configuration`、`@ComponentScan`、`@Factory` 和 `@InjectedParam` 生成跨模块 hints。
- 共享 ViewModel 不是 AndroidX ViewModel，不使用 `@KoinViewModel`。
- `KodexApplication` 使用 `@KoinApplication` 与 typed `koinApplication<KodexKoinApplication>()` 建立隔离 container，并注入进程路径、repository、scope、auth 与 client 等动态值。
- Linux X64/JVM 完整图在 strict safety 下强制重编译通过。
- 官方依据见 [Koin Compiler Plugin Setup](https://insert-koin.io/docs/setup/compiler-plugin/) 与 [Koin Annotations](https://insert-koin.io/docs/reference/koin-annotations/start/)。

## 已解决的迁移问题

- `SessionTreeCliScreen.kt` 已从约 1,544 行收敛到 531 行，只 collect application shell 和当前 selected child 所需的 contract state。
- Application 只发布 navigation 与 popup；Session、Agent、history、Settings、path picker 与 login 状态均归准确 child。
- CLI frontend 不再访问 raw runtime、storage、repository、client 或 Koin。
- Root Agent 常驻；已发现 subagent 只保留轻量 topology，按展开或选择 materialize 并复用 registry。
- Session summary/topology 仅在语义快照变化时递增 revision 并发布；轻量 runtime observer 只观察 phase、running、latest index、thread name 与 child catalog。
- Composer edit 与 output delta 留在稳定的 child-owned flow，不重发 Session summary/topology、Agent stream wrapper 或 Application navigation/popup。
- 旧 aggregate state、selected-routing wrapper、`renderRevision`、`SessionSurfaceKey`、materialization/update state 与旧模块目录均已删除。
- Directory picker 直接创建并渲染短生命周期 `DirectoryPickerViewModel`，不再保留 factory contract 或实现工厂。
- `app/cli` 现在只包含 launcher 和 coroutine failure logging，native executable 由顶层模块直接生成。

## 本轮验证记录

- 使用 OpenJDK 26.0.2；IDE Project SDK 与两个 Gradle JVM 配置均为 `openjdk-26`，旧 JDK 21 daemon 已停止。
- Koin strict-safety Linux X64/JVM 完整图以 `--rerun-tasks` 强制重编译通过。
- 全部 `app-contract-*`、`app-viewmodel-*` 与 `app-view-*` Linux X64 test 任务通过；Session 与 Application 的 JVM tests 也通过。
- 大规模 fixture 包含 32 个 direct children、每个 child 的一个 grandchild 和 96 条 root history；初始只创建一个 root Agent/history ViewModel，history window 保持 64 条。
- 展开只 materialize direct children；深层选择只 materialize exact path；测试同时断言 address、实例复用和关闭释放。
- 128 次 composer edit 只发布 composer state；64 次 stream delta 留在稳定 `SharedFlow`，不发布 Session summary/topology 或 Agent stream wrapper；child rename/edit 也不重发 Application navigation/popup。
- Compose compiler 报告确认 `SessionTreeCliScreen`、tab bar、sidebar、Agent runtime 与 history 边界均为 `restartable skippable`，并启用 Strong Skipping；Application 为 63/72 skippable，history 为 197/201 skippable。
- `:app-cli:linkDebugExecutableLinuxX64` 通过，产物为 `Kodex/app/cli/build/bin/linuxX64/debugExecutable/kodex-cli.kexe`。
- 隔离 HOME 的 PTY smoke 成功渲染初始 `0 sessions (0 running)` 画面；限时 INT 结束前没有出现 `Required value was null`、未捕获 Kotlin 异常或启动失败。
- IDEA targeted build 通过；IDE inspection 对 KMP generated/dependency source 仍有索引误报，Gradle 双目标编译为权威结果。
- `git diff --check` 与 contract/frontend 边界扫描通过；只保留既有 Native cross-compilation、`kotlin.native.cacheKind` 与 JDK 26 降级 JVM target 25 警告。

## 目标所有权

- Application ViewModel 只拥有 opened Session/New Session registry、tab navigation、当前独占 popup、registry command 和 shutdown boundary；Session catalog child 只在对应 popup 打开时创建。
- Settings、models 与 authentication 真源由实现层注入 Global Settings、New Session、Agent、Login 等准确 child；Application contract 不暴露或代理这些数据。
- Session ViewModel 只拥有一个 root Session 的 summary、轻量 Agent topology、selected Agent handle、materialized Agent registry、fork command 和 Session 生命周期。
- Agent ViewModel 只拥有一个 Agent 的完整 settings 真源、execution、composer、history、stream/pending steer、blocking interaction、Shell registry 与精确命令。
- New Session ViewModel 保留每个虚拟 tab 的进程内完整 settings 与 composer；`materialize()` 直接返回真实 Session，Application 以 `tabIndex` 定位并原子编排原位替换。
- Frontend 保留布局、hover、展开意图、滚动、焦点、dropdown、popup anchor、菜单以及 renderer 专用的 popup/dialog presentation。
- Application popup state 表达无框架的独占 surface、exact open handle 与准确 child ViewModel；不使用 `content + requestId` 消息模型，frontend 不建立第二份 route authority。
- 不建立全局 callback bus、通用 action registry 或把 Mosaic 类型放进 shared ViewModel。

## 迁移方式

- 先在旧 aggregate API 旁建立 owning state 与稳定 child handle，让 implementation 和 frontend 可逐层迁移并保持可编译。
- `RootSessionViewModel` 继续使用现有 recursive `KodexAgentSession` tree 发现执行节点，但把“发现 Session handle”和“创建详细 Agent/history ViewModel”分开；本任务不新增 storage schema 或第二套 topology repository。
- Topology slot 只保存 address、parent、depth、轻量名称/活动/执行摘要和 materialization 状态，不内嵌子 ViewModel mutable state。
- Agent address 使用 root `sessionIndex` 与 Agent identity 的组合；registry、selected handle、popup 和异步 completion 不用当前 active target 反查 owner。
- Root Agent ViewModel 在 Session 打开时创建；其他 Agent 只在展开 direct children、选择路径或明确打开时创建，且同一 address 始终复用同一实例。
- `AgentHistoryViewModel` 归入对应 Agent ViewModel 生命周期；现有有限 window、tail-first 加载、generation invalidation 和 raw cache 复用保持不变。
- Session summary 分开表达 root running 与 aggregate running；标签和 terminal title 继续使用 root-only 语义，侧栏继续显示每个 Agent 自身状态。
- Agent ViewModel 直接转发 Unified Exec 原始 `activeSessions` 真源，不建立第二份 completion-filtered Shell 状态。
- Frontend 完成迁移后一次性删除 aggregate adapter、`renderRevision`、selected-only wrapper 和 raw runtime 访问，避免长期保留两套 authority。

## 预计文件范围

- 构建配置：`Kodex/gradle/libs.versions.toml`、`Kodex/buildSrc/` 的 Koin compiler convention 与相关模块依赖。
- Contract：`Kodex/app/contract/{agent,history,session,session-catalog,application,settings,path-picker}/`。
- ViewModel：`Kodex/app/viewmodel/{agent,history,session,new-session,application,settings,path-picker}/`，login 实现归 settings 领域模块。
- View：`Kodex/app/view/{agent,history,session,new-session,application,settings,path-picker,patch,components}/` 的 renderer source sets。
- 共享装配：`app/viewmodel/application` 中的 `KodexApplication` composition root、Koin application/configuration modules、测试 bindings 与完整图验证。
- CLI launcher/application：`app/cli` entrypoint 与 `app/view/application` 中的 `SessionTreeCliScreen.kt`、`SessionTabBar.kt`、`SessionAgentSidebar.kt`、`AgentRuntimeScreen.kt`、`RuntimeStatusBar.kt` 及 popup presentation。
- 实施时更新 `checklist/cli-session-view-models.md`、`checklist/cli-view-model-state.md` 与必要的 frontend 约束，使其以当前 recursive `KodexAgentSession` 架构描述最终 contract。

## 验证边界

- Shared 测试使用既有 in-memory Session、真实 state flow 和确定性 test client；不新增 mocking framework。
- Materialization 测试已验证 root/child 的 loaded/unloaded 状态、准确创建/关闭计数、稳定复用和按路径惰性创建。
- State 测试记录 Flow emission，证明 composer edit、stream delta、Shell activity 和 topology 更新只触发相应消费者。
- Frontend 测试覆盖 owner 切换后 popup 自动失效、旧 open handle 无法关闭新 popup，并证明业务命令不再依赖多跳 callback 或当前 active target 的迟绑定。
- Koin compiler 验证覆盖每个 viewmodel library module 的本地图、composition root 聚合图、typed startup 与跨模块 hints。
- 主要 Gradle 验证覆盖所有 `app-contract-*`、`app-viewmodel-*`、`app-view-*`、composition root、CLI launcher 及相关 Linux X64 tests。
- 适用时补充 JVM tests；Gradle 与 IDE 验证统一使用 JDK 26。
- 使用 IntelliJ 检查改动文件、Compose compiler metrics/重组计数、`git diff --check` 和真实终端交互验证。

## 非目标与依赖

- 不改变 AgentRuntime、Responses、tool/hook、storage、fork/revert 数据语义或 Session 持久化格式。
- 不改变当前标签、侧栏、settings、history、cwd、spinner、terminal title 或 Shell session 的产品行为。
- 不在本任务实现 `kanban/planning/2026-08-07-fix-history-follow-latest-intent.md` 的行为修复；两项任务按最终 Agent address 与 frontend-local state contract 重新对齐。
- 不在本任务实现 text input 高级编辑，也不设计 Desktop、split pane 或新视觉形态。
- 不以行数作为唯一验收；`SessionTreeCliScreen` 的验收标准是只保留 application shell、尺寸计算、selected index 路由和顶层 overlay host。

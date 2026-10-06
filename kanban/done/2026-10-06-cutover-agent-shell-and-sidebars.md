# Task Tree

- `Trace Agent, Shell presentation, sidebars, and current child lifetimes`()
- `Specify one Agent ViewModel and direct presentation contracts`()
- `Receive review of Agent ablation and sidebar ownership`()
- `Receive implementation authorization`()
- `Move original declarations and implement the captured-owner composition`()
- `Cut over all renderer entrypoints and remove replaced layers`()
- `Compare original UI and verify binding replacement and disposal`()
- `Return evidence for whole-batch acceptance`()
- `Receive scoped acceptance with the parent batch`()

# Details

## Actual sources and approved ablation

- [Parent plan](2026-10-06-hard-migrate-frontend-ownership-roots.md)；用户已授权执行。
- [Agent 接口](../../Kodex/app/component/agent/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/agent/contract/AgentViewModel.kt)
  是真实页面身份，继承 settings commands，发布稳定 children 和 backend facts。
- 原 RpcAgentViewModel 包装 RpcAgentPresentation；现在由
  [唯一 RpcAgentViewModel](../../Kodex/app/impl/rpc/src/commonMain/kotlin/io/github/stream29/kodex/cli/rpc/RpcAgentViewModel.kt) 直接接管；
  Session 每次 presentation replacement 再建立 wrapper owner。
- **提议消融这个 wrapper/presentation 分裂**：合并到唯一 RpcAgentViewModel，
  保留完整 settings field-CAS、history confirmation、child creation、错误与 shell 投影。
  原 AgentViewModel FQCN 不变，旧 RpcAgentPresentation 类/重复创建路径删除，
  不将其改名 Adapter 后继续并存。
- RpcSessionBinding 是真正的缓存/订阅/协议对象，保持它；它不是另一个 Agent VM。
  RpcSessionView 负责 binding 存活并发布准确 AgentViewModel?，其创建/替换由协调者统一修改。
- 唯一实现工厂直接接收现有 binding/services/models/ownerScope/onCreated，
  返回原 AgentViewModel；它是 RPC implementation 的真实装配函数，不暴露到
  portable spec，不再接收 RpcAgentPresentation，也不留旧签名转发 overload。
- RpcSessionView 的实际 `agent: StateFlow<AgentViewModel?>` 替代 presentation，
  Session 的 rootAgent 借用这一发布源；删除其再造 wrapper 的 mutableAgent/collector。
  原 view 中 composer 的重复存储/独立关闭也退役，由唯一 Agent owner 关闭 child，
  真正消费者从捕获的 Agent 取得 Composer，不留 compatibility getter。
- presentation.failure → wrapper.notification 是内部中转，不是另一领域事实。
  合并后错误直接调用原 Agent report，删中转 flow/collector；Composer 自己的 typed
  summary 及真正 binding failure/recovery 仍保留，不新增错误传递服务。
- `app/viewmodel/agent` 当前只剩 AgentViewModelModule Koin marker；核对生成装配
  无真实生产消费者后删除旧项目，不新建纯 marker 的 agent impl/viewmodel。

## Target projects and actual contracts

- `app/component/agent/spec`：原 AgentViewModel、AgentSettingsViewModel、
  history confirmation、notification、lifecycle；依赖真实 child spec 与 RPC 值模型。
- `app/component/agent/impl/view`：原 AgentRuntimeView/Control/RenderLabels，
  AgentRuntimeScreen 及属于 Agent 的 RuntimeStatusBar 逻辑与测试。
- 实际唯一 RPC Agent VM 在 `app/impl/rpc`，与原 raw binding/child dependency
  装配 co-locate，避免 RPC factory→Agent impl→RPC factory 的人为循环。
  不要求为了一个 Gradle 项目增加另一套纯 Agent VM 或 dependency resolver。
- 原 AgentShellSession、AgentShellSessionRegistry 留在 Agent spec，
  同 FQCN，保留 sessionId/arguments/completed/close 与 raw registry。
  History 已直接消费这两接口，不能为了侧栏布局把它们重新定义成 ShellBrowser 契约。
- `app/component/session-sidebar/spec`：原 SidebarSettingsViewModel；
  sidebar 值模型保持现有 preferences spec 所有权，不复制 DTO。
- `app/component/session-sidebar/impl/view`：原 layout/header/splitter/content menu
  及 Shell 列表、摘要、menu/hover。Shell 是该 View 的内容分支，不新增独立 Shell
  spec/View 项目或 ShellBrowserViewModel 镜像进程状态。
  SidebarSettingsViewModelImpl 直接在 Application impl 接管，只有一个宽度来源。
- Sidebar 原接口有实际 viewport/写入命令，所以保留 spec；实现直接接管原
  class，不增加 SidebarFactory/Dependencies/provider 仅为跨项目拆分。

## Agent behavior and lifecycle gates

- root Agent 与某个 Session 当前 root 的关系只是现行组合，Agent 不命名为 Session VM。
  不新增多 Agent RPC、本地永久 AgentId、后端资源 lease 或 another registry。
- 每个 binding 有唯一 Agent child；失活先不可用，重新激活创建新 child。
  旧 Agent commands/menus 仍绑定原 owner，不能通过最新选中 tab lookup 去执行。
- 合并前列出两 owner 的 close/观察/child 停止时点，合并后逐项证明：
  frontend close 取消本地观察/等待，不 Stop 后端已接受 turn；
  SessionNotActive 只恢复保活和读取，不重放 append、fork、consume 或 create。
- settings 字段编辑保留捕获 baseline、完整 CAS、closed/binding 校验；
  Runtime Configuration 三字段原子更新不改成三次写入。
- Composer 普通 resume 错误进入 Agent 出口，不杀 binding Job；
  保留 Submit-to-steer 文案、一次 pendingSteer preview 与 revision/cursor。
- History 请求捕获 boundary/generation/requestId；确认消费准确请求，
  Index Check out 仍仅 scroll。多 timeline 历史仍归原 AgentHistoryViewModel。
- failure outlet 合并只去掉重复转换/收集层，保留 child error summary 与 Agent notification
  的各自含义；ordinary failure/cancel/close/stale 不能混为一个 Closed 状态。
- spec 写完整命令/状态/render/错误/borrowed dependency 约束；
  factory 返回原接口，不让 renderer 接受 RpcSessionBinding。

## Shell and sidebar behavior

- raw active registry 包含执行层保留的已完成 session；UI 展示正在进行列表的原筛选、
  sessionId 排序与 completed 变化。不得把 UI 隐藏当作执行层删除。
- Shell Close 捕获实际 Agent 与 process 目标；旧菜单不可关闭另一 binding 上的新项。
  不消费 stdout、不改 Process 的 destructive buffer、TTL 或本地 PTY 行为；
  摘要/arguments/hover/current-menu 的实际文本与原 UI 对照。
- 左右栏共用原 application-wide 内容与临时 widths，内容写入 frontend 文件，
  widths 不持久化；初始 viewport/指定 applicationWidth 沿原一次性规则。
- 同一个 Agent 的两栏可独立 focus/scroll/hover；History Index 的读 handle
  保留 per-consumer release。侧栏不复制 index loader/Agent settings 真源。
- Popup 排他仲裁、hover grace、拖动临时坐标和动画属于 Root/renderer；
  sidebar spec 不增加 geometry DTO 或新的 popup route authority。
- 关闭侧栏不关闭 Agent、Shell 实例或长期 child。组件 View 不反向依赖 Root View。

## Shared-file boundaries and tests

- 本工作线负责 Agent/Sidebar 两 spec、两 View 与测试。RpcAgent*、RpcSessionViews、
  ShellView、SidebarSettingsViewModelImpl、SessionTreeCliScreen 与旧共享文件由协调者改。
- ContextMenuTimestamp/SessionTreeUiPrimitives 逐函数按真实调用方分配；
  多 consumer 的纯 helper可进现有 components，但领域目标/业务确认不能藏在 visual helper。
- SessionSidebarSide 含箭头/resizeDirection，随 Shell/侧栏函数留在 Sidebar View；
  Root View 直接使用这一份 renderer 值，不建立 Shell View→Sidebar View 的反向边，
  不把侧栏专用类型塞入通用 components。
- Agent status bar 与 draft status bar分别随真实消费方迁移，后者进入既有
  New Session View。共用 layout/measurement/button helper保持单份 components
  实现；跨模块需要的 View入口改为准确 public API，不保留旧 Root forwarding 函数。
- 保留 AgentRuntimeControl、RuntimeStatusBar、HistoryComposerSeparator、
  HistoryEntryContextMenu、ComposerHostRepair、SessionSidebar 原测试。
- 真实 binding→Agent→renderer 测 ordinary failure、失活/恢复、close、
  pending child 借用与新旧实例隔离；不得只测试工厂可构造。
- 两栏布局、窄屏/展开/拖动、同帧左右宽度更新、hover/menu layering、
  完成 process 隐藏、stale Shell callback、History Index 定位均需真实动作断言。
- 保留原 DocsHistory/DocsWorkspace recording；本批不改内容展示或以新 snapshots
  全量覆盖掩盖 UI 差异。handoff 标出可见变化及平台未验证项，中央构建统一运行。

## Integration — Agent/Sidebar work line

- Status: scoped implementation and static self-audit complete; centralized validation pending. Baseline
  `034b778c637ee1749368765d307cc53dfe8acd58`. This section belongs to this work line.
- Four new builds: `app-component-agent-spec`, `app-component-agent-impl-view`,
  `app-component-session-sidebar-spec`, `app-component-session-sidebar-impl-view`.
  Conventions remain `kodex.kmp-cli` for specs and `kodex.kmp-view` for views.
- Contracts retain `io.github.stream29.kodex.app.agent.contract` and
  `io.github.stream29.kodex.app.application.contract.SidebarSettingsViewModel`.
  Copied both original Agent files and only the original Sidebar interface; preferences
  retains `SidebarContent`/`SidebarSettings`. Main deletes the original declarations.
- Root-callable renderer entrypoints retain `io.github.stream29.kodex.cli.app`:
  `AgentRuntimeScreen`; `SessionSidebar`, `SessionSidebarExpandButton`,
  `BoxScope.SessionSidebarContentMenu`, `BoxScope.ShellSessionContextMenu`,
  `BoxScope.ShellSessionHoverPopup`; `NewSessionStatusBar`, `newSessionStatusBarRows`.
- Sidebar-only public geometry/requests: `SessionSidebarSide`, `SessionSidebarColumns`,
  `ShellSessionMenuRequest`, `ShellSessionInteractionRequest`,
  `resolveSessionSidebarColumns`, `clampSessionSidebarResize`,
  `canExpandSessionSidebar`, `SessionSidebarCollapsedButtonColumns/Rows`.
  Keep them in Sidebar View, not components or spec. Row, summary, splitter and
  position-provider helpers stay internal/private. `SessionSidebarSide`'s arrow
  labels/resizeDirection are internal; root only needs the direction enum values.
- Pure helpers delivered in existing `app/view/components`, package
  `io.github.stream29.kodex.cli.components`: `StatusBarLayout.kt`
  (layout plan/width/button measurement), `StatusBarButtons.kt` (cwd/Settings),
  `SessionVisualStyles.kt` (only actually shared theme projections).
  Shared styles are exactly `SessionButtonForeground`, `SessionButtonBackground`
  (Agent/draft buttons) and `SettingsDialogHomeBackground` (Sidebar/root surfaces).
  Root imports only the last one. Sidebar-only foreground/header/navigation styles
  stay private in Sidebar View; `PopupMenuBackground` stays root-owned. Unused old
  displayName extensions are not revived; labels use the existing Runtime Configuration API.
- Runtime Configuration measurement belongs to existing
  `app/component/runtime-configuration/impl/view/.../RuntimeConfigurationWidths.kt`,
  package `io.github.stream29.kodex.cli.runtimeconfiguration`:
  `runtimeConfigurationButtonWidths`. Components has no Runtime Configuration dependency.
- Original cleanup owned by main: all `app/contract/agent/src`, original
  `SidebarSettingsViewModel.kt`, all `app/view/agent/src`, and old root
  `AgentRuntimeScreen.kt`, `RuntimeStatusBar.kt`, `SessionSidebar.kt`.
  From `SessionTreeUiPrimitives.kt`, separator/duration goes to Agent View;
  shared visual styles go to components. Single-consumer/unused remainder stays
  main-owned for actual-consumer placement or deletion.
- Original root tests are read-only here. Owned cases are copied/split into new
  views and helper projects; main removes old copies and retargets root recordings,
  root-menu tests and fixtures centrally. `HistoryEntryContextMenu` itself remains
  root-owned: preserve its actual item/generation/index routing.
- Shared RPC/Agent merge, Sidebar implementation, root calls, old Gradle and old
  source deletion are main-owned. No Shell project/VM/provider, Root View dependency,
  resolver, new registry/draft state, or new black-box statusbar slot is introduced.
- Existing helper build changes are only `app/view/components/build.gradle.kts`
  (`api(kotlinx.io.core)` for public Path-based visual helpers) and
  `app/component/new-session/impl/view/build.gradle.kts` (actual Runtime Configuration
  View API plus Mosaic/coroutines test dependencies). Runtime Configuration already
  depends on components; its build needs no edit. Four new builds use only the new
  coordinates and real child dependencies; RPC/Session implementations are test-only
  dependencies of Agent View. Sidebar's real-fixture test support is test-only.
- No Gradle, IDE, commit, branch, push, device or runtime-resource operation was run.
  No temporary files or retained resources were created by this work line.

### Test migration and remaining central ownership

| Original UI cases | New owner / defined case count |
| --- | --- |
| AgentRuntimeControlTest | Agent View, all 3 unchanged cases |
| HistoryComposerSeparatorTest | Agent View, all 5 unchanged cases |
| ComposerInputTest | Agent View, all 5 cases; borrowed existing ComposerInput |
| ComposerHostRepairTest | Agent View, all 6 real binding/backend/keyboard cases; presentation assertions now target the unique `view.agent` and notification |
| SessionSidebarTest | Sidebar View, all 21 cases, including original Index read fixtures, mouse drag, hover, menu, scroll and ANSI assertions |
| RuntimeStatusBarTest | Split: Agent View 1; components 2; New Session View 7. All 10 original case names retained; questions labels target the canonical Runtime Configuration label function |

- Six additional defined cases: Agent control callbacks/counter/wrapped row measurement;
  exact history item/index/generation forwarded by AgentRuntimeScreen; tiny draft
  statusbar draw/hit-test clipping; same Agent registry in two sidebars with completion
  filtering; equal-id Shell menu preserves its captured handle; completed Shell menu
  dismisses without Close. Total: 50 ported + 6 additional = 56 **defined, not executed**.
- Shell equal-id callback coverage above is renderer-level; production stale-binding/
  stale ShellView guard validation still belongs to main's RPC lifecycle tests.
- Main retains/retargets `HistoryEntryContextMenuTest` (all 3 action-routing cases),
  `ContextMenuTimestampTest`, `RevertAndEditTest`, `SessionViewportTest`,
  `NewSessionScreenTest`, shared Session fixtures and DocsHistory/DocsWorkspace
  recordings centrally. They are not copied behind a Root View dependency or replaced
  by object-toString snapshots. Main removes the six copied old test files after
  retargeting; mixed/root recording files must remain.

### Static evidence and integration gate

- Read-only baseline comparison passed: both Agent files and Sidebar interface have
  identical FQCN/declarations/signatures after excluding KDoc/imports. All three original
  Agent View files are byte-identical copies. AgentRuntimeScreen and the entire
  SessionSidebar executable content are unchanged apart from required public visibility
  and private/shared style relocation. Public request properties are explicitly visible
  for the existing convention's explicit API mode.
- Original statusbar layout/plan/cwd formatting/control dispatch are preserved.
  Only the configuration projection is inlined into each domain's row planner; the
  actual label/button-width function is a single Runtime Configuration View helper.
  No components→Runtime Configuration/Agent/Root View dependency is added.
- Static scoped-build checks passed: two cli specs, two view builds, no old coordinate
  references, no Agent/Sidebar View→Root View edge, and no spec→implementation edge.
  All 50 original owned case names are present. Owned helper-build/task diff whitespace
  checks passed; an unrelated pre-existing outer task whitespace finding was not edited.
- No visible UI behavior change is intended. Kotlin compilation, JVM execution,
  platform builds, actual recording comparison and whole-batch lifecycle acceptance
  were pending at this lane checkpoint. Static source comparison is not that validation;
  final coordinator evidence is below.
- Integration entrypoints:
  [AgentRuntimeScreen](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/agent/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/AgentRuntimeScreen.kt#L45),
  [complete Sidebar View](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-sidebar/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/SessionSidebar.kt#L75),
  [draft statusbar](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/new-session/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/NewSessionStatusBar.kt#L23).
- Helper locations:
  [pure layout/width plan](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/view/components/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/components/StatusBarLayout.kt#L14),
  [cwd/Settings visuals](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/view/components/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/components/StatusBarButtons.kt#L11),
  [actual configuration widths](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/runtime-configuration/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/runtimeconfiguration/RuntimeConfigurationWidths.kt#L7).
- No unresolved authored ownership conflict. Old-source deletion/root/RPC work was
  observed advancing concurrently and was not edited by this line. This handoff
  deliberately leaves central validation/acceptance pending rather than marking the
  parent task done.

## Coordinator final evidence

- 原 Agent wrapper/Presentation/failure 中转、Session 第二次包装与原 source/build
  入口已退役。唯一 RpcAgentViewModel 拥有原 children，raw view 发布 Agent flow；
  Root、History、左右侧栏及 pending renderer 直接消费原接口。
- 原 AgentRuntimeScreen 可执行代码不变；Sidebar 保持原完整呈现，
  只公开实际跨项目入口并移动 genuine shared style。Settings 字段接口的 KDoc
  区分持久 Agent CAS 与本地 New Session draft，不强称二者都执行远端 CAS。
- Agent View 22、Sidebar View 24、components 142、New Session View 8、
  Runtime Configuration View 11 项均通过；真实绑定恢复、旧 Agent/Composer 关闭、
  普通 resume 失败与 Root 录制测试通过，明细归
  [主矩阵](2026-10-06-hard-migrate-frontend-ownership-roots.md#final-validation-and-remaining-limits)。
- 新鼠标菜单测试使用 Mosaic 帧派发真实事件；动作不改变画面时不要求新 snapshot。
  Shell 菜单 fixture 以稳定 request 和真实宿主 dismissal 验证旧目标捕获/完成清理，
  没有以手工赋值代替 renderer 回调。窄屏越界 click 正确不改变画面亦有直接断言。
- Agent/Sidebar spec/View 的 Linux x64 主源码编译通过；追加真实无账号 CLI
  [局部验收](2026-10-06-hard-migrate-frontend-ownership-roots.md#actual-cli-behavior-acceptance)
  验证 Agent/History/侧栏菜单及关闭。双侧栏窄屏的基线限制已复现并记录；
  有输出的 Shell 与运行中控制未在该 CLI 实操，其余平台仍未验。
  用户已随整批验收，按 [验收范围](2026-10-06-hard-migrate-frontend-ownership-roots.md#scoped-acceptance) 归档；
  原 AgentState 风险不在此修复。

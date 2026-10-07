# Task Tree

- `Trace the six frontend responsibility units and real owners`()
- `Specify project ownership, ablations, and complete consumer cutover`()
- `Separate preserved behavior from proposed identity and lifecycle changes`()
- `Define three work lines and one shared-file coordinator`()
- `Receive review of the frontend ownership engineering plan`()
- `Receive explicit implementation authorization`()
- `Move the original contracts and cut over the actual implementations`()
- `Integrate the root renderer, CLI assembly, and all consumers`()
- `Compare original UI and validate real ownership paths`()
- `Run the actual isolated CLI and verify navigation and ownership behavior`()
- `Receive scoped acceptance and archive`()

# Details

## Authorization and baseline

- 用户完成方案及消融自查评审，明确“可以直接去执行”。四份任务进入 executable，
  授权本批硬迁移、集中构建及原 UI 对照；不包含提交或推送。
- 源码基线：`Kodex/` 的 `refactor/spec`，`034b778c637ee1749368765d307cc53dfe8acd58`，
  内层干净。外层其他任务、shared-context 改动与用户 Draft 不属于本批。
- 按真实所有权组织，不把六块职责机械变成六套新 ViewModel、十八个项目。
  原 `AgentViewModel`、`SessionViewModel`、`ApplicationViewModel`、
  `SettingsViewModel` 的领域身份保留；不改称 Workspace/Timeline/Aggregate。
- 约束：[组件与 frontend 边界](../../checklist/frontend-application-boundary.md)、
  [硬迁移规则](../../checklist/spec-impl-module-boundaries.md)、
  [RPC 生命周期](../../checklist/rpc-architecture.md)、
  [Session 视图与实体](../../checklist/rpc-session.md)、
  [资源清理](../../checklist/coroutine-resource-lifecycle.md)。

## Six responsibility units

| 范围 | 原事实来源 / 本批闭包 |
| --- | --- |
| Agent 页面 | 原 Agent 契约、RpcAgentViewModel/RpcAgentPresentation、运行控制与状态栏；直接组合 History、Composer、Runtime Configuration 和两个 pending child。 |
| Shell 会话面板 | 原 AgentShellSession/Registry、RPC ShellView、活动列表、摘要、菜单与 hover；不重新实现 Process/Shell 执行。 |
| 左右侧栏 | 原 SidebarSettingsViewModel、一次 viewport 初始化、内容持久化、临时宽度、splitter、Shell/History Index 布局与 popup 仲裁。 |
| Settings 根 | 原 SettingsViewModel/GlobalSettingsViewModel、切页与子组件清理、全局失败出口、Login 往返保留；不重复迁移已独立的编辑组件。 |
| 前端 Session 根 | 原 Session/PersistedSession 接口和 registry、加载/恢复/释放、准确 Agent 绑定及草稿消费者。 |
| Application 根 | 原 navigation/popup、tab 管理、物化入口、准确目标校验、宿主装配与前端 shutdown。 |

- Shell 是现有可观察 process 模型的呈现组件，不新增 ShellBrowserManager/
  SessionResolver。左右侧栏已有真实 ViewModel，不再造一份 SidebarState authority。
- Application/Session 根不是后端 repository；Session 当前只展示 rootAgent，
  不在本批提前增加多 Agent DTO/注册协议，亦不把 Agent 定义成 Session 的永久同义词。
- 已迁移的子组件只做必要消费方切换，现有状态和业务命令不搬回父级。

## Project mapping

| 原项目或真实声明 | 目标 |
| --- | --- |
| `app/contract/agent`，含原 Shell 两接口 | `app/component/agent/spec`；原 FQCN 保留 |
| `app/contract/settings` | `app/component/settings/spec` |
| `app/contract/session` | `app/spec/session` |
| `app/contract/application`，除 SidebarSettingsViewModel | `app/spec/application` |
| SidebarSettingsViewModel | `app/component/session-sidebar/spec`；原 FQCN 保留 |
| `app/viewmodel/rpc` | `app/impl/rpc`；实际 frontend RPC 绑定与组件装配 |
| `app/viewmodel/session` | `app/impl/session`；原 registry 与 PersistedSession 实现 |
| `app/viewmodel/application` | `app/impl/application`；Application、Sidebar 实现与真实宿主装配 |
| `app/viewmodel/settings` 的 SettingsViewModel | `app/component/settings/impl/viewmodel` |
| SettingsUpdateQueue / McpSettingsProjection | `app/impl/rpc`；queue 消费者是 RpcGlobalEditor 与 RpcNewSessionSettings，projection 消费者是 RpcMcpSettings |
| `app/view/agent` + AgentRuntimeScreen / Agent 状态栏部分 | `app/component/agent/impl/view` |
| RuntimeStatusBar 的 NewSessionStatusBar / draft rows | 原 `app/component/new-session/impl/view`；只接管现有 renderer |
| `app/view/settings` | `app/component/settings/impl/view` |
| SessionSidebar.kt 的布局/header/splitter/content menu 及 Shell 列表/摘要/菜单/hover | `app/component/session-sidebar/impl/view`；保留同一个真实呈现组合 |
| `app/view/application` 的真实根 screen、导航/菜单仲裁、title | `app/impl/view` |
| `app/viewmodel/agent`、无源码的 `app/view/session` | 删除；不另造 spec 或空项目 |

- 13 个旧项目入口退出，13 个有实际职责的目标项目接管。
  五个 spec、八个实现项目；其余已独立组件、UI 基础设施和 frozen migration 不改归属。
- Shell 仍是六块功能范围之一，但不对应新项目或新 ViewModel。其模型由 Agent
  拥有，History 与侧栏共同消费；列表/菜单/hover 是 Sidebar 的内容分支，
  没有独立的页面 owner。取消原提案中的 Shell spec/View 两个项目。
- `app/impl/rpc` 是多个真实 frontend spec 的实现/绑定项目，不建空的
  `app/spec/rpc` 或第二套 RPC ViewModel。Agent 与 Global Settings 的真实 RPC
  实现在这里接管；无需增加跨项目 provider 层才算“每个组件有实现”。
- Sidebar 实现暂与原 Application 装配一起进入 `app/impl/application`：
  它真实依赖应用级内容/宽度来源。其 spec/View 仍独立，不为分出一个实现项目
  添加转发 adapter。Planning 审查可以否决这一映射，不以旧目录为不可变前提。
- 不把真实 host composition 的客户端/Home/backend 构造 seam 暴露成 frontend
  service locator；`withKodexApplication` 是装配函数，不另造 ApplicationService。
- spec 用原 `kodex.kmp-cli`；ViewModel/绑定 topic 用 `kodex.kmp-viewmodel`，
  View 用 `kodex.kmp-view`，不修改 convention 或 SDK 项目来凑 target。

## Spec and dependency gates

- 原类型、sealed 状态、序列化身份和普通签名随声明移动；例外只有下述需明确
  评审的 index→exact-target API 变更。禁止旧坐标 forwarding、typealias 或双路工厂。
- spec 列明实际输入端口、稳定 child、状态分支、渲染要求、命令准入和
  `@throws`；不暴露 Mosaic、RpcSessionBinding、完整可写 store 或后端实例。
  已有 typed factory/参数可承担依赖声明，不要求每种实现重复造 Dependencies façade。
- 目标主图：Application spec → Settings/Session/Agent/Sidebar spec；
  Settings spec → Session spec → Agent spec；
  Sidebar spec → 原 preferences 值模型。其他 child spec 保持实际依赖方向。
- 实现主图：Application impl → Session impl、RPC impl、Settings VM；
  Session impl → RPC impl；RPC impl → 真实 child impl/spec。
  把 queue/MCP projection 移入 RPC impl 后，删除原 RPC→Settings VM helper 边。
- New Session impl 仍可借用真实 Session/RPC 实现；后者不得反向依赖 New Session
  impl。History renderer 只依赖 Agent spec，不依赖 Agent renderer。
- Root View → Agent/Sidebar/Settings 与已有 child View；这些 View
  不反向依赖 Root View。共享纯 visual helper 只有多处真实消费才放 components，
  不以“方便编译”复制一套领域逻辑。
- 原 internal renderer 入口跨项目后，协调者只公开真实 root 调用的函数/请求，
  不把所有 helper 公开，亦不把 Mosaic geometry 移入 spec。
  SessionSidebarSide 是含箭头/resizeDirection 的 renderer 值，随 SessionSidebar
  留在 Sidebar View，Root View 直接使用；不为取消的 Shell 项目把它搬进 components。
- RuntimeStatusBar 按实际调用方分割：Agent 按钮/计数在 Agent View，
  draft 状态栏在原 New Session View；共享 StatusBarLayout/宽度计划和
  工作目录/Settings 纯 visual button 在 components。对应测试按归属分割，
  不为避免拆文件传入黑箱状态栏业务 slot。
- 通用 layout只收已测量宽度/visual content，不依赖 RuntimeConfiguration 值或
  Agent controls；配置 label/button-width 投影属于既有 Runtime Configuration View，
  两个消费者共用它，禁止 components→领域 View 的反向边。

## Proposed ablations and behavior review

- **Agent 两层合并**：当前 RpcAgentViewModel 再包装 RpcAgentPresentation，
  后者持有 children、失败与 history confirmation。提议将现有行为直接归入唯一
  RpcAgentViewModel；RpcSessionView 发布准确 `AgentViewModel?`，Session 不再为同一
  binding 创建第二个包装 owner。保留 raw binding/cache 与 frontend VM 的真实分工。
- 合并时不把 raw failure、Agent notification 和 Composer typed summary 合成
  无区别错误模型；保持一个宿主可见出口和各 child 的规范语义。
  原 presentation.failure 仅供 wrapper 收集为 notification，合并后内部失败直接
  进入原 Agent report，删除该中转 flow/collector；Composer typed summary 保留在 child。
  不再为内部错误传递新增 FailureAdapter、ErrorService 或另一套错误状态。
  若发现两 owner 有不可消融的行为，先列出反例与测试，再修改计划，禁止留平行 VM。
- **准确 tab 命令**：现有 select/materialize 先从 UI target 转整数 index，
  异步取锁后可能解释成另一个 tab。既有
  [New Session 竞态调查](2026-10-03-investigate-new-session-tab-crash.md)
  是输入，不把其过期行号或旧未提交状态当作今天事实。
  提议 `selectTab(target: SessionViewModel): Boolean` 与
  `materializeNewSession(target: NewSessionViewModel): PersistedSessionViewModel?`；
  在原 commandMutex 中查准确实例，过期/已物化 target 返回 false/null，无第二次创建。
  真正创建、存储与传输错误仍传播；不靠 catch-all 把失败变成过期。
- 这是明确的 frontend 行为修正提案，不是无语义目录移动；需随本计划获得评审。
  不保留 index overload，也不新增永久物化结果表或通用去重服务。
- **Registry release/open**：原 registry 与 RpcSessionViews 是 tab handle 复用和
  RPC 保活/恢复两个真实职责，不直接凭“双 map”判断应合并。
  [原 Agent/navigation 重验](../done/2026-10-05-audit-agent-navigation-components.md)
  提出 release 后重新 open 可能误关闭新 view，仍须 gated 复现。
  若确认，先评审现有 exact-view release 或原 Mutex 范围的最小修正；
  不新增远端 lease、永久 Session identity 或第三个本地 registry。
- 旧 State compaction/CAS 覆盖缺陷不属于本批；沿用
  [已验收 State 风险记录](../done/2026-10-06-hard-migrate-agent-state.md#existing-compaction-risk)。
  Home 格式、RPC wire、TTL 60/20、控制 Hook 和 backend naming 不改变。

## Work lines and file ownership

- [Settings root](2026-10-06-cutover-settings-popup-root.md)：Settings spec/VM/View
  与其测试；只提出 queue/projection 移动，公共 RPC 路径由协调者执行。
- [Agent, Shell, Sidebars](2026-10-06-cutover-agent-shell-and-sidebars.md)：
  Agent/Sidebar 两块 spec/View、Shell 呈现与专属测试；Agent 实现合并方案交给共享 RPC owner。
- [Application and Session roots](2026-10-06-cutover-application-and-session-roots.md)：
  两根 spec/impl、准确 tab 修正和根生命周期测试。
- 协调者独占全部 `app/impl/rpc`、原 RpcSessionViews/RpcAgentPresentation/
  RpcAgentViewModel、SessionTreeCliScreen、Application.kt、所有宿主 Gradle、
  旧入口删除、CLI/Docs fixtures 和集中构建。
- 三线可并行写不重叠的新目录；从原共享文件复制的声明由协调者一次删除。
  Agent 线不自行改 RpcSessionViews，Settings 线不同时改 RpcGlobalEditor，
  root 线不与协调者重复改宿主。未获用户确认不创建执行 Session。
- 实施顺序：固定 spec/API → 唯一实现消融与移位 → 子 View → Root/CLI
  整合 → 删除所有旧入口 → 原 UI 对照与集中回归；中间态不是可交付 staged adapter。

## Acceptance matrix

- 静态：唯一原接口/实现/工厂、全部十三个旧坐标退役、真实调用链，
  main/test 依赖分别检查；所有目标入口存在，spec→impl、View→Root View 环为零。
- 保留原 Settings、RPC、Session、Application 与 renderer 测试；
  从旧模块移动有效 fixture，不用删除真实 host 测试换取新项目绿色。
- 新测试：两次同 draft 提交、旧 target 等锁时 tab 移动/关闭、同名不同对象、
  Agent binding 失活/恢复、旧 Shell/menu 回调、双侧栏同时消费同 Agent。
  Registry release/open 单列复现，不用随机 sleep 推导正确性。
- Settings→Login→return、离页清未接受 draft、关闭后排空已接受 queue、
  失败只显示一次、真正 owner close 与 renderer 卸载分别测试。
- 实际 root tests 驱动输入和 callback，不以直接调用 VM 或手动赋值替代
  UI/Session/Binding 整条链路；原 snapshot/recording 必须保留正常及窄屏输出。
- 编译/JVM：新 RPC/Session/Application 实现、Settings VM/View、Agent View、
  Sidebar View（含 Shell 呈现）、Root View 与真实受影响 child；
  原 CLI JVM、Integration 测试源码编译。
  NO-SOURCE、不执行的 live tests、UP-TO-DATE 结果分别记录，不虚报新增测试数。
- UI 对照覆盖消息/工具历史、Composer、运行/压缩、pending、Settings/Login、
  两侧栏、tab 菜单、New Session、失败及 terminal title。
- 先核对用户资源、JDK 与已有 daemon，统一 Gradle 单线低 worker。
  按可用设备补 Linux x64 编译；macOS/ARM、真实 CLI/live provider 未验则明确列出。
- 任何新关闭/取消传播、错误丢失或 RPC wire 漂移必须停止整合。
  保活/远端任务不因 frontend child close 被取消，Home 在所有使用者之后释放。
- 执行结果见下节；上述矩阵是验收门槛，不以 Planning 阶段的静态检查替代运行证据。

## Planning evidence (historical)

- 已逐项核对六块接口、生产 factory/caller、原 renderer、真实 RPC 回归、
  scoped host 和旧竞态记录，内层基线保持干净。
- 自查后静态主依赖投影为 208→208 个业务/测试项目：13 个旧入口退出，
  13 个真实 topic 接管；按本文 helper 移位和 View 分割未发现缺失主边、主图环
  或五个新 spec 的直接 impl 边。main/test 依赖分别解析；
  它不是 Kotlin/Gradle 解析或平台验证结果。
- 四份新 Planning 已关联总任务；三个实施线的共享文件与中央构建所有权明确。
  自查后的本地链接/锚点、任务阶段和内外层 diff 检查记录在下节，现交付评审。
- 用户随后批准消融后的映射、Agent 合并与 exact-target API 并授权实施。
  Planning 静态检查不是生命周期运行结果；registry 实验及当前验证另列执行证据。

## Planning ablation review (historical)

- 本轮“消融”是对真实调用链的设计自查，未修改 Kotlin、运行实验或启动实施。
  项目数量不是验收指标；每条保留边界必须有下面的实际职责。

| 候选层 | 去掉这一层后的结果 / 方案 |
| --- | --- |
| 独立 Shell spec/View | 去掉更简单。AgentState.kt 的两接口服务于 Agent、History 和侧栏，不是新的 Shell 页面模型；SessionSidebar.kt 的 Shell 分支共享侧栏布局和 popup 几何，留在同一 Sidebar View。 |
| RpcAgentPresentation → RpcAgentViewModel | 去掉包装与内部 failure 中转，保留唯一 RpcAgentViewModel。raw RpcSessionBinding 仍负责协议/缓存；它不是另一个 VM。 |
| Session.mutableAgent/包装 collector | 从唯一 RpcSessionView.agent 借用真实 child，取消第二次创建与发布。binding 每次更换仍创建新 Agent owner，而非复用旧 owner。 |
| Sidebar spec | 保留。原 SidebarSettingsViewModel 有 viewport 准入、内容写入和宽度命令，Application 与 renderer 使用同一实例；不新增 Sidebar DTO/Factory/Dependencies 包装。 |
| Settings 根 VM | 保留。它承担切页、hidePage、refresh 和 child disposal，不只是转发子组件；已存在的 child 参数就是输入。 |
| Session registry / RpcSessionViews | 两个真实职责：tab handle 与 RPC 保活/恢复。去掉任一个会改变复用或生命周期；先复现 release/open 风险，不新增第三层替代。 |
| 共享 status-bar layout | 保留原纯布局函数，Agent 与 draft 两个真实消费者；按文件归属移动，不升级为 LayoutProvider/策略对象。领域控制仍各自持有。 |
| exact-target 命令 | 保留直接接口替换以解决已调查的 index 竞态；不加 CommandTarget、receipt 或去重服务。它是原命令的准入修正，不是新抽象层。 |

- 现行接管路径：[Agent Shell 接口](../../Kodex/app/component/agent/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/agent/contract/AgentState.kt)、
  [History 的实际 Shell 消费](../../Kodex/app/component/history/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/history/AgentHistoryView.kt)、
  [Sidebar 呈现与方向值](../../Kodex/app/component/session-sidebar/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/SessionSidebar.kt)、
  [唯一 Agent 实现](../../Kodex/app/impl/rpc/src/commonMain/kotlin/io/github/stream29/kodex/cli/rpc/RpcAgentViewModel.kt)、
  [直接借用 child 的 Session](../../Kodex/app/impl/session/src/commonMain/kotlin/io/github/stream29/kodex/cli/session/SessionViewModels.kt)。
- Agent 合并的实施门槛：在原 bindingScope 下创建唯一 owner，binding 失活时停止
  child 并撤回发布；Session.close 通过现有 view.close 终止该 owner。命令失败仍 caught，
  cleanup 不 Stop backend；closed、恢复和迟到回调的真实测试通过才可验收。
  不借自查宣布这项生命周期改变已经验证。
- 自查后四份 Planning 的 31 个本地链接/锚点、单一待评审步骤及内外层
  diff 检查通过；内层 HEAD 与基线一致且干净。仅修订这四份计划及总任务入口。

## Execution checkpoint

- 用户授权后四份任务已进入 executable，三条实施 Session 为 511（Settings）、
  512（Agent/Sidebar）、513（Application/Session）。本会话拥有所有公共接线和构建。
- 原 RPC 源码/测试/构建文件移动到 `app/impl/rpc`；原 scoped
  Application.kt 移入 `app/impl/application`。queue/projection 同 FQCN
  移到实际 RPC 项目的 `app/settings` 包路径，两位 queue 消费者为
  RpcGlobalEditor 与 RpcNewSessionSettings。
- 已删除 RpcAgentPresentation，唯一工厂为
  `createRpcAgentViewModel(binding, scope, services, models, onCreated): AgentViewModel`。
  RpcSessionView 发布 `agent: StateFlow<AgentViewModel?>`，不再存储另一份 Composer；
  初始化/恢复/关闭负责该 child 发布及关闭，草稿 final resume 使用捕获的 Agent。
- 原 RPC/真实 Composer host 测试已改用该 Agent；stale history confirmation
  经实际 Agent caught command 验证，不保留已删内部 Presentation 的测试入口。
  追加恢复后旧 Agent/Composer 关闭与新 binding identity 断言。
- 原 root renderer 的 select/materialize 回调直接传捕获的 tab，而不先变成 index。
  十三个旧项目的 source/build/test 入口已全部退出；原有效测试直接迁入真实所有者。
  Agent/Session/Application/Settings 原 FQCN 保留，唯一 factory/caller 已接入。
- Registry 风险在原算法上通过 gated 真 RPC 用例复现：新 opener 被旧 release 取消。
  用原 Mutex 覆盖准确旧 view 的释放后，同一用例通过；没有第三个 registry/lease。
- 主矩阵：十四个 JVM test 目标共 507 项（包含 UP-TO-DATE 结果），零失败/跳过；
  Integration 测试源码编译通过。保留原 Docs 三套录制用例及 fixture，
  原断言未改，非 Kotlin 资源字节保持不变。
- 新菜单/边界测试首轮暴露了 fixture 的帧驱动与 owner dismissal 问题：
  Mosaic 在帧内排空输入，单纯 delay 不派发鼠标；不改变 UI 的操作也不承诺新画面。
  修正真实输入的 frame pumping、准确稳定菜单 request 与宿主 dismissal 后测试通过，
  没有改成直接调用 VM 或手工注入动作结果。
- Linux x64 补验首轮发现既有 Composer 值类型缺少 `kotlin.jvm.JvmInline` 导入；
  已最小补上导入，十三个本批新项目的 Linux x64 主源码编译通过。

## Final validation and remaining limits

- 用户追加真实运行验收；结果见下方 [实际 CLI 验收](#actual-cli-behavior-acceptance)，
  与 Mosaic/编译矩阵分开记录，未执行的链路不计为通过。

- 最终统一命令成功：十六个 JVM test 目标共 **650 项**，零失败/跳过，
  包含复用的 UP-TO-DATE 结果，不代表 650 个新用例。明细：
  Application 43、Session 13、RPC 73、Root View 52、Settings VM 12/View 18、
  Agent View 22、Sidebar View 24、components 142、New Session View 8/VM 5、
  Runtime Configuration View 11、History View 49/VM 37、RPC Server 118、
  Session in-memory 23。CLI JVM 与 Integration 测试源码编译通过；
  Integration 测试未执行；该矩阵当时只包含 Native 主源码编译，
  后续 Linux x64 CLI 链接/运行见下节。
- 原 Agent screen 与 SettingsPopup 可执行呈现内容保留，Sidebar 仅作真实入口
  可见性/样式归属调整；Root renderer 仅切换 exact-target 命令和 helper import。
  原窄屏/菜单/错误、Settings→Login→return、History/Composer 与三套 Docs
  Mosaic 输入/录制测试保留并通过。非人工操控真实终端的视觉验收。
- 最终复跑发现原 host cancellation 用例偶发 `RpcClient was cancelled`。
  相关 Application 装配与 RpcGlobalSettings 源码此前原样移动；
  追加受控 subscription 用例，确认已取消 owner 的迟到普通错误会取代取消。
  在原 `required()` 观察终结边界先检查当前协程活性，关闭时保留取消；
  活跃订阅的普通失败仍传播，未加 catch-all、重试或另一层错误模型。
  原算法的受控关闭用例失败，修正后关闭/活跃失败两用例均通过；
  原 host 取消验收加强为八次独立 Home 创建/取消/释放循环并通过。
- 本轮沿用 JVM 26 的同一个自建 Daemon，单 worker、不并行构建；未操作 IDE。
  Native 补验关闭 configuration cache，避免 Mosaic bitcode 任务的已知
  配置缓存编码问题；未修改 convention、JDK 要求或构建插件。
- 静态图为 **208 个实际 build 入口**（含结构根共 220 个 Gradle 路径，不含仓库根项目），
  main/test 边分别扫描；十三个旧入口退出、十三个真实项目接管，
  目标无缺失、主图无环、五个新 spec 无直接 implementation 边。
  原生产声明/旧坐标退役，唯一 Agent 工厂、Session 借用源与准确 tab 调用链已核对。
- macOS/ARM/Windows、live provider、Native 测试未验证。
  CLI convention 不声明 JS target，本批不虚报 JS 编译。
  已知 AgentState compaction/CAS 覆盖缺陷仍未修复，沿原风险记录独立跟踪。
- 用户范围验收见下节；未覆盖平台、运行链路和既有风险不因归档而视为已解决。
- 收尾：四份当前任务与 SOP 的 40 个链接/锚点、唯一生产声明、
  125 个新 Kotlin/build 文件文本及内外层 diff 检查通过。
  十三个旧生成目录与本轮临时日志已删除，确认无客户端构建/IDE 后仅结束
  本轮自建且空闲的 Daemon；用户其他文件、任务和 shared-context 改动未处理。

## Actual CLI behavior acceptance

- 使用当前未提交源码执行 `:app-cli:linkDebugExecutableLinuxX64`，JVM 26、
  单 worker、关闭配置缓存，成功链接 debug CLI（1 分 41 秒）。
  直接运行这个二进制，不使用旧 release 或测试版 renderer。
- 两次运行均为独立 PTY（120×40，补测 80×24、60×20），发送真实 SGR
  鼠标/键盘输入并解码实际 ANSI 输出；使用临时 HOME/工作目录，清除认证环境变量，
  `.kodex`、`.codex`、`.agents` 均指向隔离 Home。未操作用户终端、账号或浏览器。
- 实际通过：
  - Settings 七页切换；Current session 重命名更新 tab；工作目录浏览、
    选择当前目录后返回原页面；General 修改换行键后 submit key 联动。
  - 新建两个草稿，输入内容分别保留；切换回来后原输入仍在；
    tab 右键关闭只移除目标草稿。
  - 无账号提交消息成功物化 Session，History 显示原用户消息；
    resume 认证失败由 Agent 呈现，binding/进程继续可操作，未变成 Root 退出。
  - 左右侧栏与内容菜单、History Index 行及真实 Check out 菜单操作；
    无 Shell 时列表为空；打开/退出空目录，物化后目录显示准确实体。
  - 关闭持久 tab 后从目录再次打开，消息仍在；重启同一 Home 后还可再打开。
  - Settings→Login Ready→Cancel 回到 OpenAI 原页；未点击 Open browser，
    不将 Ready/Cancel 验证当成真实 OAuth 或 MCP 浏览器 effect 验证。
  - 两次 Ctrl+C 正常退出，第二次在 Settings 打开时退出，退出码均为 0；
    Home lease 文件清除，同一 Home 重启无占用错误。
    持久 new-line key 保留为 Enter，临时侧栏宽度从 28 重置为 viewport 默认 29。
- 保存了 38 个临时输出 checkpoint；对其中 12 个关键实际输出、
  两个退出码及 Home lease absence 作附加断言。不计入前述 650 个 JVM 用例。
- **既有窄屏问题已实际复现**：从 120 列缩到 60 列，两个约 29 列的侧栏
  同时展开时，中心只剩约 2 列，文本与状态按钮几乎不可读。
  原 `resolveSessionSidebarColumns` 和 `SessionSidebarMinimumContentColumns = 1`
  算法本轮未变；收起侧栏后 60 列主内容/Settings 可操作。
  这是基线体验限制，不能将“双侧栏窄屏 UI”宣布完全通过，也不在目录迁移中
  偷改最小内容宽度/自动收起策略。后续如修正需单独确定布局策略与回归门槛。
- 范围限制：未用真实账号生成回复；有输出的 Shell、运行中 steer/Stop、
  pending tools、恢复重连仍只有先前真实 fixture/Mosaic 回归证据，
  没有在这两次无账号 CLI 中实操；未验证物理终端的颜色/字体差异。
- 本轮只补运行验收与任务记录，未改生产源码，未提交或推送。
- 临时 Home、PTY 控制进程/脚本、38 个输出快照与构建日志已清理；
  确认本轮自建 Daemon 仅执行该链接任务且空闲后，按准确 PID 结束，
  原用户 Kodex/IDE 进程保留。运行检查完成后交付用户整批验收。

## Scoped acceptance

- 用户自行运行验收后确认：“我自己也验收了一遍，觉得可以提交了”。
  主任务及三份实施任务归档到 done，
  [独立 root 子线 handoff](2026-10-06-application-session-roots-child-handoff.md)
  一并保存为历史证据。
- 授权本地分批提交代码、验收记录和子模块指针，不包含推送。
  代码按可构建硬迁移与独立新增回归分批；其他用户任务、文件与 shared-context 不纳入。
- 本次验收不改变上述运行限制；窄屏双侧栏的原有可用性限制及
  AgentState compaction/CAS 覆盖并发设置的既有缺陷继续保留。
- 内层两批本地提交：`9272f26f` 接管真实前端所有者、消费方和原测试，
  包含已复现的最小生命周期修正；`8297d388` 加入独立的 exact-target、
  Session 绑定所有权及完整 Settings popup 输入回归。最终工作树与已验证代码一致；
  提交阶段未重跑构建，也未推送。

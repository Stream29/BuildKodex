# Task Tree

- `Confirm the boundary-convergence checkpoint before research`()
- `Map coroutine owners and external resources across the application`()
- `Compare current shutdown cancellation and failure propagation contracts`()
- `Verify the reproduced local cancellation against the current owner graph`()
- **`Discuss failure boundaries and minimal owner-local governance`()**

# Details

- 后续主线已按用户授权完成原问题的最小治理，以及独立前端renew和具体原因
  呈现并合入产品 main `082bfc66`；见[父卡交付记录](../done/2026-10-13-design-backend-daemon.md)及
  [lifecycle-04交付](file:///home/stream/ACodeSpace/local/kodex-backend-daemon-evidence-787/final/lifecycle-01/README.md)。
  下方847调研保留其当时源码和候选边界，不再代表原问题尚未修复；
  完整Session/MCP树仍未整体确认，不能将本卡自动归档为全局治理完成。

- 用户要求先完成本轮 Daemon 边界问题收敛，再共同调研全局协程生命周期。
  前置项是[已完成的 Daemon 实施](../done/2026-10-13-design-backend-daemon.md)。
  本卡只追踪后续调研，不授权提前建立全局 Job/Scope 管理框架。
- 延续既有 [协程资源生命周期](../../checklist/coroutine-resource-lifecycle.md)
  与 Session、RPC、组件 exact-owner 规则；不因 Daemon 常驻而重新改写
  业务归属、任务保活或错误类型。
- 调研入口：后端进程/服务图、连接、Session/runningTurn、MCP/OAuth、
  标题/模型辅助请求、Hook、文件/进程句柄及 TUI owner。
- 逐项记录 Job 父子关系、实际 owner、接受/等待的取消边界、失败传播、
  close/shutdown 可等待性及常驻资源预算；区分已有契约、实现偏差和未决项。
- 特别核对 RPC waiter 与已接受后台工作分离、临时资源取消交接、
  NonCancellable 清理范围、过期/重连/关闭竞态以及测试 fixture 与生产路径差异。
- 首轮只读绘制现状及问题清单，必要验证仍用隔离数据；重量级 Gradle
  继续在 remote MacBook。真实 Home、付费模型、全仓改造及提交发布不在授权内。
- 当前接管状态、证据口径与未决项见下节；后续首轮分析保留其历史边界，
  不将早期回归统计当作当前组合快照全通过证明或全局改造许可。

## Session847 接管核验 — 2026-10-15

### 范围与证据

- 本 Session 负责问题2及相关协程生命周期治理讨论；Session787 继续澄清
  问题3（恢复期间心跳）和4（根异常因果链/呈现）。本轮不改产品源码、
  公共 checklist、父卡或预览；不提前实施全局 Job/Scope 注册框架。
- 用户已接受问题1的环境代理行为，不要求禁用代理；旧报告的直连建议
  不再代表当前决定。远程部署仍非本期范围。
- 实际审查对象为隔离 worktree `kodex-backend-daemon-787`，
  `feat/backend-daemon`、基线 `3223a52d9ab0561e35446fa96c14342a87f363c0`。
  本轮逐文件校验 `source-manifest-simplified-02.sha256` 的 1,311 文件，
  全部匹配；正常 `BuildKodex/Kodex` 仍为 main，未切换或修改。
- 三项已采纳消融保持不变。当前组合回归依据为
  [simplified-01](file:///home/stream/ACodeSpace/local/kodex-backend-daemon-evidence-787/final/simplified-01/README.md#L1-L55)
  的20组、1,014次显式重跑，不使用含旧 XML 的历史986汇总。
- 本轮只读源码并复核已有 fixture/XML，没有新跑故障注入、Gradle、
  远端设备、真实 Home 或模型；没有创建临时副本、进程及资源占用。
  [behavior-review scope](file:///home/stream/ACodeSpace/local/kodex-backend-daemon-evidence-787/final/behavior-review-01/mac/behavior-review-final/scope.json#L1-L14)
  明确19次执行中5次为故障观察，通过不是修复。

### 问题2：结构化父子关系存在，业务退出联动缺失

- **代码事实**：直接父子结构是 `runBackendHost coroutineScope` →
  `withLocalBackendServer coroutineScope` → `work launch`；
  `expiry/signals` 是 work 的兄弟。服务图由 work 直接调用 scoped 工厂，
  auth/global/Session 的 supervisor owner 再挂在各层工厂 scope 下。
  不能把此问题描述为 startup work 无父 Job 或泄漏了独立全局 scope。
- **已复现**：登录工厂独立抛出 `CancellationException`，work 的 catch
  原样重抛；宿主仍 Active，认证 status 为 Starting，独占锁仍占有。
  测试最后显式取消宿主后才释放。JVM 与 Mac Native 均有观察用例。
  [实际注入及断言](file:///home/stream/ACodeSpace/local/kodex-backend-daemon-evidence-787/final/behavior-review-01/mac/behavior-review-final/test-fixtures/app/impl/backend-host/src/commonTest/kotlin/io/github/stream29/kodex/backend/host/BackendReviewObservationTest.kt#L18-L41)。
- **终止性推导**：主控制流只等待 `lifecycle.awaitStop()`；
  work 的独立取消不会完成 stop，Starting 又不参与 idle expiry。
  因而没有新 stop/OS 信号/父取消就没有自动退出路径；不是做了无限时长测试。
  fixture 使用 TTL=-1，Starting 不受有限 TTL 回收的结论另由源码支撑。
- **机制**：普通 launched child 以取消结束也不向上取消父 Job；
  “换成普通 Job”不能补上关键任务退出联动。不能反过来将所有局部取消
  包装为非取消异常，把取消升级成全局故障。
  [work 与唯一 stop 等待](file:///home/stream/ACodeSpace/local/kodex-backend-daemon-787/app/impl/backend-host/src/commonMain/kotlin/io/github/stream29/kodex/backend/host/BackendHost.kt#L82-L145)、
  [Starting 不到期](file:///home/stream/ACodeSpace/local/kodex-backend-daemon-787/app/impl/backend-daemon/src/commonMain/kotlin/io/github/stream29/kodex/backend/BackendLifecycleImpl.kt#L89-L99)。
- **同类静态风险**：运行期整个 work 独立取消时也没有宿主联动；
  Ready 下 TTL=-1 或前端仍续期时，不能指望空闲退出兜底。此分支未新增实测，
  不把 startup 观察测试当成运行期故障已经复现。

### 故障矩阵：执行归属与故障策略分开

下表为本轮静态核验；“已有观察”仅指上方独立 startup 取消。
正常完成必须按业务角色判断，不能把所有长期/后台 Job 的完成一概判错。

| 对象 | 正常完成 | 独立局部取消 | 父 owner 取消 | 非取消失败 |
| --- | --- | --- | --- | --- |
| 宿主 work / 整个服务图 | 生产 block 在 Ready 后 awaitCancellation，无正常自退出路径；宿主也未观察完成 | Starting 挂住已有观察；运行期同类缺口为推导 | 宿主 finally 取消并等待 work，再关网络与锁 | catch 请求 stop；异常因果链交由主线问题4 |
| 全局 accepted work `inBackend` | 返回 R，不结束 owner | 该 work 结束，不取消同级业务；RPC waiter 取消本身不取消它 | 随 Session host owner 取消、join | supervisor 隔离；Deferred 由 waiter await，waiter 离开后的通用观察出口不足 |
| Session operations / runningTurn | 保持结果语义，turn 清理后释放真实 Job 槽 | 显式取消 turn 不应取消整个 Session/Daemon | 到期先取消 operations，再取消 Session并等待 | 单次操作隔离；Agent 已有业务失败通知，不应另造重复通知 |
| Session renewal / expiry / completion jobs | expiry 卸载及 completion 结束有合法正常路径 | `background` 原样重抛取消，不通知 failure monitor；意外失去续期/到期维护需单独验证 | 随 host owner 结束 | 非取消异常送 backgroundFailure，由普通 monitor launch 升至 scoped 宿主 |
| MCP service 配置 collector | 源为 StateFlow，没有业务上的正常耗尽 | finally 释放 service 并取消其 scope，但 global owner 无对应失效接线 | global close + owner join 兜住生命周期 | service 收尾；supervisor 隔离，上层 Ready 不表达此失效 |
| MCP manager 状态 collector | 源不应耗尽；本身没有完成出口 | collector 可停、manager scope 仍活，servers 保留最后值 | manager close，再等 global owner | supervisor 隔离，缺业务失效出口；不等于普通 MCP server 连接失败 |
| OAuth attempt | 提交/清理 attempt 槽，不结束 OAuth owner | 显式撤销或等待过期结束 attempt 属预期 | OAuth closeAndJoin 等 owner | attempt Deferred 与 authorization 保存失败；已返回 URL 后的观察边界另核验 |

源码锚点：
- [真实 child helper](file:///home/stream/ACodeSpace/local/kodex-backend-daemon-787/utils/coroutines/spec/src/commonMain/kotlin/io/github/stream29/kodex/utils/coroutines/ChildScope.kt#L22-L54)、
  [Session host 监视与释放](file:///home/stream/ACodeSpace/local/kodex-backend-daemon-787/rpc/impl/server/src/commonMain/kotlin/io/github/stream29/kodex/rpc/server/BackendSessionHost.kt#L59-L83)、
  [accepted work](file:///home/stream/ACodeSpace/local/kodex-backend-daemon-787/rpc/impl/server/src/commonMain/kotlin/io/github/stream29/kodex/rpc/server/BackendSessionHost.kt#L162-L188)、
  [维护任务与取消分支](file:///home/stream/ACodeSpace/local/kodex-backend-daemon-787/rpc/impl/server/src/commonMain/kotlin/io/github/stream29/kodex/rpc/server/BackendSessionHost.kt#L243-L314)。
- [MCP service collector](file:///home/stream/ACodeSpace/local/kodex-backend-daemon-787/mcp/impl/composition/src/commonMain/kotlin/io/github/stream29/kodex/mcp/impl/McpServiceImpl.kt#L58-L69)、
  [manager supervisor/collector](file:///home/stream/ACodeSpace/local/kodex-backend-daemon-787/mcp/impl/composition/src/commonMain/kotlin/io/github/stream29/kodex/mcp/impl/McpManagerImpl.kt#L64-L112)、
  [OAuth attempt 清理](file:///home/stream/ACodeSpace/local/kodex-backend-daemon-787/rpc/impl/server/src/commonMain/kotlin/io/github/stream29/kodex/rpc/server/BackendOAuth.kt#L79-L128)。

### 资源屏障矩阵：先证明释放，不以 timeout 代替

| 资源 | 获得/取消交接 | owner 结束屏障 | 当前判断 |
| --- | --- | --- | --- |
| 文件租约 | publication 前已挂真实 lifetime child；失败获取等 rollback | lifetime join + 保存的 completion result | 已有可复用模式；单纯 join 不会自动读取保存的失败，不能推定所有调用方均已观察 |
| 进程 acquisition | OS acquire 前安装 owner child guard，返回交接前捕获 session | 未交接时 guard 等 acquisitionDone，closeAndJoin并读取 releaseResult | 已有取消交接治理，不应改造成调用者取消后泄漏子进程 |
| JVM / POSIX process | 各有 cancellation guard、exit observer 和 releaseResult | closeAndJoin 等释放结果、guard、observer | JVM close 的同步 waitFor、POSIX observer 的 waitpid 仍需论证终止性；7秒协程 timeout 不是任意 OS 阻塞有界的证明 |
| Daemon instance guard | scoped 宿主最外层取得 | work清理、网络结束后最外层 finally 释放 | 不能为解决 Starting 挂住而提前释放锁，或用根 timeout 放弃还存活的资源 |

- [租约 lifetime 与结果屏障](file:///home/stream/ACodeSpace/local/kodex-backend-daemon-787/utils/filesystem-lease/src/commonMain/kotlin/io/github/stream29/kodex/utils/filesystemlease/FileSystemLeaseImpl.kt#L39-L105)、
  [进程 acquisition guard](file:///home/stream/ACodeSpace/local/kodex-backend-daemon-787/utils/process-client/impl/src/commonMain/kotlin/io/github/stream29/kodex/utils/processclient/ProcessClient.kt#L54-L117)。
- [JVM close / closeAndJoin](file:///home/stream/ACodeSpace/local/kodex-backend-daemon-787/utils/process-client/impl/src/jvmMain/kotlin/io/github/stream29/kodex/utils/processclient/JvmProcessClient.kt#L83-L132)、
  [POSIX close / exit observer](file:///home/stream/ACodeSpace/local/kodex-backend-daemon-787/utils/process-client/impl/src/posixMain/kotlin/io/github/stream29/kodex/utils/processclient/PosixProcessClient.kt#L200-L268)。
- 当前进程 close 已有终止自己所拥有子进程的行为；它不等于管理 CLI
  获得强杀 Daemon 的权限。不在研究阶段修改任何终止策略。
- `NonCancellable + dispatcher` 返回到已取消调用方时的结果交接仍需专项验证；
  文件租约已用外层仅 NonCancellable、内层切 dispatcher 保存结果作对照。
  原始失败及 suppressed 的实现协调交给主线问题4，不并行修改。

### 候选最小治理路线 — 未确认、未授权实施

1. **补 owner 退出联动，不新增权威状态。** 宿主同时处理 stop 与核心 work
   退出；区分父取消/已接受停止和核心任务独立退出，包括取消与意外正常完成。
   发现核心失效时关闭准入；取消请求、释放完成、锁释放是不同节点。
   沿现有 phase + stop Deferred 收尾，不恢复重复 DTO，不自动重启或强杀。
2. **按角色界定监督策略。** accepted 命令保留隔离、后端接受和 RPC 等待分离；
   失去关键维护循环不能仅记录日志后继续假装健康。MCP整体失效的监督边界
   尚未确认；先解释“与后端同寿命”为何未被当前接线完整保证，
   不预设需要新增 MCP 降级运行模式；
   单个 MCP server 断连、OAuth撤销、Agent取消继续沿现有局部语义。
3. **补 owner 本地结果观察。** 区分维护故障和命令结果；waiter 离开后的失败
   需要后端观察责任，不靠取消 accepted work 解决，也不重复 Agent 通知。
4. **逐个证明叶子屏障。** 先核验进程/Hook/租约的正常释放、局部取消、
   父取消及清理竞态，再决定是否需要局部实现调整。没有 deadline 不单独判错，
   不以根 timeout、全局活跃表或整体替换 SupervisorJob 代替证明。

候选接线可以是 scoped 控制流同时等待退出与停止，或本地完成信号；
尚未选定 helper/API。完成回调只适合快速非挂起通知，不能承担挂起清理；
也不能假设只等最终 completion 就能在下层清理阻塞时及时关闭准入。

### 首次讨论反馈 — 尚未作出故障策略决定

- 用户追问：“为什么MCP这种生命周期和后端等同的东西会非预期退出？”
  没有选择整体停机或局部隔离，不能将此追问记录为实施授权。
- “与后端同寿命”是应保障的业务契约；挂在父 Job 下只保证父结束时向下取消，
  不能单独保证某条必要 collector 一直工作到父结束。
  manager 这里使用 supervisor 隔离，且未保存/观察 collector 的终态。
- StateFlow 不会正常耗尽；常规连接/初始化失败已有 client Failed 状态，
  不是 manager collector 退出的已证实触发条件。目前没有生产 MCP 整体退出
  的复现，只有故障传播接线的静态缺口，不能把假设故障包装成日常行为。
- 前一问过早将“整体停机/切片降级”交给用户选择；先厘清哪些节点必须与
  后端同生共死、哪些连接/操作允许局部失败，再确定需要的监督接线。

### 用户确认的后续沉淀方向

- 用户确认未来需要将“不同 scope 的业务协程的结构化关系”写入 checklist。
  这是后续规范沉淀要求，不是立即修改产品、建立框架或选择 MCP 故障策略的授权。
- 后续条目需从业务关系说明 owner、同寿命的必要任务、可独立失败的子操作，
  并明确正常完成、局部取消、父取消、非取消失败及资源收尾的责任；
  不能只检查是否存在父 Job，或只规定使用 Job / SupervisorJob。
- 先继续逐项讨论并核验，再协调公共 checklist 的落点及修改；本次仅记录在
  讨论卡，尚未改写 `checklist/coroutine-resource-lifecycle.md`。

### 业务协程树草案 v1 — 待逐层确认

- 用户要求细化协程树。本节是目标业务所有权草案，不是当前 Job 树的精确转录，
  也不是实施/API授权；先确认业务边，再选 scoped 调用、Job 或 SupervisorJob。
- `[必]` 表示上级可服务时不可缺少的职责；职责失效必须使上级失效或收尾，
  不把仍 Active 的空 scope 当作健康。正常启动和既定停止期间另有合法阶段。
- `[隔]` 表示允许独立结束的业务子单元；结束不直接结束同级和上级，
  但必须完成结果观察、状态失效及清理。隔离不等于无人观察。
- `[借]` 是使用依赖，不是父子边；树内节点是 owner/任务/职责组，
  不要求每个模块、对象或职责组都新建一个 Job。

```text
BackendHost（后端进程中的根生命周期）
├─ 宿主控制职责 [必]
│  └─ 启动/停止协调、空闲到期、OS停止信号
├─ 网络服务职责 [必]
│  ├─ 管理接口及业务监听
│  └─ Connection[n] [隔]
│     ├─ transport收发与连接存活职责 [必于该连接]
│     └─ RPC调用等待 / 远程状态订阅 [隔]
└─ 后端核心服务图 [必]
   ├─ 统一设置、认证、模型/账户服务本体 [必]
   │  └─ 单次加载/刷新/维护尝试 [隔，沿现有业务失败或重试策略]
   ├─ MCP管理与配置协调职责 [必]
   │  └─ MCP server connection[n] [隔]
   │     └─ SDK client / transport / stdio进程资源
   ├─ OAuth服务本体 [必]
   │  └─ Attempt[exact-id] [隔，可撤销/到期]
   ├─ 后端业务通知消费者职责 [必]
   │  └─ 单次Hook执行 [隔，执行内拥有进程及临时IO]
   ├─ 已接受全局操作[n] [隔]
   └─ SessionHost服务职责 [必]
      ├─ Session续期/到期/失活协调职责 [必]
      └─ Repository（首次实际访问才取得）
         └─ Session[exact-owner] [隔]
            ├─ 存储缓存、AgentState/runtime及其实际子任务（须逐项分类）
            ├─ Session所属资源（文件租约、工具及可跨turn存续的shell）
            ├─ 已接受Session操作[n] [隔]
            │  └─ 实际执行Job（执行时占用runningTurn槽）
            │     └─ 本次模型请求/工具调用/临时资源
            └─ 自动标题等辅助请求 [隔，不独立续命]
```

树的解释与未决边界：
- 顶层“宿主 ↔ 核心服务图”的同寿命，以及图中必要服务集合是候选方向，
  尚未获得本轮逐项确认；不能据此宣称用户已授权 MCP 故障导致全后端退出。
- RPC waiter 只等待 accepted work，不是其父 Job。状态订阅随连接关闭，
  已接受全局/Session操作则随自己的业务owner；这里有跨树 await，不改成
  caller取消后连带取消work，也不引入断线命令队列。
- 全局操作画在核心图下表达业务归属；当前实现仍借助SessionHost.inBackend
  接受，不因本图立刻拆出新scope或修改接入API。
- runningTurn是操作当前执行Job的槽，不另造一个并行“turn管理scope”。
  MCP调用在调用它的操作/turn中执行，借用后端MCP连接；连接不是调用的父Job。
  标题是Session辅助工作，不挂在一次append RPC或runningTurn下面。
- Session TTL、真实Session owner及exact binding继续作为权威；
  本图没有新增“Session residency wrapper”。维护任务画在SessionHost下，
  针对已捕获Session/binding工作，不能在失效后自动操作新owner。
- 续期/到期任务会因卸载、替换或宿主停止合法结束；`[必]`约束维护职责
  不被静默丢失，不要求每一代timer永远不完成。外部连接失败、无认证、
  刷新失败或某次Hook失败均不直接等于服务本体消失。
- FileSystemLease原语仍遵守“租约失效/续期失败只结束资源child，不自动取消
  父owner”的既有契约。业务层如何观察并处理不在原语里暗改；
  本图的`[必]`不是给所有资源child添加自动向上取消。
- 前端各自是另外的进程根，不是BackendHost的子Job；TUI内的本地页签/草稿
  owner与连接binding分开。其详细树另轮讨论，本轮不接管主线心跳恢复设计。

资源与收尾不是第二棵权威活跃树：
- 实例锁、prepared Home、HTTP client等由对应owner持有并在finally释放；
  普通句柄不冒充Job。外部资源用实际清理child或scoped释放接入owner屏障。
- 清理依赖按“消费者先结束、借用资源后关闭”排序；例如Session/请求结束
  后才能关共享HTTP client。父子关系保证等待，不能单独表达全部借用依赖。
- 宿主进入Stopping先关业务准入，随后按既定顺序取消并等待业务与连接资源；
  网络和Home释放结束后最后放实例锁。不把管理stop已接受回执当退出屏障，
  也不因采用同寿命关系而跳过该顺序。
- 必要子任务的非取消失败可沿普通结构化失败路径传播；独立取消和意外正常
  完成仍需owner的局部退出联动。SupervisorJob只用于确实允许独立结束的边界，
  不用全局替换或全局Job表实现本图。

Session分支用词校正：
- 用户追问“什么叫作必要状态同步职责”。此前用该抽象词询问“单Session失活/
  整后端退出”缺少具体任务前提，撤回该问法；用户没有作出该故障策略选择。
- 当前没有一个统一的“Session后台状态同步协程”。AgentState的state/latestIndex
  主要在业务操作中随存储修改发布；CachedIndexVersioned的正常尾索引也在set
  路径更新，不能为它们虚构一个长期同步Job。
- 实际要列的是：存储缓存的定期清理任务、AgentState/runtime scope下的
  MCP工具派生状态收集、工具资源owner、标题任务，以及Session下operations
  supervisor中的命令Job。RPC订阅则归连接/collector，不是后端Session的
  统一同步任务。必要性和退出策略需对每个真实任务分别论证。
- [操作内发布state/latestIndex](file:///home/stream/ACodeSpace/local/kodex-backend-daemon-787/agent-state/impl/state/src/commonMain/kotlin/io/github/stream29/kodex/agentstate/impl/KodexAgentStateImpl.kt#L338-L372)、
  [缓存定期清理](file:///home/stream/ACodeSpace/local/kodex-backend-daemon-787/agent-session/impl/filesystem/src/commonMain/kotlin/io/github/stream29/kodex/agentsession/filesystem/CachedAgentStorage.kt#L159-L168)、
  [缓存操作内更新尾索引](file:///home/stream/ACodeSpace/local/kodex-backend-daemon-787/agent-session/impl/filesystem/src/commonMain/kotlin/io/github/stream29/kodex/agentsession/filesystem/CachedAgentStorage.kt#L283-L329)。

本轮补核源码（只读，未新增动态验证）：
- [真实Session owner与租约](file:///home/stream/ACodeSpace/local/kodex-backend-daemon-787/agent-session/impl/filesystem/src/commonMain/kotlin/io/github/stream29/kodex/agentsession/filesystem/FileSystemKodexAgentSession.kt#L27-L68)、
  [Repository打开并保留原Session](file:///home/stream/ACodeSpace/local/kodex-backend-daemon-787/agent-session/impl/filesystem/src/commonMain/kotlin/io/github/stream29/kodex/agentsession/filesystem/FileSystemKodexSessionRepository.kt#L205-L228)。
- [真实runningTurn槽](file:///home/stream/ACodeSpace/local/kodex-backend-daemon-787/agent-runtime/impl/composition/src/commonMain/kotlin/io/github/stream29/kodex/agentruntime/impl/KodexAgentRuntimeComposition.kt#L140-L183)、
  [标题辅助任务](file:///home/stream/ACodeSpace/local/kodex-backend-daemon-787/rpc/impl/server/src/commonMain/kotlin/io/github/stream29/kodex/cli/sessiontitle/AgentTitleGeneration.kt#L23-L85)。

### 后续验证门槛（尚未运行）

- startup 独立取消、核心 work 意外完成/取消、父取消、stop/完成竞争，
  均不能留下无人工作的 Starting/Ready；不将普通业务取消升级为停后端。
- 关键 collector 故障与单 MCP client 故障分开验证；不重启执行、不重放命令。
- waiter 取消后 accepted work 继续，后续普通失败有后端观察；成功与预期取消
  不误报，Agent通知不重复。
- 资源交接前后取消、双重close、释放失败、owner join屏障各自验证；父owner
  完成时资源释放已结束，saved failure不因只join而丢失。
- 需新增动态验证时先与787协调远端私有副本/共享重型锁，不复用主线运行中的
  source/cache/scripts，不在本机/用户IDE运行重量级Gradle。

本轮核对的官方 kotlinx.coroutines 1.11.0 依据：
[Job取消不向上等同失败](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-job/)、
[SupervisorJob隔离与观察](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-supervisor-job.html)、
[withContext取消交接](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/with-context.html)、
[完成回调约束](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-job/invoke-on-completion.html)。

## 首轮现状 — 2026-10-14

- 审查对象为本机 `feat/backend-daemon` 未提交工作区，
  基线 `3223a52d`，前置验证源码清单 `source-manifest-boundary-03.sha256`。
  本轮没有修改产品源码、占用远端构建或运行真实 Home。
- 生命周期已经是分层结构，不是所有后台工作均游离于 Job 树。
  owner 树中 guard/handle/turn 槽不是额外 Job，
  借用服务对象、状态投影及 TTL 不等于所有权。

### 用户反馈与实际启停测量

- 用户澄清：若能确认不会永久等待，没有 deadline 本身不算大问题；
  优先确认 `kodex backend start/stop` 的响应速度。后续以终止性和
  真实响应为判断，不以机械增加 timeout 作为整改目标。
- 已用相同 boundary-03 Debug 二进制完成 Mac/Linux 各 40 轮：
  新 Home、已有 Home、一个/两个真实在线 TUI，每组 10 次。
  首轮 exploratory 批次另保留，不混入正式样本。
- 已有 Home 的启动到 Ready 中位数：Mac 17.37 ms、Linux 48.24 ms；
  stop 命令完整返回：空闲 Mac 21.97 ms、Linux 12.33 ms；
  两 TUI 时 Mac 41.07 ms、Linux 12.00 ms。
- stop 到实际进程退出：空闲 Mac 22.34 ms、Linux 17.59 ms；
  两 TUI 时 Mac 1065.22 ms、Linux 1045.84 ms。每轮都独立取得原
  内核锁确认释放，无卡住，测试 Home 已删除，Mac 共享设备锁已释放。
- 约 1 秒在线连接退出时间与 Ktor 3.5.0 默认 graceful shutdown 窗口
  相符；尚未做逐阶段 profiling，不把此归因写成精确实测阶段耗时。
- 这是无真实模型/凭据、无配置 MCP 的普通启停证据，不证明任意
  running turn/外部失控资源也必然结束。叶子资源的等待终止性仍单独核查。
- [完整方法、最大值与原始样本](file:///home/stream/ACodeSpace/local/kodex-backend-daemon-evidence-787/final/backend-response.md#L1-L79)。

### Owner 树

```text
Backend process: runBackendHost coroutineScope
├─ stable instance guard                 由最外层 finally 释放
├─ network engine / WS connection owners 服务图外的宿主资源
├─ expiry monitor / OS signal poller
└─ work launch
   ├─ prepared Home handle               服务图结束后 closeAndJoin
   └─ withBackendServices coroutineScope
      ├─ supervisor owner                auth / OAuth / account / HTTP client 消费者
      └─ withBackendGlobalState coroutineScope
         ├─ supervisor owner             MCP manager/service、模型目录、设置投影
         └─ withBackendSessionHost coroutineScope
            ├─ background failure monitor
            └─ supervisor owner          已接受全局工作、repository、TTL jobs
               └─ repository / Session owners
                  ├─ state/storage/lease/resource children
                  ├─ operations supervisor / accepted command jobs
                  └─ runningTurn         当前真实执行 Job 的槽，不是新建全局 Job

TUI: withLocalRpcClient coroutineScope
├─ reconnect loop / heartbeat / transport supervisor children
└─ withFrontendApplication coroutineScope
   └─ frontend Job
      ├─ required global subscriptions
      ├─ local tab/draft/component owners
      ├─ exact-generation Session RPC binding owners
      └─ settings command queues / frontend error Hook
```

### Owner 与释放契约

| 切片 | 当前 owner / 取消边界 | 释放或故障出口 |
| --- | --- | --- |
| Daemon | 宿主 work，与 TUI/管理请求无父子关系 | 关闭业务准入，取消服务图并等待，再关网络及实例锁 |
| 已接受全局工作 | Session host 的 `owner.async`；RPC caller 只等待 | caller 取消不取消已接受工作，宿主结束则取消 |
| Session 操作 | 原 Session 下 operations supervisor | 到期先取消操作再取消 Session；nonce/绑定失效不换 owner |
| runningTurn | runtime 使用调用协程的真实 Job，CAS 占槽 | 执行/清理结束才清槽，续命不扩展到标题、Hook、等待 UI |
| MCP | global owner 下 service/manager，client owner 锁控制连接 | close 请求取消，服务 lifecycle finally 释放，global owner join 才是父级屏障 |
| OAuth | backend auth owner 下独立 supervisor 和 attempt Deferred | 10 分钟等待截止；撤销取消 attempt；closeAndJoin 等 owner，TUI callback listener 独立 |
| 模型/账户刷新 | 生产使用 `owned...` 挂父 Job | close 取消子 scope，backend/global owner join 后再关闭借用 HTTP client |
| Hook | 一个消费者属于各自 backend/frontend owner，命令资源属单次 operation | 不在停止时排空通知；操作 finally 终止/等待当前进程 |
| 文件租约 | acquisition 前即挂入 owner 的 lifecycle Job | 有保存的 completion result；物理清理有 deadline，close 请求不等于 closeAndJoin |
| 前端设置队列 | command worker 属 application scope，不属 popup | popup 关闭可排空；application 取消可中断，`drained` 区分结果 |
| RPC transport | 每次连接的 supervisor；不拥有 accepted backend work | socket completion/传输失败取消 transport；shutdown 关闭并等待其子节点 |

源码锚点：
- [宿主停止顺序](file:///home/stream/ACodeSpace/local/kodex-backend-daemon-787/app/impl/backend-host/src/commonMain/kotlin/io/github/stream29/kodex/backend/host/BackendHost.kt#L82-L155)、
  [服务图释放](file:///home/stream/ACodeSpace/local/kodex-backend-daemon-787/rpc/impl/server/src/commonMain/kotlin/io/github/stream29/kodex/rpc/server/BackendServices.kt#L52-L105)。
- [后台与 Session 接受边界](file:///home/stream/ACodeSpace/local/kodex-backend-daemon-787/rpc/impl/server/src/commonMain/kotlin/io/github/stream29/kodex/rpc/server/BackendSessionHost.kt#L156-L188)、
  [续命与到期释放](file:///home/stream/ACodeSpace/local/kodex-backend-daemon-787/rpc/impl/server/src/commonMain/kotlin/io/github/stream29/kodex/rpc/server/BackendSessionHost.kt#L243-L313)、
  [真实 turn 槽](file:///home/stream/ACodeSpace/local/kodex-backend-daemon-787/agent-runtime/impl/composition/src/commonMain/kotlin/io/github/stream29/kodex/agentruntime/impl/KodexAgentRuntimeComposition.kt#L140-L183)。
- [MCP lifecycle](file:///home/stream/ACodeSpace/local/kodex-backend-daemon-787/mcp/impl/composition/src/commonMain/kotlin/io/github/stream29/kodex/mcp/impl/McpServiceImpl.kt#L58-L68)、
  [OAuth attempt owner](file:///home/stream/ACodeSpace/local/kodex-backend-daemon-787/rpc/impl/server/src/commonMain/kotlin/io/github/stream29/kodex/rpc/server/BackendOAuth.kt#L69-L125)、
  [生产模型刷新 owner](file:///home/stream/ACodeSpace/local/kodex-backend-daemon-787/openai/model-catalog/src/commonMain/kotlin/io/github/stream29/kodex/openai/modelcatalog/OpenAiModelCatalogStoreImpl.kt#L85-L94)。
- [连接关闭](file:///home/stream/ACodeSpace/local/kodex-backend-daemon-787/rpc/impl/local/src/commonMain/kotlin/io/github/stream29/kodex/rpc/local/LocalBackendClient.kt#L105-L174)、
  [TUI 收尾](file:///home/stream/ACodeSpace/local/kodex-backend-daemon-787/app/impl/application/src/commonMain/kotlin/io/github/stream29/kodex/cli/app/Application.kt#L173-L190)。

## 问题分类

### A. 停止链的终止性与可观测性尚需完整论证

- **代码事实**：宿主 `work.cancelAndJoin`、Session 的 operations/session join、
  transport `socket.close` 和 Hook 最后的 `exitCode.await` 在
  `NonCancellable` 中没有该层 deadline。宿主 5 秒预算只等待 stop reply，
  Hook 10 秒预算只覆盖输入/正常退出，不覆盖其 finally。
- **风险推导**：一个下层清理或不可协作操作停住，可能将整个 Daemon
  长期留在 Stopping 并持锁；不是已经观察到普通 stop 会卡死。
- 缺少 deadline 不是独立判错依据；若叶子资源可以证明有限完成，
  上层 join 可以不另设 timeout。普通启停已有上方实际响应证据。
- 不能简单给根层加 timeout 后释放锁：那会违反全部资源结束后才放锁。
  需要先给叶子资源定义可终止/可等待契约，并记录停止阶段和阻塞 owner。
- [Hook 预算与最后等待](file:///home/stream/ACodeSpace/local/kodex-backend-daemon-787/hook/impl/notification/src/commonMain/kotlin/io/github/stream29/kodex/cli/notification/NotificationHooks.kt#L105-L132)。

### B. 有 timeout 不等于阻塞清理有界

- **代码事实**：JVM ProcessSession 的 `closeAndJoin` 有 7 秒
  `withTimeout`，但调用的同步 `close()` 中仍有无参数 `process.waitFor()`。
  协程 deadline 不能仅靠存在 timeout 就证明这一同步等待可中断。
- 同一 closeAndJoin 还用 `NonCancellable + Dispatchers.IO` 一次切换；
  leaseCleanup 已采用外层仅 NonCancellable、内层切 dispatcher 的保存结果模式。
  两者返回到已取消调用方的失败保留语义需要用同样矩阵核对。
- 本次 Native 演示不能证明 JVM 路径，也不能证明恶意/失控进程场景；
  暂不把现有 force-termination 后正常 wait 误称必现死锁。
- [JVM 进程 close](file:///home/stream/ACodeSpace/local/kodex-backend-daemon-787/utils/process-client/impl/src/jvmMain/kotlin/io/github/stream29/kodex/utils/processclient/JvmProcessClient.kt#L83-L131)、
  [已有租约清理对照](file:///home/stream/ACodeSpace/local/kodex-backend-daemon-787/utils/filesystem-lease/src/commonMain/kotlin/io/github/stream29/kodex/utils/filesystemlease/FileSystemLeaseSupport.kt#L24-L46)。

### C. 后台失败的观察责任没有完整覆盖

- **代码事实**：Session host 的 renewal/expiry `background` 有显式失败
  monitor；通用 `inBackend` 的已接受 `owner.async` 则只有 RPC waiter await，
  没有对应的后端完成结果观察出口。waiter 离开后，普通命令失败不保证上报。
- **边界**：Agent resume 自己有日志/通知、标题有 best-effort 策略、认证维护
  有重试诊断；不能把所有 supervisor child 都说成没有失败处理。
- **需要验证**：用已接受的设置/管理操作，在 waiter 取消后注入失败，
  检查至少一个真实后端观察出口、无误报取消、无重复 Agent 通知。
  不为解决观察问题取消已接受工作或重新绑定到 RPC caller。

### D. 宿主根错误的因果链丢失

- **确定实现缺口**：`runBackendHost` 将非取消异常保存为 `failed` 并请求
  停止，但最后抛出的通用“初始化失败”异常没有携带原 `failed`，
  捕获点也没有记录该异常。运行阶段故障同样走“初始化失败”文字。
- prepared Home `closeAndJoin` 的 finally 没有像
  `closeBackendResources` 一样将清理失败附加给原失败；这一异常链需统一核对。
- 这是错误可诊断性问题，不是需要用户重新决定业务失败归属。
- [宿主异常出口](file:///home/stream/ACodeSpace/local/kodex-backend-daemon-787/app/impl/backend-host/src/commonMain/kotlin/io/github/stream29/kodex/backend/host/BackendHost.kt#L112-L156)、
  [现有保留原失败的 helper](file:///home/stream/ACodeSpace/local/kodex-backend-daemon-787/rpc/impl/server/src/commonMain/kotlin/io/github/stream29/kodex/rpc/server/BackendCleanup.kt#L6-L22)。

### E. 部分子服务失败可能留下看似有效的快照

- **代码事实**：MCP service/manager 在 supervisor 下启动长期 collector。
  service collector 结束会释放/取消 service；manager collector 没有显式终止
  状态出口。宿主 Ready 当前表示服务图已初始化，不监控每条 collector 的健康。
- **风险推导**：非预期 collector 失败可能只被协程异常处理器处理，
  上层 StateFlow 仍保留最后值；不能据此断言每次 MCP 连接失败都会停止服务，
  因为普通连接失败已有独立 client 状态。
- 需区分“单个 MCP client 失败”和“服务观察循环失效”，验证后者是否使
  原服务 owner 失效并被上层识别；不能通过把所有 supervisor 改成 Job 来解决。
- [MCP manager 长期观察](file:///home/stream/ACodeSpace/local/kodex-backend-daemon-787/mcp/impl/composition/src/commonMain/kotlin/io/github/stream29/kodex/mcp/impl/McpManagerImpl.kt#L64-L111)。

## 不应在本轮误判的项目

- 生产模型目录/账户服务使用 owned factory；独立 public factory 的默认
  `SupervisorJob()` 不是当前 Daemon 根路径的无父 Job 证据。
- 文件租约的清理在真实 child 中执行并保存结果，Session owner 结束会等待
  该 child；`invokeOnCompletion { lease.close() }` 不能单独证明“未等待释放”。
  但只有 join、不读取 saved completion 的调用者仍需检查失败观察。
- 设置队列保证 popup 关闭后按序排空，不保证整个 TUI 退出后无限排空。
  当前 root 取消会中断未提交的命令，这与队列 `drained=false` 契约一致；
  不自行扩大为所有前端退出必须等待后端写入。
- 状态值、TTL 计时器和资源 Job 不是三个互相替代的生命周期管理器。
  全局“管理”首先应是各 owner 的共同契约与诊断，不是另建权威活跃表。

## 面向未来 MCP / Tool as CLI 的核验

- 用户要求带着未来前端承载 MCP/工具的需求检查本期是否挖坑，
  不提前实施该功能。对应
  [Tool as CLI 讨论卡](2026-10-03-discuss-integrate-tool-cli-into-kodex.md#L16-L34)：
  配置的 MCP 与选定原生工具可被 CLI/交互前端调用，长寿命资源仍属 Daemon，
  不是将 SDK client、进程或原始凭据 owner 搬进 TUI。
- 本次只读核验 boundary-03 实现，未修改产品源码或运行下一期功能测试。
  下方区分可复用接点与未来接入条件，不把尚未设计的工具 API 当成本期缺陷。

### 当前没有封死的路径

- MCP service/manager、认证和连接在 backend global owner 下，
  与 tab、TUI 及特定 Agent 无父子关系。Agent 消费同一 `mcpService`；
  后续无需为每次工具 CLI 调用再启动一份 MCP 图。
- 业务客户端没有 Mosaic/ViewModel/前端设置依赖，`withLocalRpcClient`
  block 可以直接使用 RPC，不强制进入 `withFrontendApplication`。
  `FrontendRpcConnection` 中 frontend 表示逻辑客户端，不只表示 TUI。
- 业务 server 接受服务注册回调，不知道 tab/工具/Session 类型。
  当前注册八项服务是组合根选择，不是 transport 的固定服务列表。
- 全局 accepted work 已有 `inBackend` owner；该方法本身不打开 repository。
  名称在 Session host 下有耦合，但没有强制独立工具先创建或打开 Session。
- `BackendLifecycle.renew()` 的实现不依赖 Agent 类或 Job 身份，
  源的选择在本期接线/契约中。以后新增获准保活来源无需替换锁、IPC 或状态机。
- 管理协议保持 start/stop/status/reload 的小范围，与业务协议分离。
  未来工具调用属于业务能力，不应为了 CLI 易兼容而塞入管理 HTTP 通道。

### 用户澄清后的判断

- **现有流式恢复不是执行重放**：用户确认当前采用 index / StateFlow
  状态订阅，而不是重启原始执行流。重连恢复订阅不重新执行工具，幂等性
  已由该边界保障。撤回将“未来假设的执行型 Flow”列为当前重放风险的判断。
- **独立工具将成为续命来源**：这是已明确的未来方向；当前只接前端心跳与
  runningTurn 是本期范围，不是漏掉工具续命。底层 renew 无 Agent 类型依赖，
  后续可按真实工具 owner 接入，不提前改变本期续命规则。
- **CLI 执行环境改造留在 Tool as CLI**：后续会处理 CLI 执行环境的修改。
  当前 cwd/环境安排不因此列为本期缺陷，不提前增加透传字段、修改共享 cwd
  或设计环境同步机制。

### 留给下一期工具契约的其他接入条件

| 条件 | 当前事实 | 未来误用后果 / 处理位置 |
| --- | --- | --- |
| Session 与工具进程 | 原生 Agent 工具由 Session/runtime 组装，Session TTL 为 60 秒；后台 heartbeat 不刷新 Session TTL | 独立工具不得为了复用而造一个隐藏 Agent/页签，也不能把持久 shell 偷挂到短 RPC 调用。按工具类别选择明确的 backend/task/Session owner |
| 执行取消与未知结果 | waiter 取消不取消已接受 work；命令不会自动重放；无通用工具 operation handle | Ctrl-C 的含义及结果查询/显式取消需要未来工具契约定义，不能直接等同于“关连接就是停止”，也不能失败后盲目重试 |
| 大输出 | 当前 WS 单帧上限 16 MiB，RPC 字符串消息，没有通用工具输出分块契约 | 大型 JSON/图片/批量结果不能默认一次塞入；未来选择有界分块、文件引用或明确限额，不以无限增大 frame 绕过内存预算 |

### 尚未设计不是当前挖坑

- 现有 MCP `handle` 消费 pending/stable Agent event，原生工具中的上下文数、
  标题、计划等也有 Agent 语义。它们不是现成独立 CLI API；
  对选定工具增加领域适配是预期下一期工作，不应直接序列化/导出 handler 对象。
- 未知结果、任务 ticket、审批、输出格式、stdout/stderr 和退出码是未来工具
  契约的一部分。现有通知是 replay=0 事件，不能当持久审批或结果队列。
- 每用户唯一 Home、同机协议匹配与后端实例重启失效，没有阻止本机多客户端
  工具入口；跨设备、多 Home 或新身份/权限隔离不是这次核验的前提。
- 前一节 C/D/E 的后台失败观察、因果链及服务失效可见性仍值得优先处理：
  对未来脚本调用尤其重要，但不因此先实现 Tool as CLI。

### 核验结论与后续门槛

- 没有发现需要撤销本期 Daemon 分层才能支持看板方向的结构性死路。
  不能据此宣称现有 RPC 已完整支持未来工具或已经验证相关行为。
- 当前状态订阅的幂等性已有保障；工具续命和 CLI 执行环境改造已明确留给未来。
  这三项不再作为当前风险或本期阻塞，也不重新要求用户决定其方向。
- 本期不增加工具 RPC、通用任务注册表、保活来源或前端资源图；
  上述条件进入后续设计/生命周期核验，不用预留空接口改变现有协议。

源码锚点：
- [客户端重订阅边界](file:///home/stream/ACodeSpace/local/kodex-backend-daemon-787/rpc/impl/local/src/commonMain/kotlin/io/github/stream29/kodex/rpc/local/LocalBackendClient.kt#L44-L76)、
  [底层续命与不可逆停止](file:///home/stream/ACodeSpace/local/kodex-backend-daemon-787/app/impl/backend-daemon/src/commonMain/kotlin/io/github/stream29/kodex/backend/BackendLifecycleImpl.kt#L62-L95)。
- [MCP 图与借用 Agent 依赖](file:///home/stream/ACodeSpace/local/kodex-backend-daemon-787/rpc/impl/server/src/commonMain/kotlin/io/github/stream29/kodex/rpc/server/BackendServices.kt#L65-L85)、
  [服务注册接点](file:///home/stream/ACodeSpace/local/kodex-backend-daemon-787/rpc/impl/server/src/commonMain/kotlin/io/github/stream29/kodex/rpc/server/BackendServices.kt#L108-L125)。
- [Agent 工具的 cwd/context](file:///home/stream/ACodeSpace/local/kodex-backend-daemon-787/agent-runtime/impl/composition/src/commonMain/kotlin/io/github/stream29/kodex/agentruntime/impl/KodexAgentTools.kt#L39-L96)、
  [底层 exec 工厂默认 cwd](file:///home/stream/ACodeSpace/local/kodex-backend-daemon-787/tool/impl/unified-exec/src/commonMain/kotlin/io/github/stream29/kodex/tool/unifiedexec/UnifiedExecToolClient.kt#L190-L199)。
- [独立原生进程启动](file:///home/stream/ACodeSpace/local/kodex-backend-daemon-787/app/impl/backend-daemon/src/nativeInterop/cinterop/backend_platform.def#L90-L126)、
  [MCP 配置的 cwd/environment](file:///home/stream/ACodeSpace/local/kodex-backend-daemon-787/mcp/spec/stdio/src/commonMain/kotlin/io/github/stream29/kodex/mcp/stdio/McpStdioTransport.kt#L36-L45)。
- [帧预算与服务无关的注册回调](file:///home/stream/ACodeSpace/local/kodex-backend-daemon-787/rpc/impl/local-server/src/commonMain/kotlin/io/github/stream29/kodex/rpc/localserver/LocalBackendServer.kt#L29-L37)。

## 首轮建议验证顺序 — 历史记录，接管顺序以上节为准

1. D 的根异常/清理异常保留，以及 C 的 waiter 离开后失败观察。
2. A/B 的叶子资源清理屏障：故障注入不结束的 exit/close、取消后的 dispatcher
   交接、正常/超时/双重失败；确认父 owner 完成时资源结果已可观察。
3. E 的长期 collector 异常：子服务失效状态、重建/拒绝准入及失败传播。
4. 再形成跨层最小契约：接收与执行分离、关闭准入、请求取消、等待释放、
   保存失败、停止诊断；以本地 owner 为权威，不先设计全局 Job 注册中心。

依据为源码和官方 API，不是新回归结果：
- [SupervisorJob：失败隔离需有对应观察策略](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-supervisor-job.html)。
- [NonCancellable：清理及 dispatcher 交接注意事项](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-non-cancellable/)。
- [Job completion handler：同步且不能挂起等待](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-job/invoke-on-completion.html)。
- 官方页面与工作区依赖均为 kotlinx.coroutines 1.11.0；不能用 completion
  handler 代替挂起清理，也不能用协程 timeout 假定任意阻塞调用可被终止。

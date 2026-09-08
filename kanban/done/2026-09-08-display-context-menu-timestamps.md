# Task Tree

- [done] 在 Session 与历史消息右键菜单展示日期时间
  - [done] 核对菜单入口与既有时间戳来源
  - [done] 确认展示格式、入口覆盖与 fallback 行为
  - [done] 确认时间戳读取方案与验收方式
    - [done] 静态审查读取、刷新和菜单状态阻塞点
    - [done] 确认已缓存 timeline 的首时间读取
    - [done] 确认未打开 Session 的 Catalog 首时间接入
    - [done] 确认菜单刷新时机
    - [done] 确认时间字段失败隔离与加载展示
    - [done] 确认既有设置更新的时间语义
    - [done] 确认窄菜单允许裁剪
    - [done] 确认由 agent 在 tmux 独立 window 端到端验收
  - [done] 复审并制定具体实施计划
  - [done] 实现按需时间读取
    - [done] Session repository 与 ViewModel 分别读取创建和更新时间
    - [done] 两套历史 ViewModel 精确读取 Message 时间并校验目标
  - [done] 接入四个菜单与本地日期时间格式
    - [done] 每次菜单打开使用独立 request 身份
    - [done] 时间行纳入 Index 只读信息块，避免异步插入使动作焦点跳转
  - [done] 验证取值、失败隔离与菜单交互
    - [done] 执行相关模块 JVM 测试
    - [done] 构建含布局修复的最终 Linux x64 CLI
    - [done] 在独立 tmux window 验收四入口、时间语义和失败隔离
    - [done] 确认侧栏窄终端绘制异常的处理范围
    - [done] 定位并修复已授权的侧栏绘制异常
    - [done] 补做窄终端与侧栏动画验收
  - [done] 整理持久约束、验证结果并清理临时环境

# Details

- 状态：Done；时间戳功能与用户追加授权的布局修复均已实现并验收，持久约束及验收记录已整理，临时环境已清理。未创建 Git commit。
- 本文路径以仓库根目录为基准；不涉及 KRPC 或其他并行任务。
- 下方 Discussion 基线及原审阅事实保留为决策背景，不代表修改后的代码现状；当前约束以[右键菜单日期时间](../../checklist/context-menu-timestamps.md)为准，最终结果见“实现与验收记录”。
- 审阅方式：已提交意见已整理为下方决定；仍可在各项 `FIXME:` 后补充自由意见。空白表示暂无评论，不视为确认待决事项或阶段推进授权。
- 用户要求：
  - 保留 Index 信息，并在右键菜单补充日期时间，而非仅时分秒、相对时间或耗时。
  - Message 展示对应消息的日期时间。
  - Session 同时展示创建时间与最近更新时间（Updated at）；创建时间按用户定义取最早的第一个时间戳。

FIXME:

## 已确认的展示与取值决定

- 格式：运行 CLI 设备的本地时区，24 小时制，`YYYY-MM-DD HH:mm:ss`，附 UTC 偏移；紧邻 Index 下方分行展示。
- 标签：Message 使用 `Timestamp`；Session 使用 `Created at`、`Updated at`。
- 范围：Session Catalog、已持久化 Session Tab，以及主历史和 History Index 侧栏中的 Message；主历史 Message 菜单补 Index。
- 不扩展非 Message、折叠 WorkGroup、pending/streaming，不改变各入口现有菜单开启条件。
- Fallback：缺少精确时间、读取失败和加载中均隐藏对应时间字段，不显示 `Unavailable`、`Error` 或 `Loading…`；不借用邻近时间。未物化 New Session 不展示持久化时间。
- Message 取对应 index 的精确 timestamp。
- Session 创建时间按 timeline/index 顺序取首条已存 timestamp，不遍历求最小墙钟值；Updated at 取当前 timeline 最新记录，复用 `lastActivityAt`。
- Catalog 的创建时间直接读取 `timestamp[0]`；缺少 index 0 按本入口读取失败处理，隐藏 Created at，不枚举后续索引寻找首条。已打开 Session 仍使用缓存上的 `ceilToIndex(0)`。
- 接受 fork 继承历史首时间和 revert 后更新时间回退；rename/settings 已写入的 timestamp 自然影响 Updated at，不额外增加操作审计时间，也不排除已有记录。
- 每次打开菜单重新读取，读取完成后在本次菜单内保持快照；目标失效仍须关闭或丢弃结果。
- 窄菜单允许裁剪，不要求必须保留日期、秒或偏移；不以完整时间展示为由要求额外展开或换行。
- 上述决定已根据文档意见及后续逐项讨论更新；验收由 agent 在 tmux 独立 window 中运行 CLI 端到端完成，不再要求用户逐项审批验收清单。

FIXME:

## Discussion 阶段基线（实现前）

- Session Catalog 与已持久化 Session Tab 都有只读 Index 菜单项：
  `Kodex/app/view/application/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/SessionTreeCliScreen.kt:1122-1145,1326-1351`。
- History Index 侧栏菜单有 Index；主历史消息菜单当前只有 Revert/Fork，没有 Index。后续需区分这两个入口，不能按“所有历史菜单已有 Index”实施：
  `Kodex/app/view/application/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/SessionSidebar.kt:898-925`、
  `Kodex/app/view/application/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/SessionTreeCliScreen.kt:1412-1432`。
- `SessionCatalogEntry.lastActivityAt` 已存在，但没有创建时间；已打开的 `PersistedSessionViewModel` 也尚未提供这两个日期时间字段：
  `Kodex/app/contract/session-catalog/src/commonMain/kotlin/io/github/stream29/kodex/app/sessioncatalog/contract/SessionCatalogEntry.kt:5-10`、
  `Kodex/app/contract/session/src/commonMain/kotlin/io/github/stream29/kodex/app/session/contract/PersistedSessionViewModel.kt:13-21`。
- Filesystem Catalog 的 `lastActivityAt` 取 timestamp timeline 最新记录；正常快速路径读取 latest 指针，不打开 Agent runtime：
  `Kodex/agent-session/filesystem/src/commonMain/kotlin/io/github/stream29/kodex/agentsession/filesystem/FileSystemKodexSessionRepository.kt:348-370,414-425`。
- Message Ready 状态目前只有 event、elapsed 和 turnDuration，没有绝对时间：
  `Kodex/app/contract/history/src/commonMain/kotlin/io/github/stream29/kodex/app/history/contract/item/MessageHistoryItemViewModel.kt:27-32`。
- 现有 elapsed 查询已使用 `timestamp.getExact(index)`；普通 `get(index)` 是稀疏 timeline 的前值查询，不能据此把别条记录时间当成消息自己的时间：
  `Kodex/app/viewmodel/history/src/commonMain/kotlin/io/github/stream29/kodex/cli/history/IndexHistoryRead.kt:174-183`、
  `Kodex/agent-storage/contract/src/commonMain/kotlin/io/github/stream29/kodex/agentstorage/contract/IndexVersioning.kt:6-35`。
- 已打开 Session 的 timestamp timeline 使用完整内存索引缓存；`ceilToIndex(0)` 在内存中二分查询，不枚举目录。时间值的缓存未命中仍可能读取单条文件：
  `Kodex/agent-session/filesystem/src/commonMain/kotlin/io/github/stream29/kodex/agentsession/filesystem/CachedAgentStorage.kt:26-41,165-176,187-192`。
- Catalog 是另一条读取路径：`listEntries/getEntry` 经 `rootEntry` 新建未包装缓存的 `FileSystemAgentStorage`，正常路径读 latest 指针；不能把已打开 Session 的缓存事实直接推广到 Catalog：
  `Kodex/agent-session/filesystem/src/commonMain/kotlin/io/github/stream29/kodex/agentsession/filesystem/FileSystemKodexSessionRepository.kt:68-94,347-356`。

FIXME:

## 审阅结论

- 已处理本轮全部已填写意见；不再保留被替代的 fallback 文案、实时刷新或窄菜单完整展示建议。
- 已缓存路径直接 ceil；Catalog 缺少 index 0 时隐藏创建时间，不扫描寻找后续首条。首时间读取阻塞已收口。
- Discussion 待决项已收口；本次复审未发现新的需求阻塞，依据用户后续明确授权制定实施计划。

FIXME:

### 1. 首时间读取（已确认）

- 用户决定：复用已有完整索引缓存，直接对 index 0 执行 ceil，再读取对应首条 timestamp；不因查询位置较早就增加目录枚举。
- 已核对：`CachedIndexVersioned.ceilToIndex(0)` 符合该要求；本轮撤回对此路径的重复扫描担忧。
- 事实：正常初始化写入 `timestamp[0]`，常见 Session 可直接读取首时间：
  `Kodex/agent-storage/contract-ext/src/commonMain/kotlin/io/github/stream29/kodex/agentstorage/contract/ext/AgentStorageInitialization.kt:12-18`。
- 仅未缓存路径：`FileSystemIndexVersioned.ceilToIndex(0)` 在 `0.json` 存在时直接返回；缺少时才枚举并排序 timestamp 索引文件名：
  `Kodex/agent-storage/filesystem/src/commonMain/kotlin/io/github/stream29/kodex/agentstorage/filesystem/FileSystemIndexVersioned.kt:64-68,162-167`。
- Catalog 当前不接入已打开 Session 的 timeline cache；为未打开 Session 新建 cache 也不是零扫描，现有缓存构造会先读取全部 stored indexes：
  `Kodex/agent-session/filesystem/src/commonMain/kotlin/io/github/stream29/kodex/agentsession/filesystem/CachedAgentStorage.kt:98-108`。
- 用户决定：未打开 Session 的 Catalog 入口缺少 index 0 时，视为一次创建时间读取失败，套用既有 fallback 隐藏 Created at；即使后面存在 timestamp，也不继续寻找首条。
- Catalog 创建时间使用精确 index 0 读取，不调用会回退扫描的底层 `ceilToIndex(0)`；不为本字段新增索引缓存或持久化指针。
- 仅隐藏失败的 Created at，成功读取的 Updated at、Index 和已有菜单动作不受影响。
- 已打开 Session 的缓存 ceil 规则不变。因此缺少 index 0 时，Tab 可能展示后续首条时间，而 Catalog 隐藏 Created at；不能再把这种可见性差异认定为入口不一致缺陷。
- 继续遵守 Catalog 正常快速路径不枚举 timeline、不为列表展示打开 Agent runtime 的边界：
  `checklist/cli-session-view-models.md:77-80`。

FIXME:

### 2. 菜单刷新时机（已确认）

- 事实：Catalog 通过刷新取得列表快照；已打开 Session 的 `refresh()` 当前只刷新名称：
  `Kodex/app/viewmodel/session/src/commonMain/kotlin/io/github/stream29/kodex/cli/session/SessionViewModels.kt:283-299,579-600`。
- 决定：采用原 A。每次打开菜单重新读取，读取完成后在本次菜单内保持快照，不在菜单打开期间持续订阅时间更新。
- 两个 Session 入口采用相同刷新规则；同一存储状态下双方成功展示的对应字段取值一致，不承诺不同打开时刻数值相同。Catalog 缺少 index 0 时隐藏 Created at 的例外见第 1 项。
- 目标失效仍须关闭或丢弃异步结果，不因保持快照而保留失效目标。

FIXME:

### 3. 时间字段失败隔离（已确认）

- 事实：Message 当前是整条 Loading/Ready/Failed；读取过程异常会进入整条 Failed：
  `Kodex/app/viewmodel/history/src/commonMain/kotlin/io/github/stream29/kodex/cli/history/MessageHistoryItemViewModel.kt:21-43`。
- 事实：Catalog 刷新失败恢复旧状态并抛异常，当前不支持新增时间字段的独立失败处理：
  `Kodex/app/viewmodel/session/src/commonMain/kotlin/io/github/stream29/kodex/cli/session/SessionViewModels.kt:579-600`。
- 决定：缺失或读取失败时隐藏对应时间字段；不显示失败占位，不影响原本可读的消息、Session 行、其他成功时间字段、Index 和已有菜单动作。
- 隐藏的是新增时间字段，不改变原有消息内容已损坏时的失败展示，也不扩大原有菜单可用条件。

FIXME:

### 4. 加载期间的展示（已确认）

- 决定：加载也按 fallback 隐藏，不显示加载占位；成功取得时间后再插入字段，缺失或失败则保持隐藏。
- 不等待全部时间加载成功才展示已有菜单；字段出现后菜单尺寸可能变化，后续需验证位置和焦点不异常。

FIXME:

### 5. 既有 rename/settings 更新的时间语义（已确认）

- 新确认事实：rename 经 `updateThreadName` 调用 settings 更新，而 settings 更新已经写入新的 timestamp：
  `Kodex/app/shared/session-title/src/commonMain/kotlin/io/github/stream29/kodex/cli/sessiontitle/AgentTitleGeneration.kt:117-123`、
  `Kodex/agent-state/contract/src/commonMain/kotlin/io/github/stream29/kodex/agentstate/contract/KodexAgentStateExtensions.kt:13-15`、
  `Kodex/agent-state/impl/src/commonMain/kotlin/io/github/stream29/kodex/agentstate/impl/KodexAgentStateImpl.kt:398-405`。
- 因此，“不额外增加 rename/archive 操作审计时间”不等于“rename 不影响 Updated at”。
- 决定：采用原 A，保持 timeline 首尾语义；rename 等既有 settings 更新自然推进 Updated at，不新增也不排除这类记录。
- 本项只确认展示结果，不授权修改 timestamp 写入行为。

FIXME:

### 6. 窄菜单与验收方式（已确认）

- 事实：现有菜单限制宽高，并按条目实际高度安排可见区域；尚未验证新增完整日期时间和 UTC 偏移后的效果：
  `Kodex/app/view/components/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/components/TuiPopupMenu.kt:433-478`。
- 决定：采用原 B，允许裁剪时间信息，不要求必须保留日期、秒或偏移，不增加完整时间必须另行可达的要求。
- 用户允许菜单放宽宽高限制并按内容尺寸展开；不是要求为新增时间设置固定尺寸或强制换行，也不是本轮修改通用菜单的授权。
- 静态边界：现有菜单宽度已按内容固有宽度计算，再限制到父布局最大宽度；宽高必须有界。不能把宿主可用尺寸上限误报成任意业务固定上限：
  `Kodex/app/view/components/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/components/TuiPopupMenu.kt:440-458`。
- 验收决定：后续实现后，由 agent 在 tmux 新建独立 window，实际运行 CLI，按本文已确认行为直接端到端验收；具体检查由 agent 负责，不再把验收清单作为待用户审批的阻塞点。
- 使用任务相关环境，不干扰已有 window 或用户 Session；结束后清理自建测试 window 和临时数据。
- 按项目规则执行相关检查，并区分编译、测试与实际端到端结果；对未覆盖或受阻部分如实说明，不以编译或单元测试代替菜单运行验证。

FIXME:

## 技术约束与验证状态

- 复用既有 `kotlin.time.Instant` 与 timestamp timeline，不根据 Index 或文件 mtime 推算时间；格式化遵循
  [日期与时间约定](../../checklist/datetime.md)；该 checklist 规定时间类型，不预先规定本任务的显示格式。
- 读取经现有 ViewModel/storage 边界，避免渲染线程 I/O；不为新增字段让每次列表刷新全量扫描历史。
- History Index 侧栏使用另一套 ViewModel，当前只接收 index timeline；只给 Message Ready 增加时间不能覆盖侧栏：
  `Kodex/app/viewmodel/agent/src/commonMain/kotlin/io/github/stream29/kodex/cli/agent/HistoryIndexViewModel.kt:35-40,84-101`。
- 两个历史入口已有 generation 校验可复用；后续时间读取也必须校验目标，不另建全局状态机制：
  `Kodex/app/viewmodel/agent/src/commonMain/kotlin/io/github/stream29/kodex/cli/agent/HistoryIndexViewModel.kt:141-161`、
  `Kodex/app/viewmodel/history/src/commonMain/kotlin/io/github/stream29/kodex/cli/history/MessageHistoryItemViewModel.kt:30-40`。
- 八个相关模块共 253 项 JVM 测试通过；包含布局修复的最终 Linux x64 CLI 构建及实际 tmux 验收均已通过，见下节。

FIXME:

## 实现与验收记录

- Session repository 和两种 Session ViewModel 使用独立按需读取方法；没有新增持久化字段、索引扫描或全局缓存。
- Message child 和 History Index 都执行精确 timestamp 查询并校验 generation；时间读取不改变原消息内容状态。
- 菜单 request 使用每次打开的对象身份；时间行放在既有 Index 只读信息块中，避免异步插入独立菜单项导致 Fork 焦点跳回 Revert。
- 统一本地格式与读取快照见 `Kodex/app/view/application/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/ContextMenuTimestamp.kt:23-58`。
- JVM：`app-view-application`、`app-view-history`、`app-viewmodel-agent`、`app-viewmodel-session`、`app-viewmodel-history`、`agent-session-filesystem`、`agent-session-in-memory`、`app-viewmodel-application` 的 `jvmTest` 共 253 项通过；覆盖精确读取、缺少 0 不扫描、首时间按索引而非墙钟、字段隔离、重开刷新、过期结果丢弃、加载后保持动作焦点，以及 Session/New Session 小视口绘制。
- 编译：`:app-cli:linkReleaseExecutableLinuxX64` 成功；显式使用运行中 Gradle Daemon 的 Java 25 路径。首次尝试因原 Daemon 忙而未复用，不计作有效验证；后续构建复用 Daemon。Native 构建提示 configuration-cache 不兼容并丢弃缓存，不影响编译成功。
- 实际运行：独立 tmux window、独立 HOME、无用户认证及真实 Session 数据；最终构建复验 Catalog、Tab、主历史、History Index 四入口，显示本地秒级时间和 UTC 偏移。
- 实际行为：消息精确时间缺失时保留 Index/动作、隐藏 Timestamp；非 Message 不扩展时间；Catalog 缺少/损坏 0 只隐藏 Created at；Updated at 损坏只隐藏该字段；菜单打开后保持快照，重开读取新值。
- 实际语义：rename 推进 Updated at；fork 继承首时间；revert 后 Updated at 回退至保留历史的末时间。已打开 Session 缺少 0 时通过缓存首索引展示 Created at，Catalog 对应字段隐藏。
- 最终构建 32×18 终端：Catalog、Tab、主历史、History Index 四入口均已验证允许裁剪时间并保留动作；受阻的侧栏路径已在修复后补验收通过。
- 最终构建连续 3 轮执行 110×32 ↔ 32×18 切换及侧栏展开/收起步骤，CLI 保持运行；另验证在 32 列直接打开侧栏和消息菜单不再崩溃。
- 长期约束已收录于[右键菜单日期时间](../../checklist/context-menu-timestamps.md)和[TUI 交互组件](../../checklist/tui-interaction-components.md)。
- 未执行 macOS、Windows、Linux ARM64 实机验收；本项使用当前 Linux x64 环境。
- 清理：已正常退出测试 CLI，确认 Session/Home lease 文件释放；自建 tmux window、独立测试 HOME、复制的 CLI、截图文本和临时构建日志已删除，未干扰原有 window 或用户 Session。

FIXME:

## 验收中发现并修复的布局异常

- 现象：`Mosaic TextSurface.get → textLeaderAtOrNull → TextCanvasDrawScope.drawRect → BackgroundModifier.draw` 抛出 `IllegalStateException: Check failed`，CLI 退出。
- 最小复现：新启动 CLI，打开测试 Session，不打开任何右键菜单；展开再收起侧栏，并在动画期间从 110×32 缩小至 32×18，同样触发异常。32×18 下展开侧栏也触发过同类异常。
- 无侧栏动画的 New Session 和 Session 缩放，以及 32 列下前三类菜单的显示已成功；因此不把所有窄终端显示都描述为失败，也不把此异常归因于时间字段读取。
- 本轮没有修改 Mosaic、通用菜单或侧栏尺寸/动画实现；没有在未修改版本上做二进制基线复现，定位与修复证据如下。
- 用户决定：将上述侧栏绘制问题纳入本任务修复，并补做受阻的窄终端验收。不另建或推进其他任务，不据此扩大到无关布局重构。
- 定位与修复：无菜单的 JVM 回归先复现同一 `TextSurface.get` 横向越界。`StatusBarLayout` 无界测量控件自然宽度，但没有把绘制限制在分配宽度内；增加既有 `clipToBounds()`，保留自然宽度换行规则与 Mosaic 严格边界检查。只改状态栏局部视口，不修改 Mosaic 或侧栏动画。
- JVM 验证：`Kodex/app/view/application/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/app/SessionViewportTest.kt:18-97` 覆盖终端/侧栏宽度改变、主区仅剩 1 列，以及 New Session 小视口；修复后两项均通过。最终 native 构建按原路径补验收通过。
- 修复入口：`Kodex/app/view/application/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/RuntimeStatusBar.kt:151-153`；未新增布局模式或全局容错。

FIXME:

## 实施计划

- Repository 增加独立的精确首时间与最新时间读取；Catalog 正常路径不扫描、不打开 runtime，保留失效 latest 指针的既有恢复边界。
- Persisted Session 从自身已缓存 storage 读取首尾时间；Catalog 通过自身 repository 按需读取，不增加列表字段缓存或全局状态。
- Message child 增加按需精确时间读取，主历史菜单传递准确 Message handle；History Index 从已有 index 条目判定 Message，再读取精确 timestamp。读取前后校验 generation。
- Renderer 以本次菜单 request 为生命周期，异步独立读取字段；取消继续传播，普通读取失败仅隐藏该字段。菜单动作立即可用，不持续订阅时间。
- 保持 Index 只读菜单项的身份，将时间作为该信息块内的新增行，而非动态插入菜单项；已验证异步加载不把 Fork 焦点移回 Revert。
- 四入口共用本地秒级格式化及只读时间项；复用当前版本的 kotlinx-datetime，不升级依赖版本，不改 timestamp 写入行为。
- 测试覆盖精确取值、Catalog 缺失 0 不扫描、首尾语义、字段失败隔离、菜单重开与失效、只读信息和原动作；实际 tmux 验收四入口与窄终端。
- 构建显式复用运行中 Gradle Daemon 的 JVM；测试与实际 CLI 运行分别记录，不能互相替代。

FIXME:

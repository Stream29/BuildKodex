# CLI ViewModel 状态与懒 History

## 状态所有权原则

- Application、Session、Agent 和 NewSession ViewModel 不得用一个聚合 `StateFlow` 承载全部可变 UI 状态。
- 父 ViewModel 只发布自身状态、父级关系和稳定 child ViewModel handle；child mutable state 只由对应 child 发布。
- 只有真实状态机或必须一起校验、提交的字段才组成一个不可变 state，例如 composer text/cursor/revision 和 request-user-input
  args/answers；不要机械地为每个字段建立 Flow 或 wrapper。
- Settings 按字段命令必须由 owning ViewModel 基于最新完整快照原子更新。
- 稳定 identity、parent address 和生命周期 handle 使用只读属性，不包装成不会变化的 `StateFlow`。
- 高频 composer edit 和 stream delta 不得复制或重新发布 Session catalog、Agent topology、settings、完整 history 或
  application popup state。
- ViewModel registry 只保存子 ViewModel 引用和 identity，不把子 ViewModel 的 mutable state 扁平复制到父 ViewModel state。
- Derived state 从唯一真源按需组合；不得为了方便 renderer 在多个 ViewModel 中保存可独立漂移的副本。
- 单一 command serialization boundary 不要求单一 state object；命令顺序和观察状态的粒度必须分开设计。
- Frontend 直接使用准确的 child ViewModel，并只 collect 当前组件消费的 Flow；顶层 screen 不得汇总
  application/session/agent 的全部字段再向下转发。

## Application与Session状态

- Application ViewModel 只发布 navigation、当前独占 popup 与稳定 child handle。
- Navigation state 只由有序 `List<SessionViewModel>` 和 `selectedIndex` 组成；`selected` 从同一 snapshot 的下标派生，不保存
  active handle、业务 key 或 revision。
- Application popup state 只能是 `Closed` 或一个具有对象身份的 `Open` handle；不得使用 `content + requestId`、消息队列或消费式
  effect 模拟 popup。
- 每个 open popup 直接携带可渲染的准确 child ViewModel；Application 必须先完成 child 创建，成功后用一次 state update 替换
  popup 并关闭旧 child，创建失败则保持原 popup。
- `dismissPopup(expected)` 只关闭仍为当前值的 exact open handle；旧 popup 的异步 completion 不得关闭后来打开的 popup。
- `SessionCatalogViewModel` 只在 Select Session popup 打开时创建；构造时不读取数据，frontend 取得 child 后才调用
  `refresh()` 并 collect `sessions`。
- Settings popup 携带绑定准确 target 的 `SettingsViewModel`；Rename/Delete popup 分别携带自己的 child ViewModel，draft
  与确认命令不放回 Application state。
- Settings、models 与 authentication 不由 Application 暴露；Global Settings、New Session、Agent 和 Login 等准确 child 由实现层通过
  constructor injection 获取真源。
- `OpenAiAuthStore.state` 继续是后端凭据真源；只有需要展示认证信息的 Settings/Login child 发布去敏状态，frontend contract
  不得暴露 access token。
- `OpenAiAuthState.Unavailable` 必须使用明确原因枚举，不得用自由文本 message 代替认证状态分类。
- Persisted Session ViewModel 直接暴露稳定 `rootAgent` 和 `StateFlow<AgentViewModel>` `selectedAgent`，另行发布自身
  summary、轻量 topology、lifecycle 和 notification。
- Session ViewModel 持有 Agent ViewModel registry，但 frontend 必须直接订阅 selected Agent ViewModel；不得增加 selection
  wrapper 或镜像 Agent mutable state。
- Agent topology 更新不得触发已打开 conversation history、composer 或 Agent settings 的重新发布。
- Selected Agent 切换必须先 materialize 并取得稳定 handle，再原子发布该 handle；不能短暂发布无法解析的 address。

## Agent与NewSession状态

- Agent ViewModel 直接持有 composer、history、request-user-input 和 Shell registry child handle，并发布完整
  `StateFlow<KodexAgentSettings>`、execution、token count、direct children、history action、notification 与 lifecycle；
  committed、pending tool 和 streaming History 状态统一由 history child 发布。
- `threadName` 和 `plan` 直接来自 `KodexAgentSettings`；不得为 frontend 再建立 Agent summary 或 plan state。
- Frontend 直接读取完整 `KodexAgentSettings` 真源；model、working directory、reasoning effort、service tier 与 collaboration
  mode 只能通过 owning ViewModel 的按字段更新方法修改，禁止 frontend 回传旧完整快照或建立 editable configuration 投影。
- Execution state 只包含 Agent phase、running、activity、capabilities 和需要一起显示的轻量运行信息；token count
  可以在不要求同批一致时独立发布。
- `AgentHistoryViewModel` 使用独立高频 streaming item state；已提交 child sequence 不得随每个 delta 重建。
- Request-user-input child 以 call id 为 replacement boundary，并原子表达 args、answer draft、revision 与 submission
  phase；history revert confirmation 使用独立 Agent-owned action state。
- NewSession ViewModel 只持有进程内 `MutableStateFlow<KodexAgentSettings>` 和 composer；`materialize()`
  在自身命令边界捕获并消费两者，直接返回 `PersistedSessionViewModel`。
- Materialization 不发布额外 state/request/result；`ApplicationViewModel.materializeNewSession(tabIndex)` 在自身串行化边界内解析当前
  tab slot，调用其中的 exact New Session child，并用一次 navigation update 原位替换返回的 persisted child。
- Materialize 的 `tabIndex` 无效或当前不再指向 New Session 时必须失败且不修改 navigation；child 失败保留
  draft，成功替换保持列表长度与 `selectedIndex`。

## 懒History数据源

- Agent ViewModel 不在任何普通 state 中保存完整 `List<ConversationHistoryItem>`。
- 每个 Agent ViewModel 持有一个 `AgentHistoryViewModel`；后者拥有 committed、pending tools、streaming item 和 History
  滚动状态，不建立第二套 raw history repository。
- History View 明确由 committed stable items、零到多个 pending tool items 和至多一个 streaming item 组成；三部分独立
  发布，只有 committed 部分使用 `HistoryItemViewModel`。
- 每个 committed 一级 item 使用一个具有稳定对象身份的 sealed `HistoryItemViewModel`；`AgentHistoryViewModel` 管理这些
  child 的 newest-first 线性窗口。
- Committed 线性窗口以单个不可变快照发布；generation、size、`peek(index)` 与 `get(index)` 必须属于同一个
  `HistoryItemWindow` 实例，已发布旧窗口在被替换后仍保持可索引。
- 自动折叠落地前，一个 `HistoryItemViewModel` 对应一个 committed stable event；未来一个 child 可以覆盖线性相邻的多个
  stable event，History View 仍只消费同一套一级 item contract。
- Pending tools 与 streaming item 只使用朴素 `StateFlow`，不实现 committed row interface，也不建立 child ViewModel。
- Agent ViewModel 不再发布供 renderer 二次拼接的 `AgentStreamState`；pending steer 仍是独立的 Agent 状态。
- 当前只支持单一 Mosaic frontend；每个已 materialize Agent 的 `AgentHistoryViewModel` 可以直接持有实际
  `LazyListState`、scroll interaction、follow-latest 和 child 展开状态，不支持同一实例被多个 frontend surface 同时测量。
- Sealed `HistoryItemViewModel` 不发布聚合 `StateFlow` 或通用 state DTO。Message 只保存 stable index；reasoning、tool 和 patch
  保存 stable index 与 expanded state；plan update 和 context compaction 只保存 stable index；未来 group 保存 index range。
- Child 不缓存 decoded event。Renderer 需要内容时通过底层 `IndexVersioned` raw-value LRU 读取。
- 每个 filesystem timeline wrapper 私有持有完整有序 stored-index 列表和独立 raw-value LRU；默认最大容量为 1,024。
- Full index 只在打开 timeline 时扫描一次，之后由 append/revert 增量更新；上层不得复制或取得 index list 与 LRU。
- 初次读取只物化最新的有限 batch；之后沿 `prevIndex/get` 读取有限的 `k` 个 event，允许 `O(k log n)`，不得增加目录扫描或
  仅为避免二分而暴露 range API。
- 每个 `HistoryItemWindow` 的 `peek(index)` 只返回已物化 child；`get(index)` 返回同一 child，并在该窗口仍为当前窗口且
  接近已加载旧端时向 ViewModel 注册合并后的加载需求。
- 每个成功加载的旧端 batch 追加到持久平衡序列，不复制全部 child；所有已物化 child 保留到 generation 失效。
- 未访问的久远历史不创建 child、不读取 raw value，也不计入 frontend `itemCount`。
- 自动折叠落地前，一个 committed stable event 严格投影为一个 history entry 和一个 frontend item；继续由
  `LazyColumn` 的正常 item access 声明式触发旧端加载。
- Frontend 不观察 viewport 来手动分页，不持有 history window/cache，也不主动回收已加载 child。
- `HistoryItemViewModel` 实例本身作为 History row 的稳定 key；不得再包装通用 `HistoryItemIdentity`。需要 revert、fork
  或其他 storage target 的 sealed variant 自己暴露准确能力，不把 target 强塞进所有 item。
- 可见 row 的 raw event 由 renderer 异步读取；读取完成前固定显示一行空白，读取失败显示一行红色 `Error` 并记录完整异常。
- Raw-value loading/error 只属于 renderer 的短暂投影，不进入 child contract；Compose 已加载内容存活超过底层 LRU 淘汰是可接受的。
- Raw index 与 value LRU 的 append/revert 更新继续由 AgentStorage 负责；History ViewModel 只更新 child sequence。
- Remote compaction 不改变当前 CLI 展示本地 committed history 的语义；History source 继续读取 storage history timeline，不把
  model-visible compacted prefix 混入 UI history。
- Completed-turn/checkout point 通过独立的按需查询或增量索引提供，不为了状态栏或普通 history rendering 每次扫描完整
  history。
- Agent ViewModel materialize 时只绑定现有 AgentSession storage、读取轻量状态和有限 tail；不得借 materialize 之名恢复完整
  history snapshot。

## History窗口失效与重载

- 普通 append 和旧端扩展保持 generation，并复用所有未变 child 实例与展开状态。
- Stable timeline 缩短或 external replacement 时递增 generation，丢弃全部旧 child，并从新的 tail 重新物化有限 batch。
- Destructive replacement 使用新 generation 原子替换完整 committed window；不得分别发布 item count、generation 与
  实际 sequence，也不得用越界占位 row 掩盖发布竞态。
- 旧 generation 的 row read、context action 和异步结果不得作用于当前 sequence。
- 普通更新由同一 child key 保持 `LazyListState` anchor；destructive reload 不复用旧 generation key，并将失效位置确定性夹取到新列表。
- AgentStorage 在 revert 时更新 stored-index cache 并整体清空对应 raw-value LRU；ViewModel 不得绕过或弱化这次完整缓存失效。
- Revert 后第一次 child classification 和 row read 可以是 cold read，但每次工作量必须受 batch/viewport 需求限制。
- History 条目操作使用真实 stable storage index，并以 `storageIndex + 1` 作为 exclusive boundary；执行前重新校验所选
  Session、Agent、generation、已物化 target 和 idle turn job。
- `Revert to here`只截断所选Agent的全部storage timeline suffix，并同步pending steer、自动标题one-shot gate及root Session
  catalog标题；确认后已接受的revert由Agent ViewModel lifetime持有，不依赖确认弹窗的协程。`Fork from here`由所属`PersistedSessionViewModel`使用exact Agent child将prefix复制成无descendants的新root
  Session，不修改source或Application navigation。

## Compose稳定性与缓存

- Application、Session 和 Agent ViewModel 引用必须具有稳定 identity；registry 对同一地址复用同一实例，Composable 不得在重组时重新包装
  ViewModel。
- Revert 与 Compose stability 不冲突：immutable 指每次发布的 snapshot/content 不会就地变化，stable 指所有可变结果都通过可观察
  state 发布；它们都不表示 timeline 永远不变。
- 只有真正满足 Compose stability contract 的类型才能标记 `@Stable` 或 `@Immutable`；包含 expanded state 的
  `HistoryItemViewModel` 是具有稳定身份的可观察 state holder，不得伪装成 immutable value。
- 普通 `List`、`Set` 和 `Map` 不因放进 data class 就自动稳定；committed sequence 变化时仍必须复用未变 child 实例。
- Kotlin 2.4 Compose compiler 的 strong skipping 可以跳过参数未变的 restartable Composable 并记忆 lambda，但 unstable
  参数按实例比较；不得每次 emission 重建等价 list、wrapper 或 callback 来抵消 skipping。
- Composable 边界至少拆到 application shell、Session surface、Agent surface、history list 和 history
  row；每层只接收该层需要的稳定参数，并在最靠近消费者的位置 collect 对应 child 的 state。
- History list 分别观察 committed window、pending tools 与 streaming item；LazyColumn 的 committed count、key、
  `contentType` 和 item lambda 必须闭包捕获同一个 window 快照。每个 committed row 是独立可跳过的 Composable，只接收对应
  `HistoryItemViewModel` 和展示依赖。
- LazyColumn item 直接使用 `HistoryItemViewModel` 实例和语义 `contentType`；普通更新保留未变 child，使已有 composition
  slot 可以移动、跳过或复用。
- 每个 `HistoryItemViewModel` 持有自己的展开状态，不把整份 expanded-id set 作为所有 row 的参数；child 被移除或 generation
  失效时一并释放对应状态。
- 纯展示派生值使用 `remember(item, width, expanded)` 缓存；storage 读取、history projection、命令结果和生命周期状态不得放进
  `remember` 充当业务缓存。
- Frontend 不用 `snapshotFlow` 将 scroll offset 转换成分页命令；旧端需求只由 composed item 调用 `get(index)` 注册。
- LazyColumn 使用 interval content 和 anchor 附近的 key-index map；不得为完整 `itemCount` 枚举 key 或建立全量反向 map。
- Hover、composer、activity 或无关 overlay 更新不得重新物化 history child 或枚举完整已加载 sequence。
- Subcomposition reuse budget 保持有界，并以 viewport、overscan、history row `contentType` 和滚动基准结果调优；不得把当前固定保留数量当成性能结论。
- 使用 Compose compiler stability/metrics report 和重组计数验证边界；不要用注解掩盖不满足契约的类型。

## Streaming与frontend衔接

- Committed children、pending tools 和 streaming item 由 History ViewModel 分开发布，renderer 在一个 LazyColumn 中组合。
- Streaming output 直接转发执行层的 replaying `SharedFlow`，不得把每个 delta 复制进 committed sequence。
- Pending tools 读取 latest snapshot 可见的 sparse unstable value；后续仅更新 settings 等 timeline 不能让仍 pending 的工具消失。
- 初始 child classification、旧端扩展和 row raw-value read 都在 render dispatcher 之外执行；composition 与 measure 不直接做
  storage I/O。
- Follow-latest 是 History ViewModel 持有的用户意图；流式新增和 row 高度变化只在该意图开启时请求 latest position。
- 通用 LazyColumn 只发布布局信息并执行虚拟布局，不依赖 AgentStorage、history cursor、prefetch job 或 conversation 模型。

## 验证

- 验证 composer 每次编辑只发布 composer state，不发布 history、execution、topology 或 catalog。
- 验证 stream delta 只更新 streaming/execution 状态，不扫描或复制 committed sequence。
- 验证 global settings、application popup 和 Session selection 更新不会重建无关 Agent history。
- 验证打开长 history 的 Agent 只沿 AgentStorage index cache 执行有限次 `prevIndex/get`，不复制 index list、不扫描完整
  timeline，也不建立第二套 raw page cache。
- 验证 LazyColumn 在 10,000 items 下只计算 anchor 附近的 key 并只组合有界 viewport/overscan 内容。
- 验证连续旧端 batch 追加后 child identity、展开状态、首尾顺序和 follow-latest 保持正确。
- 验证 append、stream commit、revert、Session fork 和 Agent ViewModel reopen 的 generation 行为。
- 验证 composer edit、stream delta、hover 和 application popup 不重组未变 history row；append 只影响 tail，展开只影响目标
  row。
- 验证 raw-value read 的一行 loading、success 和红色 `Error` fallback。
- 验证 revert 只提升一次 generation 并重载有限 tail，不扫描完整 history，也不复用旧 generation 的 child state。
- 验证 revert 对 raw-value LRU 执行完整失效，旧缓存值不会跨 generation 使用；首次窗口重载只读取有限项，随后重复访问命中重新填热的
  LRU。
- 记录超长 history 打开、连续向前滚动和流式追加时的 storage read、projection、item composition、recomposition 和 slot reuse
  计数。

# CLI ViewModel 状态与懒 History

## State切分原则

- Application、Session、Agent 和 NewSession ViewModel 不得用一个聚合 `StateFlow` 承载全部可变 UI 状态。
- 按所有权、原子一致性、更新频率和实际消费者划分 state slice；不要机械地为每个字段建立一个 Flow。
- 必须一起校验和提交的字段保留在同一不可变 slice，例如 composer text/cursor/revision、配置 draft/revision/dirty 和 request-user-input args/answers。
- 稳定 identity、parent address 和生命周期 handle 使用只读属性，不包装成不会变化的 `StateFlow`。
- 高频 composer edit 和 stream delta 不得复制或重新发布 Session catalog、Agent topology、配置、完整 history 或 overlay state。
- ViewModel registry 只保存子 ViewModel 引用和 identity，不把子 ViewModel 的 mutable state 扁平复制到父 ViewModel state。
- Derived state 从唯一真源按需组合；不得为了方便 renderer 在多个 slice 中保存可独立漂移的副本。
- 单一 command serialization boundary 不要求单一 state object；命令顺序和观察状态的粒度必须分开设计。
- Frontend 只 collect 当前组件消费的最窄 slice；顶层 screen 不得 collect 整个 application/session/agent 状态再向下传递全部字段。

## Application与Session slices

- Application ViewModel 分开发布 Session catalog、active Session/NewSession target、global overlay 和 application lifecycle。
- `KodexGlobalSettingsStore.settings` 与 model catalog 的现有 Flow 直接作为真源暴露；Application ViewModel 不复制完整快照到另一个 aggregate state。
- Session ViewModel 分开发布 root summary/aggregate running、Agent topology、selected Agent identity 和 Session lifecycle/notification。
- Session ViewModel 持有 Agent ViewModel registry，但 renderer 必须直接订阅 selected Agent ViewModel 的 slices。
- Agent topology 更新不得触发已打开 conversation history、composer 或配置 draft 的重新发布。
- Selected Agent 切换必须以 identity 与 registry 可用性作为一个原子 consistency boundary，不能短暂发布无法解析的 selection。

## Agent与NewSession slices

- Agent ViewModel 至少分离 configuration、composer、execution、stream tail、blocking interaction 和 history source。
- Configuration slice 原子包含 settings draft、revision、dirty、可配置状态以及与当前 model 相关的有效选择。
- Execution slice 只包含 Agent state、running、activity 和需要一起显示的轻量运行信息；token count 可以在不要求同批一致时独立发布。
- Stream tail 使用独立高频 slice；已提交 history 不得随每个 delta 重建。
- Blocking interaction slice 原子表达 pending checkout 或 request-user-input 及其 answer draft；auto-resolution timer 仍由 Agent ViewModel 持有。
- NewSession ViewModel 分离 defaults draft、composer 和 materialization；首次提交命令捕获三个 slice 的 immutable revision snapshot。

## 懒History数据源

- Agent ViewModel 不在任何普通 state slice 中保存完整 `List<ConversationHistoryItem>`。
- 每个 Agent ViewModel 暴露薄 `AgentHistorySource` 和独立 history window slice；source 只负责 storage 遍历、语义投影和有序 mutation event，不建立另一套 raw history repository。
- 复用 AgentSession 已有的完整 stored-index cache 与 raw-value LRU；Agent ViewModel 不得重复缓存 index list、raw page 或已解码 storage value。
- History cursor 使用 Agent storage 的 stored index、exclusive boundary 与 window generation，不使用页面内 ordinal、列表 offset 或终端行号。
- 初次读取捕获 `history.latestIndex()` 作为稳定 upper bound；之后从当前 stored index 反复执行 `get(index)` 和 `prevIndex(index)`，直到满足本批需求或到达 timeline 起点。
- `indexesDescending` 只视为上述循环的便捷表达；不得为 history UI 增加新的目录扫描或 range API。
- Agent ViewModel 只保留有界的 projected semantic window、两端 cursor 和独立 edge loading state；这不是 raw storage cache。
- 初次打开只读取尾部有限 window；LazyColumn 接近已加载前边界时再请求更早记录，远离视口的 semantic batch 可以按 cursor 回收。
- Fallback item identity 使用 window generation 与真实 storage index；provider item id 可参与 generation 内 identity，但不得使用当前 batch 中的相对 ordinal。
- Conversation projection 必须正确处理跨 batch 的 tool call/output 配对；可以扩展原始读取边界或在相邻 batch 合并时重归并，不得永久显示错误的 unmatched result。
- Raw index 与 value cache 的 append/revert 更新继续由 AgentStorage 负责；Agent ViewModel 只接收成功 mutation 的 index boundary 与类型并更新语义投影。
- Remote compaction 不改变当前 CLI 展示本地 committed history 的语义；History source 继续读取 storage history timeline，不把 model-visible compacted prefix 混入 UI history。
- Completed-turn/checkout point 通过独立的按需查询或增量索引提供，不为了状态栏或普通 history rendering 每次扫描完整 history。
- Agent ViewModel materialize 时只绑定现有 AgentSession storage、读取轻量状态和有限 tail；不得借 materialize 之名恢复完整 history snapshot。

## History窗口失效与重载

- `HistoryWindowSnapshot` 是一个有限且不原地修改的 value snapshot，包含 generation、entries、两端 cursor 和边界状态；collection 类型不满足 Compose contract 时不得强标 `@Immutable`，它也不代表完整 timeline。
- 普通 prepend、append、tool output 和 stream commit 保持 generation；新 snapshot 复用未变 entry 实例，只创建新增或内容确实变化的 entry。
- 成功 revert 直接递增 generation、取消旧 generation 的在途加载并使整个 semantic window 失效；不尝试复用旧窗口中的局部语义对象。
- Revert 后根据 frontend 保存的 stored-index anchor 与 follow-tail 状态选择新 upper bound；anchor 已被删除或 frontend 跟随尾部时回退到新的 storage tail。
- 重载只沿 index cache 执行 `prevIndex/get`，直到填满当前 viewport、overscan 和有限 semantic boundary lookaround；不得扫描或投影完整 history。
- 新窗口完成 storage 读取和语义投影后一次性发布，避免先发布空窗口再逐项抖动；旧 generation 的异步结果一律丢弃。
- Entry key 使用 `(Agent address, window generation, semantic primary stored index/provider id)`；revert 后整窗 composition、render cache 和 expansion state 一起换代，不尝试复用旧 generation 的 item state。
- Frontend 将阅读位置另存为 stored-index anchor，而不是旧 generation 的 Compose key；重载后只恢复仍存在的 anchor，否则使用确定性的 tail fallback。
- AgentStorage 在 revert 时更新 stored-index cache 并整体清空对应 raw-value LRU；ViewModel 不得绕过或弱化这次完整缓存失效。
- Revert 后第一次窗口重载允许是 cold read，但读取与投影工作量必须受 semantic window item budget 限制；重载完成后该窗口自然重新填热 LRU。
- History条目操作使用真实stable storage index，并以`storageIndex + 1`作为exclusive boundary；执行前必须重新校验所选Session、Agent、stable target和idle turn job。
- `Revert here`只截断所选Agent的全部storage timeline suffix，并同步pending steer、自动标题one-shot gate及root Session catalog标题；`Fork here`把所选Agent的prefix复制成无descendants的新root Session，不修改source。

## Compose稳定性与缓存

- Application、Session 和 Agent ViewModel 引用必须具有稳定 identity；registry 对同一地址复用同一实例，Composable 不得在重组时重新包装 ViewModel。
- Revert 与 Compose stability 不冲突：immutable 指每次发布的 snapshot/content 不会就地变化，stable 指所有可变结果都通过可观察 state 发布；它们都不表示 timeline 永远不变。
- 只有真正满足 Compose stability contract 的类型才能标记 `@Stable` 或 `@Immutable`；history entry 使用递归 immutable snapshot，不把可变 collection 伪装成 immutable。
- 普通 `List`、`Set` 和 `Map` 不因放进 data class 就自动稳定；`HistoryWindowSnapshot` 变化时允许 list boundary 重组，但同 generation 的普通更新必须复用未变 entry 实例。
- Kotlin 2.4 Compose compiler 的 strong skipping 可以跳过参数未变的 restartable Composable 并记忆 lambda，但 unstable 参数按实例比较；不得每次 emission 重建等价 list、wrapper 或 callback 来抵消 skipping。
- Composable 边界至少拆到 application shell、Session surface、Agent surface、history list 和 history row；每层只接收该层需要的稳定参数，并在最靠近消费者的位置 collect state slice。
- History list 只 collect `HistoryWindowSnapshot`；每个 history row 是独立可跳过的 Composable，只接收 immutable entry、宽度和自己的展开 state。
- LazyColumn item 使用 generation-scoped stable key 和语义 `contentType`；同 generation 的 prepend、append 或邻项变化保留未变 entry identity，使已有 composition slot 可以移动、跳过或复用。
- Frontend 为每个 generation-scoped item key 持有独立展开 state，不把整份 expanded-id set 作为所有 row 的参数；generation 变化时整体丢弃旧展开 state。
- 纯展示派生值使用 `remember(entry, width, expanded)` 缓存；storage 读取、history projection、命令结果和生命周期状态不得放进 `remember` 充当业务缓存。
- 高频 scroll offset 不直接驱动整棵 history 重组；只对 near-boundary、follow-tail 等低频布尔结果使用 `derivedStateOf`，异步加载触发通过 `snapshotFlow` 加 `distinctUntilChanged` 观察。
- LazyColumn 的 item provider 与 key index 只在 window snapshot 变化时更新；hover、composer、activity 或无关 overlay 更新不得重新枚举 window。
- Subcomposition reuse budget 保持有界，并以 viewport、overscan、history row `contentType` 和滚动基准结果调优；不得把当前固定保留数量当成性能结论。
- 使用 Compose compiler stability/metrics report 和重组计数验证边界；不要用注解掩盖不满足契约的类型。

## Streaming与frontend衔接

- Committed history window、stream tail 和 activity 分开发布，renderer 在当前 Agent 表面合并显示。
- Stream item 提交进 storage 后，以稳定 identity 从 stream tail 去重，并只刷新受影响的 tail entries。
- LazyColumn 只消费当前 projected window；其 `layoutInfo.visibleItemsInfo`、viewport 与已加载边界共同构成 history demand signal。
- Frontend 使用 `snapshotFlow` 在进入前边界预取区时请求 Agent ViewModel 异步扩窗；同一 cursor/generation 只允许一个在途请求，切换 Agent、关闭 Session 或 generation 失效时取消旧请求。
- 初始请求按有限 item budget 读取；测量后如果 projected rows 尚未覆盖 viewport 与 overscan，则继续异步补充，不能在 composition 或 measure 中同步读取 storage。
- Prepend 后依靠 stable key 与 item 内行偏移保持原可见锚点；预取必须早于用户抵达已加载边界，正常滚动不显示阻塞式 loading gap。
- Follow-tail 和 unread 使用 tail append/revert 事件增量更新，不通过比较完整 history list 推导。
- Renderer 只格式化可见及 overscan entry；storage I/O、协议解码、语义投影、文本换行和 item composition 是独立步骤，前三者在 ViewModel coroutine 中完成并原子发布 window delta。
- 通用 LazyColumn 只发布布局信息并执行虚拟布局，不依赖 AgentStorage、history cursor、prefetch job 或 conversation 模型。

## 验证

- 验证 composer 每次编辑只发布 composer slice，不发布 history、execution、topology 或 catalog。
- 验证 stream delta 只发布 stream/execution相关 slice，不扫描或复制 committed history。
- 验证 global settings、overlay 和 Session selection 更新不会重建无关 Agent history window。
- 验证打开长 history 的 Agent 只沿 AgentStorage index cache 执行有限次 `prevIndex/get`，不复制 index list、不扫描完整 timeline，也不建立第二套 raw page cache。
- 验证 LazyColumn viewport 驱动提前预取和自适应首屏填充；composition、measure 和滚动输入路径不执行 storage I/O、协议解码或全量 semantic projection。
- 验证 prepend batch 后可见 stable key、item内偏移、follow-tail 和 unread 状态保持正确。
- 验证 tool call/output 跨 batch、并行完成、重复 call id 和 orphan output 的投影语义。
- 验证 append、stream commit、checkout/revert、Session fork 和 Agent ViewModel reopen 的 cursor/generation行为。
- 验证 composer edit、stream delta、hover 和 overlay 不重组未变 history row；append 只影响 tail，展开只影响目标 row。
- 验证未变 state slice、semantic entry、item callback 和 window segment 保持 identity，并通过 compiler report 确认关键 Composable 可跳过。
- 验证 revert 只提升一次 generation并重载当前有限 window，工作量与 window budget 成正比，不扫描完整 history，也不复用旧 generation 的 expansion/remember state。
- 验证 revert 对 raw-value LRU 执行完整失效，旧缓存值不会跨 generation 使用；首次窗口重载只读取有限项，随后重复访问命中重新填热的 LRU。
- 记录 revert 后 cold window reload 的 storage read、投影项数与耗时，以及 reload 后的 warm-cache hit。
- 验证 revert 后 surviving stored-index anchor 可恢复，已删除 anchor 确定性回退到新 tail。
- 记录超长 history 打开、连续向前滚动和流式追加时的 storage read、projection、item composition、recomposition 和 slot reuse 计数。

# Task Tree

- [done] 审批下游读写需求与 storage API
  - [done] 枚举生产读写调用点
  - [done] 还原各 consumer 的独立数据流
  - [done] 审批逐项读写需求
  - [done] 审批 storage API 调整意见
  - [done] 检查协议、迁移与测试阻塞点
  - [done] 将批准结论写入 implementation planning

# Details

## Scope

- 本文只盘点需求和评审 API。
- 本阶段不修改下游代码。
- AgentState、History 和 TurnHook 分别拥有自己的扫描与投影逻辑。
- Storage 只提供通用 timeline primitive。
- 禁止新增任何跨 timeline 的 `stableEventAt`、`historyEventAt`、stable merge、
  History projection 或 compaction projection helper。
- 现有
  `kanban/done/2026-08-28-design-indexed-history-timeline.md`
  中与上述边界冲突的 helper 设计不再有效，待本轮审批后统一修订。

## Approved Result

- 保留 `IndexVersioned.get(index)` 的 floor-visible value 语义。
- 在 `IndexVersioned` 接口增加：
  - `getExact(index: Int): T?`。
  - `indexesIn(range: IntRange): List<Int>`。
- `indexesIn` 只返回 detached ascending list；调用方需要反向时使用
  `asReversed()`。
- 不增加独立 exact-entry timeline 接口。
- 不增加通用 append transaction scope。
- `AgentStorage.activeMessageWindowAt(index)` 返回 active compaction point 与
  完整 events。
- Active message window 必须在一个 `buildList` 中先添加 derived prefix，再按
  index boundaries 分段添加 work/index events；禁止完整拼接后排序。
- `AgentStorage.activeTurnMarkerAt(index)` 统一读取 active turn marker。
- `turnId` 从 `KodexAgentSettings` 移入 `CleanTurnMarker`。
- 初始化写入：
  - `CleanCompactionPoint` at index `0`。
  - `CleanTurnMarker` at index `1`。
- WorkGroup 由前一个 index entry 锚定其后、下一个 index entry 之前的 work
  range。
- 同一个 index entry 可以同时锚定自身顶级 item 和后续 WorkGroup。
- TurnMarker 是 index entry；History 不再扫描 settings 推断 turn boundary。
- AgentState state reconstruction 分别查找 index/work 的最新 relevant
  candidate，再比较 candidate indexes；不构造 merged sequence。
- 普通 item、WorkGroup 与 TurnMarker 的 duration 都从投影时已知的边界 index
  通过固定次数 exact timestamp reads 得到。
- Compaction remote request 期间不持有阻塞 settings update 的 write lock。
- Compaction commit 使用 commit 时最新 settings，不能覆盖期间发生的 rename。
- Session repository fork API 只接收 `sourceEntryIndex`，执行完整 storage fork。
- History range fork 在完整 fork 后对 target storage 执行 revert。
- Filesystem 使用完整 raw file fork；in-memory 保留 backend-private full clone。
- 未单独填写意见的 R4、W3-W6、D4、D6-D8 保持现有业务语义，只做新协议适配。
- 允许的共享跨 timeline storage 语义仅有：
  - `activeMessageWindowAt`。
  - `activeTurnMarkerAt`。
- 继续禁止 stable/history exact-event helper、History merge/projection helper。

## Blocker Check

- 当前没有阻止实现开始的 hard blocker。
- Kodex submodule worktree 干净。
- Root worktree 只有本 task 文档改动。
- 下游当前仍因旧 stable/compaction API 不可编译，这是预期实施起点。
- 上游协议必须先再次修改并通过单独审批：
  - 新增 `CleanTurnMarker`。
  - 从 `KodexAgentSettings` 删除 `turnId`。
  - 初始化 latest index 从 `0` 变为 `1`。
  - 增加 `getExact`、`indexesIn`、active window 与 active turn helpers。
- Compaction 锁调整有现成反向测试：
  - `Kodex/agent-state/impl/src/commonTest/kotlin/io/github/stream29/kodex/agentstate/impl/KodexAgentStateImplTest.kt:757`
    当前断言 settings update 等待 compaction。
  - 实施时必须反转为 settings update 在 remote compaction 未完成时成功，并验证
    compaction commit 保留新 settings。
- 本地数据需要第二次离线迁移，且迁移仍由用户在代码完成后执行。
- 只读预检结果：
  - 174 个 root sessions 全部已是 index/work schema。
  - 174 个 session 的现有 index `1` 均已占用。
  - 必须把所有 timeline 的 index `> 0` 整体顺延一位，为初始 turn marker
    保留 index `1`。
  - 现有 settings 中有 1,246 次后续 turnId change。
  - 这些 change indexes 与现有 index entries 无碰撞，可在顺延后的同 state
    index 写入 turn marker。
  - 887 个 compaction points 与 6,409 个含 turnId settings entries 均可解析。
  - 预检没有 JSON parse error。
- Migration 必须：
  - 将 settings 中的 initial turnId 写入新 index `1` marker。
  - 将每次后续 turnId change 写入对应 `CleanTurnMarker`。
  - 删除或忽略 settings JSON 中的 legacy turnId。
  - 重建六条 timeline 的 latest pointers。
  - 重新验证 compaction point/output adjacency。
- Full fork 不需要新的通用 AgentStorage copy API：
  - Filesystem repository 可复用 raw timeline file copy 进行完整 fork。
  - In-memory repository 在 backend 内复制完整 timeline state。
  - Range semantics 只存在于 fork 后的 target revert。

## Read Cases

### R1. Snapshot state values

- 调用位置：
  - `Kodex/agent-state/impl/.../KodexAgentStateImpl.kt:165`
  - `Kodex/agent-state/impl/.../KodexAgentStateImpl.kt:257`
  - `Kodex/agent-state/impl/.../KodexAgentStateImpl.kt:721`
  - `Kodex/agent-state/context-window/.../ContextWindowTokenBudget.kt:20`
  - `Kodex/app/viewmodel/agent/.../AgentRuntimeViewModel.kt:387`
- 业务输入是已捕获的全局 snapshot index。
- `settings`、`tokenCount` 和 `unstable` 需要读取不大于 snapshot 的最新值。
- 这是 versioned-state 读取，不是 exact-entry 读取。
- `unstable` 在首个 change point 之前表示空集合，不能把任意旧 event 当作值。
- 该 case 不需要枚举 index/work，也不需要任何 history helper。

#### 审批区

- 状态：待审批
- 用户意见：
这个就是IndexVersioned.get的本意了。我们已经为此甚至添加了专门的IndexVersioned.latestValue()拓展。

### R2. AgentState active compaction point and model input

- 现有旧逻辑位于：
  - `Kodex/agent-state/impl/.../KodexAgentStateImpl.kt:172`
  - `Kodex/agent-state/impl/.../KodexAgentStateImpl.kt:641`
- 对 snapshot `n`，AgentState 自己执行以下流程：
  - 从 `index` 的 `(-∞, n]` 反向扫描 exact entries。
  - 解码到最新 `CleanCompactionPoint`，记其 index 为 `p`。
  - 从 `[0, p)` 的 index entries 中筛选 retained items。
  - 按现有 64K 规则派生 prefix，并以 `p` 为 key 缓存在 AgentState。
  - 分别读取 `(p, n]` 的 index/work indexes。
  - 按 state index 升序合并后读取全部 exact payload，构造模型输入。
- index metadata 只能定位候选 entry，不能区分 event 与 compaction point。
- point 的识别属于 AgentState 对 `CleanIndexEntry` 的解释，不下沉到 storage。
- 非初始 point `p` 的 output 位于 `work[p + 1]`。
- 该 case 需要完整 payload，不采用 History 的 opaque work 逻辑。

#### 审批区

- 状态：待审批
- 用户意见：

重点在于这个合并的过程，我们需要一个单一的buildList {}来确保不会写出粗暴拼接+sort的实现，而是先得到prefix再逐段拼接的过程。
这个应该写成AgentStorage.activeMessageWindowAt()函数放在agent-storage里。

### R3. AgentState durable state reconstruction

- 现有旧逻辑位于
  `Kodex/agent-state/impl/.../KodexAgentStateImpl.kt:662`。
- 读取顺序：
  - 先读取 snapshot 可见的完整 `unstable` 值。
  - 若没有 pending tool，则分别反向读取 index/work indexes。
  - AgentState 自己按 state index 合并两个反向序列。
  - 只为候选 entry 读取 exact payload，遇到第一个 state-relevant event 即停止。
- State-relevant：
  - user、agent、assistant messages。
  - index-side 或 work-side completed tools。
- State-irrelevant：
  - compaction point。
  - developer message。
  - reasoning。
  - context compaction output。
- 该扫描的停止规则与 History folding 完全不同。

#### 审批区

- 状态：待审批
- 用户意见：

这里其实不需要合并序列，因为我们都只关心序列的最新值。

### R4. TurnHook message scans

- 调用位置：
  - `Kodex/agent-runtime/decorator/turn-hook/.../TurnHookRuntime.kt:137`
  - `Kodex/agent-runtime/decorator/turn-hook/.../TurnHookRuntime.kt:153`
- `currentUserPromptTextOrNull` 只反向扫描 `index`：
  - 仅 trailing user/developer block 可继续。
  - 第一个其他 index entry，包括 compaction point，会终止 block。
- `latestAssistantMessageSince` 只反向扫描指定 snapshot 后的 `index`：
  - 非 assistant entries 被跳过。
  - 找到第一个 assistant message 后停止。
- TurnHook 不读取 work timeline，也不复用 AgentState 或 History cursor。
- 最常见读取只触及少量尾部 index entries，适合 bounded descending query。

#### 审批区

- 状态：待审批
- 用户意见：

### R5. History paging and fold projection

- 调用位置：
  - `Kodex/app/viewmodel/history/.../AgentHistoryViewModel.kt:393`
  - `Kodex/app/viewmodel/history/.../AgentHistoryViewModel.kt:529`
  - `Kodex/app/viewmodel/history/.../AgentHistoryViewModel.kt:571`
- History 自己维护 index/work paging cursor。
- 每一页分别获取有界 index metadata 和 work metadata，再按 state index 合并。
- index entry 必须读取 payload：
  - `StableIndexEvent` 形成独立 History item。
  - `CleanCompactionPoint` 不显示，并标记 `p + 1` 的 work entry 为
    context-compaction item。
- work entry 默认是可折叠 opaque descriptor。
- 除 point 后的 context-compaction output 外，fold grouping 不读取 work payload。
- 页边界不足时，History 自己用最后消费的 state index继续取下一批。
- Fold group、turn boundary 和 open suffix 都是前端规则，不进入 storage contract。

#### 审批区

- 状态：待审批
- 用户意见：

这个就是我们的重点了，需要在IndexVersioned上开个专门的口：IndexVersioned.indexesIn(IntRange): List<Int>。
AgentHistory里的每一个顶级ItemViewModel，都对应index timeline里的一项。只不过group这个ViewModel比较特殊，如果需要展开的话会跑到work timeline里加载对应的item。

### R6. History item payload and elapsed time

- 调用位置：
  - `Kodex/app/viewmodel/history/.../HistoryItemLoadContext.kt:82`
  - `Kodex/app/viewmodel/history/.../HistoryItemLoadContext.kt:87`
  - `Kodex/app/viewmodel/history/.../AgentHistoryViewModel.kt:555`
- History descriptor 必须记录 payload 所属 timeline。
- 独立 index item 或展开后的 work item按所属 timeline 读取 exact payload。
- Collapsed work group 不创建 payload owner，也不读取 payload。
- Elapsed 使用 History 自己合并后得到的前一个可见 stable event index。
- `timestamp` 必须读取 exact entry；缺少任一 exact timestamp 时 elapsed 为零。
- Compaction point 不参与前一个可见 stable event 的计算。

#### 审批区

- 状态：待审批
- 用户意见：

应该任何item（包括group/turn）的time都可以O(1)得出。

### R7. History turn markers

- 调用位置：
  - `Kodex/app/viewmodel/history/.../AgentHistoryViewModel.kt:864`
  - `Kodex/app/viewmodel/history/.../AgentHistoryViewModel.kt:950`
- History 按 settings change-point indexes 升序读取 exact settings entries。
- Turn end 是下一个 turn marker 之前的最后一个可见 stable event。
- Turn start timestamp 优先使用 marker 的 exact timestamp。
- 初始 marker 没有合适 timestamp 时，回退到首个可见 stable event timestamp。
- “可见 stable event”由 History 自己合并 index/work 并排除 point 得到。
- 该规则不进入 AgentState 或 storage。

#### 审批区

- 状态：待审批
- 用户意见：

TurnMarker疑似时候合并进index timeline。这样也符合上面说的AgentHistory里的每一个顶级ItemViewModel，都对应index timeline里的一项。

### R8. History revert target validation

- 调用位置：
  - `Kodex/app/viewmodel/agent/.../AgentRuntimeViewModel.kt:329`
  - `Kodex/app/viewmodel/agent/.../AgentRuntimeViewModel.kt:668`
- UI target 已由当前 History generation 证明是可见 stable item。
- 执行 revert 前仍需重新验证 target index：
  - exact work entry 存在，或
  - exact index entry 存在且是 `StableIndexEvent`，不是 point。
- 用户文本保留检查只扫描 target 之前的 index-side user messages。
- Revert 删除六条 timeline 的 `[untilExclusive, +∞)`。
- Target 是 context-compaction output 时会保留其前一个 point。
- Point 本身不可能成为 UI target。
- 验证逻辑留在 Agent ViewModel，不新增共享 exact stable-event helper。

#### 审批区

- 状态：待审批
- 用户意见：

这里的语义和行为应该不需要改变。

### R9. Session fork

- 调用位置：
  - `Kodex/app/viewmodel/session/.../SessionViewModels.kt:287`
  - `Kodex/app/viewmodel/session/.../SessionViewModels.kt:341`
  - `Kodex/agent-session/contract/.../KodexSession.kt:88`
- 实际业务只复制 `[0, untilExclusive)`。
- History fork target 的重新验证规则与 R8 相同，但由 Session ViewModel 自己执行。
- 完整 Session fork 使用 `latestIndex + 1` 作为 boundary。
- Fork 只发生在 runtime 非运行状态，因此不会截断正在写入的 point/output pair。
- Filesystem repository 应直接复制六条 timeline 的前缀文件。
- Repository 已拥有 source entry，不需要接收任意 `KodexAgentStorage` 和 `from`。
- In-memory repository 使用自己的 backend-private prefix copy。
- 不恢复通用逐 payload 反序列化 fork。

#### 审批区

- 状态：待审批
- 用户意见：

因为我们已经取消了基于内存实现的区间fork，所以现在fork唯一可用的实现就是基于文件系统的全量fork。所以区间fork的正确实现就是全量fork+revert。

### R10. Session catalog and lightweight current-value readers

- 调用位置：
  - `Kodex/agent-session/filesystem/.../FileSystemKodexSessionRepository.kt:346`
  - `Kodex/agent-session/in-memory/.../InMemoryKodexSessionRepository.kt:223`
  - `Kodex/agent-runtime/impl/.../KodexAgentTools.kt:43`
  - `Kodex/hook/tool-utils/.../ToolHookUtils.kt:86`
- Catalog 只读取各 timeline 自己的 latest exact settings/timestamp entry。
- Tool、hook、title 和 settings consumers 读取当前可见 settings value。
- 这些 case 不枚举 history，不受 index/work merge 影响。
- Filesystem catalog 的 pointer repair 继续是 backend-private 优化。

#### 审批区

- 状态：待审批
- 用户意见：

嗯，不受影响，所以无所谓。

### R11. Revert compensation

- 现有实现位于
  `Kodex/agent-storage/contract/.../IndexVersioning.kt:163`。
- Compensation 需要一次获得 `[untilExclusive, +∞)` 的 exact stored indexes。
- 随后读取这些 exact entries，执行 revert，并在失败时按升序恢复。
- 当前 Flow extension 会对每个 index 重复执行 floor/ceil lookup。
- 该 case 需要通用 bounded index snapshot，不需要任何 history 语义。

#### 审批区

- 状态：待审批
- 用户意见：

应该也不受影响。

## Write Cases

### W1. Single-index logical append

- 调用集中在
  `Kodex/agent-state/impl/.../KodexAgentStateImpl.kt:340-505`。
- 现有组合包括：
  - index event + timestamp。
  - work event + timestamp。
  - completed event + unstable snapshot + timestamp。
  - unstable snapshot + timestamp。
  - settings + timestamp。
  - token count + timestamp。
- Writer 已由 AgentState `writeMutex` 串行化。
- 每次 mutation 从捕获的 global latest 后分配新 state index。
- 失败时只需删除 mutation 开始 index 之后的新 suffix。

#### 审批区

- 状态：待审批
- 用户意见：

这个地方之前出过问题，compaction会导致改名被阻塞，需要细化锁粒度。

### W2. Injected history

- Contract 位于
  `Kodex/agent-state/contract/.../KodexAgentState.kt:216`。
- 生产调用仅来自 steer 和 turn hooks：
  - `Kodex/agent-runtime/decorator/steer/.../SteerRuntime.kt:40`
  - `Kodex/agent-runtime/decorator/turn-hook/.../TurnHookRuntime.kt:116`
- 实际输入均为 `StableIndexEvent.Steerable`。
- 每个输入占一个连续 state index，并写 exact timestamp。
- 整批失败时回退整段新 suffix。
- 生产代码没有注入 reasoning、work tool output 或 context-compaction 的需求。

#### 审批区

- 状态：待审批
- 用户意见：

这个应该也不受影响。

### W3. Completed tool

- Contract 位于
  `Kodex/agent-state/contract/.../KodexAgentState.kt:249`。
- `StableCleanEvent.CompletedTool` 跨 index/work 两类 timeline。
- AgentState 根据具体 subtype 在本地路由：
  - request_user_input 和 update_plan 写 index。
  - 其他 completed tools 写 work。
- 同一 state index 还写 remaining unstable snapshot 和 timestamp。
- Storage 不负责 completed-tool 分类。

#### 审批区

- 状态：待审批
- 用户意见：

### W4. Provider response events

- 调用位置：
  - `Kodex/agent-state/impl/.../KodexAgentStateImpl.kt:421`
  - `Kodex/agent-state/impl/.../KodexAgentStateImpl.kt:475`
- AgentState 本地将 provider item 映射到具体 index/work subtype。
- Messages 写 index。
- Reasoning、hosted tools 和普通 tool output 写 work。
- Pending calls 只写 unstable。
- Provider output 的转换和路由不进入 storage contract。

#### 审批区

- 状态：待审批
- 用户意见：

### W5. Compaction mutation

- 上游 helper 位于
  `Kodex/agent-storage/contract/.../CompactionStorage.kt:23`。
- AgentState 负责：
  - 读取 previous point。
  - 构造 provider input。
  - 生成 next window id 和 encrypted output。
- Storage helper 负责一个固定 invariant：
  - point 写在 `p`。
  - settings 与 synthetic token count `0` 写在 `p`。
  - exact timestamp 与 `StableContextCompaction` 写在 `p + 1`。
  - 返回 `p + 1`。
- History 和 AgentState 只在 runtime 发布 `p + 1` 后读取该 mutation。

#### 审批区

- 状态：待审批
- 用户意见：

### W6. Initialization and destructive revert

- 调用位置：
  - `Kodex/agent-storage/contract/.../AgentStorageInitialization.kt:15`
  - `Kodex/agent-storage/contract/.../KodexAgentStorage.kt:137`
- Initialization 在 index `0` 写 point、settings、timestamp 和 token count。
- Destructive revert 接收合法 UI/session boundary，删除六条 timeline suffix。
- 两者需要 operation-level compensation，但不需要 event merge。
- Point/output 边界合法性由产生 target 的 consumer 和 runtime 非运行状态保证。

#### 审批区

- 状态：待审批
- 用户意见：

## API Design Opinions

### D1. Separate exact-entry and versioned-state semantics

- 当前问题：
  - `IndexVersioned.get(index)` 总是返回 floor-visible value。
  - 对稀疏 index/work/timestamp timeline，调用缺失 index 会静默返回旧 entry。
  - 下游只能依赖“先拿 exact index，再调用 floor get”的隐含约束。
- 建议：
  - 新增通用 `IndexedTimeline<T>`。
  - `entry(index)` 只读取 exact stored entry，缺失时返回 `null`。
  - `IndexVersioned<T>` 扩展 `IndexedTimeline<T>`，额外保留 floor-visible value
    读取。
  - Mutable contract 同样分为 exact timeline 与 versioned timeline。
- Storage 属性建议：
  - exact：`index`、`work`、`timestamp`。
  - versioned state：`settings`、`tokenCount`、`unstable`。
- Backend 可以继续用一个实现类，同时通过 storage property 暴露更窄的 contract。
- 该设计不会新增任何跨 timeline event lookup。

#### 审批区

- 状态：待审批
- 用户意见：

我们加一个IndexVersioned<T>.getExact(index): T?在接口上，不需要新增接口。

### D2. Replace Flow extensions with one bounded member query

- 当前问题位于
  `Kodex/agent-storage/contract/.../IndexVersioning.kt:85`。
- `indexes` 和 `indexesDescending` 逐项调用 floor/ceil。
- `CachedIndexVersioned` 已在
  `Kodex/agent-session/filesystem/.../CachedAgentStorage.kt:121`
  缓存完整排序 indexes，却无法优化 extension 内部循环。
- 建议 contract：

```kotlin
suspend fun indexes(
    range: IntRange = 0..Int.MAX_VALUE,
    order: IndexOrder = IndexOrder.Ascending,
    limit: Int = Int.MAX_VALUE,
): List<Int>
```

- 返回值是调用时 detached、已排序的 index snapshot。
- `limit` 按 `order` 方向截取。
- Cached/in-memory 实现只需：
  - 一次 read lock。
  - 两次 binary search。
  - 复制命中的结果段。
- Direct filesystem 实现每次只扫描目录一次。
- History 使用小批量 descending query。
- AgentState model input 和 compensation 使用完整 bounded range。
- TurnHook 和 state reconstruction 使用 descending chunks，并以最后 index 继续。
- 不引入有状态 cursor，也不让实现持锁跨 consumer 代码。

#### 审批区

- 状态：待审批
- 用户意见：

我在上面已经给了更优的index形状。不需要limit，也不需要order（调用方自己asReverse一下就OK）。

### D3. Add one storage-level append compensation scope

- 当前问题：
  - 每个 AgentState mutation 手工嵌套两到三层 `setWithTransaction`。
  - index/work 分流后，批量 inject 还需要同时补偿三条 timeline。
- 建议增加通用 append-only primitive：
  - 进入 scope 时捕获 `firstIndex = storage.latestIndex() + 1`。
  - block 可在一个或多个后续 state indexes 写任意 timeline。
  - block 失败时对六条 timeline 执行 `revert(firstIndex)`。
  - caller 仍必须串行化 writer。
- 该 primitive 只提供失败补偿，不解释 event，不分配业务 payload。
- `appendCompaction`、initialization 和 AgentState mutations 可复用。
- Destructive revert 继续使用独立的 suffix compensation。

#### 审批区

- 状态：待审批
- 用户意见：

无所谓，需要手动嵌套的地方很少，不需要为此承担优化成本。

### D4. Narrow injected-history contract

- 将 `injectHistory` 从 `List<StableCleanEvent>` 收窄为
  `List<StableIndexEvent.Steerable>`。
- 将 steer provider 和 app pending-steer contract 同步改为
  `StableIndexEvent.Steerable`。
- 这样 injected history 永远只写 index，不需要通用 stable-event router。
- AgentState response 和 completed-tool 路由仍由各自业务代码处理。

#### 审批区

- 状态：待审批
- 用户意见：

### D5. Make Session fork repository-owned and prefix-only

- 将 repository API 从：

```kotlin
createFork(source: KodexAgentStorage, from: Int, until: Int)
```

- 收窄为：

```kotlin
createFork(sourceEntryIndex: Int, untilExclusive: Int)
```

- Repository 用自己的 backend 定位 source。
- Filesystem 始终走 raw prefix file copy。
- In-memory 使用 backend-private prefix copy。
- 删除 arbitrary-source fallback 和非零 `from`。
- AgentStorage contract 继续不暴露 fork。

#### 审批区

- 状态：待审批
- 用户意见：

上面说了，只提供全量fork接口（这也是文件系统的真实行为）。所以只需要sourceEntryIndex一个参数。

### D6. Keep all cross-timeline interpretation in each consumer

- Storage 只提供：
  - exact entry。
  - versioned value。
  - bounded index metadata。
  - append/revert primitive。
- AgentState 自己实现 point lookup、retained prefix、model merge 和 state scan。
- History 自己实现 visible-event merge、fold groups、turn boundaries 和 targets。
- TurnHook 自己实现 message stop conditions。
- 不新增：
  - `stableEventAt`。
  - `historyEventAt`。
  - merged stable iterator。
  - shared compaction projection。
  - shared History descriptor。

#### 审批区

- 状态：待审批
- 用户意见：

### D7. Do not add a persisted commit timeline in this refactor

- 当前多 timeline transaction 是 failure-compensated，不是 reader-atomic commit。
- AgentState 通过 `writeMutex` 串行化写入，并只在完整 mutation 后更新
  runtime `latestIndex`。
- History 以 runtime `latestIndex` 作为刷新 publication boundary。
- Fork/revert 只在 runtime 非运行状态执行。
- 本轮建议不新增 commit marker 或第七条 timeline。
- D3 的 append scope 不能宣称提供并发 reader atomicity。

#### 审批区

- 状态：待审批
- 用户意见：

### D8. Keep compaction pairing as one specialized write helper

- `appendCompaction` 是唯一保留在 storage contract 的业务形状。
- 原因是 point/output 必须连续跨两个 state indexes，并同步重置 token count。
- Active-point lookup 和 output projection仍不属于该 helper。
- Generic revert 不解码 point，也不执行 History target validation。

#### 审批区

- 状态：待审批
- 用户意见：

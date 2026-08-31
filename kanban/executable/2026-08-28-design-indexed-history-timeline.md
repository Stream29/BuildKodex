# Task Tree

- 实现 index/work history timelines
  - [done] 确定 index/work 持久化分区
  - [done] 重构 clean event 与 agent-storage 六条 timelines
  - [done] [审批下游读写需求与 API](../done/2026-08-30-review-downstream-storage-access.md)
  - 修订上游协议
    - 将 turnId 从 settings 移入 index marker
    - 增加 exact 与 range index primitives
    - 增加 active turn 与 message-window helpers
    - 更新 initialization 与 agent-storage tests
  - 更新 agent-storage implementations
    - 更新 in-memory index operations
    - 更新 direct filesystem index operations
    - 固定 range snapshot 与 exact-read contract
  - 重写 AgentState
    - 路由 index/work writes
    - 使用 active message window
    - 使用独立 latest-candidate state scan
    - 细化 compaction 锁粒度
    - 写入 turn markers
  - 迁移非 History consumers
    - 更新 turn、compaction 与 tool hooks
    - 更新 steer 与 completed-tool contracts
    - 更新 history target validation
  - 重构 Session storage ownership
    - 更新 cached/session timeline wrappers
    - 收窄 repository full-fork API
    - 实现 filesystem full raw fork
    - 实现 in-memory private full clone
    - 用 target revert 实现 history fork
  - 重构 History projection
    - 以 index entries 建立顶级 anchors
    - 以 work ranges 建立 lazy groups
    - 从 turn markers 建立 turn items
    - 以已知边界读取 duration
    - 保持 paging、invalidation 与 targets
  - 编写第二次本地迁移
    - 顺延所有 index 大于零的 records
    - 从 settings 生成 turn markers
    - 清理 legacy turnId 与 pointers
    - 增加 dry-run 和完整校验
  - 验证实现
    - 运行分层 compile 与 tests
    - 验证 compaction/settings 并发
    - 验证 History 不读取 collapsed work payload
    - 构建 CLI 供用户迁移

# Details

## Approval Source

- 下游读写 case、用户原始审批意见与 blocker preflight 保存在：
  - `kanban/done/2026-08-30-review-downstream-storage-access.md`
- 本文件只记录批准后的 implementation route。
  - 用户已授权执行；本任务现已进入 executable。

## Architecture Boundaries

- `index` 保存：
  - standalone stable index events。
  - compaction points。
  - turn markers。
- `work` 保存：
  - reasoning。
  - 普通 completed tools。
  - encrypted context-compaction output。
- AgentState 与 History 不共享 cursor、merge、descriptor 或 projection。
- Agent-storage 只增加两个经批准的领域读取：
  - `activeTurnMarkerAt`。
  - `activeMessageWindowAt`。
- 禁止增加：
  - `stableEventAt`。
  - `historyEventAt`。
  - merged stable iterator。
  - shared History projection。

## Turn Marker Protocol

- 新增 `CleanTurnMarker(turnId: String) : CleanIndexEntry`。
- 从 `KodexAgentSettings` 删除 `turnId`。
- Initialization 使用同一个 initial timestamp 写入：
  - index `0`：`CleanCompactionPoint`。
  - index `0`：settings、token count `0` 和 timestamp。
  - index `1`：`CleanTurnMarker` 和 exact timestamp。
- Initialized storage 的 global latest index 是 `1`。
- Empty Agent 已拥有 index `1` 的 initial turn marker，因此第一次
  `markNewTurn()` 保持 no-op。
- 后续 `markNewTurn()` 在一个新 state index 写：
  - fresh `CleanTurnMarker`。
  - exact timestamp。
- Settings update 不再复制或更新 turnId。
- `activeTurnMarkerAt(snapshot)`：
  - 获取 `index.indexesIn(0..snapshot)`。
  - 从尾部读取 exact entries。
  - 返回第一个 `CleanTurnMarker`。
  - 对缺少 marker 的 initialized snapshot 失败。
- Hook 和 request metadata 从 active turn marker 读取 turnId。

## IndexVersioned API

- 保留：

```kotlin
suspend operator fun get(index: Int): T
```

- `get` 继续返回不大于 index 的最新 visible value。
- 新增：

```kotlin
suspend fun getExact(index: Int): T?

suspend fun indexesIn(range: IntRange): List<Int>
```

- `getExact`：
  - 只读取 exact stored change point。
  - 缺失时返回 `null`。
  - 不回退到 floor value。
- `indexesIn`：
  - 使用 inclusive `IntRange`。
  - 返回 detached ascending list。
  - 空 range 返回 empty list。
  - 非空 range 的 indexes 必须非负。
  - 不接受 order 或 limit。
- 反向 consumer 使用 `indexesIn(range).asReversed()`。
- 删除逐项 floor/ceil 的 Flow `indexes` 与 `indexesDescending` 实现。
- `nextIndex`、`prevIndex`、floor 与 ceil 可继续作为 single-index primitives。
- Cached/in-memory implementation：
  - 一次 read lock。
  - binary-search range 两端。
  - 只复制命中区间。
- Direct filesystem implementation：
  - 每次 `indexesIn` 只扫描目录一次。
  - `getExact` 先验证 exact file，再解码。
- `revertWithTransaction` 使用 `indexesIn` 与 exact values 保存 suffix。

## Active Message Window

- 新增：

```kotlin
data class ActiveMessageWindow(
    val point: CleanCompactionPoint,
    val events: List<StableCleanEvent>,
)

suspend fun KodexAgentStorage.activeMessageWindowAt(
    index: Int,
): ActiveMessageWindow
```

- 对 snapshot `n`：
  - 读取 `index.indexesIn(0..n)`。
  - 从尾部 exact-read，找到最新 compaction point `p`。
  - 从 `[0, p)` 的 index events 派生 retained prefix。
  - 在一个 `buildList` 中先添加 prefix。
  - 从 point `p` 开始按后续 index boundaries 分段处理。
  - 每个 boundary 前先按升序添加其前一段 work exact events。
  - Boundary 是 `StableIndexEvent` 时再添加该 event。
  - Boundary 是 turn marker 时不加入模型事件。
  - 最后添加最后一个 boundary 后直到 `n` 的 work events。
- 不创建两个完整 event lists 后 concat/sort。
- 非初始 point 的首个 work entry必须是 exact `p + 1`
  `StableContextCompaction`。
- Derived prefix 继续使用现有 retained item 集合和 64K truncation。
- Retention algorithm 从 AgentState impl 移到 agent-storage contract 内部。
- AgentState 可按 point index 缓存 returned prefix/window，但 cache 不属于 storage
  contract。

## AgentState Reads

- Model request：
  - 捕获 snapshot index。
  - 读取 snapshot settings。
  - 读取 active turn marker。
  - 读取 active message window。
  - 使用 window point 生成 request-window identity。
  - 使用 window events 生成 Responses input。
- Compaction：
  - 使用同一 active message window。
  - 从 window events 计算下一次 retained prefix。
  - Provider output 成功后调用现有 specialized `appendCompaction`。
- Durable state：
  - 先读取 snapshot-visible unstable state。
  - 分别从 index/work 反向查找最新 state-relevant candidate。
  - Index scan 跳过 turn marker、point 和 developer message。
  - Work scan 跳过 reasoning 与 context compaction。
  - 比较两个 candidate indexes，只投影较新的 candidate。
  - 不构建 merged sequence。

## AgentState Writes

- User、assistant、developer 与 agent messages 写 index。
- Completed request-user-input 和 update-plan 写 index。
- Reasoning、普通 tools、patch 与 hosted tools 写 work。
- `StableContextCompaction` 只能由 compaction mutation 写 work。
- `injectHistory` 和 steer contracts 改用
  `List<StableIndexEvent.Steerable>`，业务语义不变。
- Completed-tool API 保留跨 timeline contract，由 AgentState 本地路由。
- 保留现有少量 timeline-level `setWithTransaction` nesting。
- 不增加 storage-level generic append scope。

## Compaction Locking

- 当前 `mutate` 在完整 remote compaction 期间持有 `writeMutex`，必须拆分。
- 新流程：
  - 短锁内验证 stable state、设置 `Compacting`、捕获 snapshot。
  - 释放锁后读取 bounded snapshot 并执行 remote request。
  - Commit 短锁内重新验证 in-flight ownership。
  - Commit 时读取最新 settings。
  - 用 snapshot point lineage、provider output 和最新 settings 写入 compaction。
  - Finally 短锁内从最新 storage 恢复 durable state。
- `updateSettings` 在 remote request 期间必须成功。
- Compaction 期间 settings-only append 不改变 snapshot message window。
- Compaction commit 不得用旧 settings 覆盖期间发生的 rename、model 或 cwd
  update。
- 同时仍拒绝第二个 response、compaction 或 history mutation。

## Runtime And Hook Consumers

- `toHookTurnContext` 不再从 settings 读取 turnId。
- Hook context projection接收：
  - snapshot settings。
  - active turn marker turnId。
- TurnHook message scans 只使用 index：
  - current user prompt 保留 trailing user/developer stop rule。
  - latest assistant scan 保留 snapshot lower bound。
- ToolHook、CompactionHook 和 request metadata 使用 active turn marker。
- History target validation保持现有行为：
  - exact work event，或 exact `StableIndexEvent`。
  - `CleanCompactionPoint` 与 `CleanTurnMarker` 不是 target。
- User-text retention scan 只读取 index-side user messages。

## Session And Fork

- Repository full-fork API：

```kotlin
suspend fun createFork(sourceEntryIndex: Int): Int
```

- 不接受 source storage、from 或 until。
- Filesystem repository：
  - 在同 repository 中定位 source directory。
  - 完整 raw-copy 六条 timeline。
  - 不反序列化 payload。
- In-memory repository：
  - 使用 backend-private full timeline clone。
  - 不恢复通用 range fork。
- 完整 Session fork：
  - createFork。
  - 打开 target。
  - 追加 `[fork]` title settings。
- History fork：
  - createFork。
  - 打开 target。
  - 对 target storage 执行 `revert(target.untilExclusive)`。
  - 再追加 `[fork]` title settings。
- Fork 失败必须删除 reserved target。

## History Projection

- History 只使用 index timeline 建立顶级 anchors。
- 对相邻 anchors `a` 与 `b`：
  - `a` 自身按 exact `CleanIndexEntry` 类型投影。
  - `work.indexesIn((a + 1)..<b)` 是 `a` 锚定的 sealed work range。
- Latest anchor 后的 work suffix 是 open range。
- Anchor projection：
  - `StableIndexEvent`：独立顶级 item。
  - `CleanTurnMarker`：turn boundary item。
  - Initial compaction point：不显示 compaction output。
  - Non-initial compaction point：使用 exact `work[p + 1]` 的
    context-compaction item。
- Non-initial point 锚定 work range 时，`p + 1` 不再进入普通 work group。
- Sealed work range：
  - 一个 work item保持独立 child item。
  - 两个及以上 work items形成 `WorkGroup`。
  - 超过现有 child 上限时按上限切分多个 groups。
- Open work suffix 保持现有逐项展示语义。
- WorkGroup 只保存 work indexes、边界和 expansion state。
- Collapsed WorkGroup 不调用 `work.getExact`。
- 展开时按保存的 work indexes exact-read payload 并 materialize child。
- 同一 anchor 可同时对应：
  - anchor 自身的顶级 item。
  - 后续 work group 或 open work children。
- Newest-first 顺序必须保持：
  - newer anchor item。
  - preceding anchored work range。
  - older anchor item。

## History Duration And Turns

- Projection 时为每个 item保存计算 duration 所需的边界 indexes。
- 普通 stable item：
  - previous visible stable index。
  - current stable index。
- WorkGroup：
  - oldest child 之前的 visible stable index。
  - newest child index。
- Expanded work child：
  - previous work/stable index。
  - current child index。
- Turn item：
  - turn marker timestamp。
  - 该 turn 最后一个 visible stable index。
- 每次 duration load 只执行固定次数 `timestamp.getExact`。
- 缺失 timestamp、负数或非有限 duration 保持现有 fallback。
- History 不再扫描 settings changes。
- Initial turn marker at index `1` 是第一个 turn start。
- Revert/invalidation 后重新建立 anchor descriptors，旧 generation 不发布结果。

## Local Migration

- 新增独立 `uv` Python 脚本，本轮实现不自动执行。
- 输入是已经迁移到 index/work schema 的本地 root sessions。
- 对所有旧 state indexes：
  - index `0` 保持不变。
  - 所有 timeline 的 index `> 0` 加一。
- 从 settings timeline 读取 turnId：
  - initial settings turnId 写入新 index `1` 的 `CleanTurnMarker`。
  - 后续 turnId 与前一个 settings value不同时，在该 settings state index 的新
    映射位置写入 `CleanTurnMarker`。
- 迁移前验证后续 marker index 不与现有 index entry 冲突。
- Settings payload 删除 legacy turnId；reader 不保留 legacy fallback。
- 重建六条 timeline 的 latest pointers。
- 验证：
  - 所有 timeline indexes 严格递增。
  - 初始 point at `0`、initial turn marker at `1`。
  - 每个 turnId change 恰有一个 marker。
  - 每个 non-initial compaction point 的 output 仍在 `p + 1`。
  - stable event 总数和顺序不变。
  - 第二次 dry-run 报告 already migrated。
- 迁移使用 sibling staging、完整校验、原子切换与失败回滚。
- 用户在新代码和 CLI 构建完成后手动执行迁移。

## Implementation Order And Gates

- Gate 1：只修改 clean models、OpenAI settings model 与 agent-storage contract。
- Gate 1 必须通过协议审批和对应 tests 后，才能继续下游。
- Gate 2：修改 in-memory/filesystem implementations 并通过 agent-storage tests。
- Gate 3：编写迁移脚本但不执行。
- Gate 4：修改 AgentState 与 runtime hooks。
- Gate 5：修改 session cache、repository 与 fork。
- Gate 6：最后修改 History。
- 任何 gate 发现协议需要变更时，先回到上游，不在下游增加兼容 workaround。

## Validation

- IndexVersioned：
  - exact hit/miss。
  - visible floor get 不变。
  - empty、full、partial `indexesIn`。
  - append/revert 后 cached index snapshot。
- Turn protocol：
  - point at `0`、marker at `1`。
  - turnId 不存在于 settings。
  - active marker lookup across ordinary index events。
- Active message window：
  - no compaction、one compaction、multiple compactions。
  - prefix first and segmented chronological events。
  - turn markers excluded。
  - no concat/sort implementation。
- AgentState：
  - independent latest index/work candidates。
  - tool completion across both timelines。
  - remote compaction does not block settings update。
  - commit preserves latest settings。
- Session：
  - filesystem full fork。
  - in-memory full clone。
  - history fork equals full fork plus revert。
  - failed fork removes target。
- History：
  - every top-level item has an index anchor。
  - collapsed group performs zero work payload reads。
  - expansion exact-reads only group children。
  - compaction output and turn markers are independent items。
  - ordinary/group/turn duration uses known boundaries。
  - paging、append、revert、fork target 与 generation semantics 不变。
- Migration：
  - synthetic fixtures。
  - copied real-session structure。
  - dry-run、apply、rollback 和 idempotence。
- Gradle：
  - 按 gate 运行最小 JVM compile/tests。
  - 最后运行受影响模块 checks。
  - 构建 LinuxX64 release CLI 供迁移使用。

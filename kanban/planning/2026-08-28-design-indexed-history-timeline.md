# Task Tree

- 实现 index/work history timelines
  - [done] 确定事件分区与持久化语义
  - 重构 clean event 类型
    - 建立 index entry 与 work event 类型
    - 将 stable subtypes 分配到唯一 timeline
    - 将 checkpoint 缩减为 compaction point
    - 更新 polymorphic serialization tests
  - 替换 AgentStorage timelines
    - 修改 contract 与全局 index helpers
    - 修改 in-memory storage
    - 修改 filesystem 与 cached storage
    - 收窄 prefix-only fork 和 revert validation
  - 重写 AgentState 投影
    - 路由 index/work mutations
    - 合并 model-visible stable events
    - 派生并缓存 retained prefix
    - 写入配对 compaction point/output
    - 重建 stable Agent state
  - 迁移非 History consumers
    - 修改 turn hooks 与 message scans
    - 修改 history action target validation
    - 清理 compaction timeline references
  - 重构 History folding
    - 建立 index/work timeline cursor
    - 延迟 collapsed work payload reads
    - 配对 compaction point/output items
    - 保持 turn、paging 与 invalidation 语义
  - 实现本地离线迁移
    - 扫描并验证旧 root sessions
    - 重编号并分流所有 timeline files
    - 原子切换、回滚和幂等复检
  - 验证新存储与性能
    - 运行模块级 tests 和 checks
    - 迁移本地 session 数据
    - 重跑长 History benchmark
    - 运行根项目 check

# Details

## Decisions

- 用 `index` 和 `work` 替代现有 `compaction` 与 `stable` timelines。
- `index` 保存完整的独立 History 事件：
  - user、assistant 与 developer messages。
  - completed `request_user_input`。
  - completed `update_plan`。
- `index` 同时保存不参与模型投影的 compaction points。
- `work` 保存其余 stable events：
  - reasoning。
  - patch 与普通 completed tools。
  - encrypted `ContextCompaction` output。
- Compaction point 只保存 window lineage：
  - `windowNumber`。
  - `firstWindowId`。
  - `previousWindowId`。
  - `windowId`。
- Compaction point 不保存 `prefix`、`historyBaseIndex` 或 provider output。
- 每个非初始 compaction point 的下一条 work event 必须是对应的
  `ContextCompaction`。
- Runtime 为新 compaction 连续分配 point 与 output indexes。
- 只支持历史前缀 `(-∞, n]` 的 fork、copy 与 revert 边界。
- 现有本地数据一次性离线迁移，不保留 legacy reader。

## Execution Preconditions

- 先完成当前 root-only/subagent-removal 任务，再开始本任务实现。
- 本设计以最终不存在 AgentMessage、multi-agent events 与非零 range fork 为前提。
- 实现前确认 Kodex worktree 中现有用户修改已完成或明确保留。
- 执行本地迁移前停止所有 Kodex 进程并确认没有有效 session lease。

## Clean Models

- 保留统一的 `StableCleanEvent` 根类型，供注入、状态投影和 OpenAI 投影使用。
- 新增 `StableIndexEvent`：
  - 同时属于 `StableCleanEvent` 与 index entry。
  - messages、`request_user_input` 和 `update_plan` 实现该类型。
- 新增 `StableWorkEvent`：
  - reasoning、普通 completed tools 和 `ContextCompaction` 实现该类型。
- 新增不属于 `StableCleanEvent` 的 `CleanCompactionPoint`。
- `index` 的值类型是 `StableIndexEvent | CleanCompactionPoint` 的 sealed union。
- `work` 的值类型是 `StableWorkEvent`。
- `RemoteCompactionV2RetainedItem` 只能由 `StableIndexEvent` 实现。
- `StableCleanEvent.CompletedTool` 继续作为 tool completion API 的横切类型。

## Storage Contract

- `KodexAgentStorage` 保留六条 timeline：
  - `index`。
  - `work`。
  - `settings`。
  - `timestamp`。
  - `tokenCount`。
  - `unstable`。
- 初始化在 `index[0]` 写入 window number `0` 的 compaction point。
- 初始 storage 没有 work event。
- Global latest、floor、ceil、revert 与 fork 均覆盖六条 timeline。
- 新增 exact history-event helper，统一判断一个 state index 是否属于：
  - `StableIndexEvent`。
  - `StableWorkEvent`。
- 新增 ordered merge helper，按 shared state index 合并 index events 与 work
  events；compaction points 不作为 stable events 返回。
- Active compaction point 通过 index timeline 反向查找。
- `CachedIndexVersioned` 已缓存完整有序 indexes；稀疏 point lookup 和 timeline
  merge 不触发重复目录扫描。

## Compaction

- 对 snapshot `n`，令 `p` 为不大于 `n` 的最新 compaction point index。
- 模型输入按以下顺序构造：
  - 从 `[0, p)` 的 index events 派生 retained prefix。
  - 合并 `(p, n]` 的 index events 与 work events。
- `ContextCompaction` 是 point 后的首个 work event，因此自然位于 active window
  开头。
- Retained prefix 从 user messages、`request_user_input` 和 `update_plan`
  中按现有 64K token budget 派生。
- Derived prefix 可按 compaction point index 缓存在 AgentState 中，但不是规范
  持久化状态。
- Retention 类型与截断规则属于 storage schema 语义；以后修改时必须迁移数据。
- Remote compaction 成功后执行一个补偿式 mutation：
  - 在 `pointIndex` 写入新 compaction point、settings 与 synthetic token count
    `0`。
  - 在后续 `outputIndex` 写入 `ContextCompaction` work event 与 timestamp。
  - 只在完整 mutation 成功后发布 `outputIndex` 为 latest。
- Runtime 和 storage validation 必须拒绝没有配对 output 的非初始 point。

## History Projection

- 所有 index events 都是独立 History items。
- Compaction point 自身不显示；它与下一条 `ContextCompaction` work event
  共同投影成一个独立 compaction item。
- 其余 work indexes 在 index entries 与 turn boundaries 之间形成 fold groups。
- Collapsed group 只保留 work indexes、item count 与 elapsed，不读取 work
  payload。
- Group 展开时才读取 payload，并分类为 reasoning、patch 或普通 tool。
- 最新尚未由 index entry 或 turn boundary 封口的 work suffix 保持展开。
- Group 保留显式最大 child 数，避免一次展开无界 work range。
- History elapsed、turn end、fork target 与 revert target 使用合并后的 stable
  event indexes，不把 compaction point 当作可选 History target。

## Mutation Routing

- User message 与 injected developer message 写入 `index`。
- Assistant message 写入 `index`。
- Completed `request_user_input` 与 `update_plan` 写入 `index`。
- Reasoning、普通 tool output、patch 与 hosted tool result 写入 `work`。
- `ContextCompaction` 只能由 compaction mutation 写入 `work`。
- Tool completion 继续与 unstable snapshot、timestamp 做补偿式组合写入。
- `stateAt` 在 index events 与 work events 上反向合并：
  - 跳过 compaction points、developer messages、reasoning 与
    `ContextCompaction`。
  - messages 和 completed tools 保持现有状态语义。

## Fork And Revert

- Fork 只复制 `[0, untilExclusive)`。
- 所有 retained index events 都随前缀复制，因此无需 materialized prefix。
- Compaction point 不含 state index 字段，filesystem fork 可原样复制。
- Fork 与 revert 边界不得停在 compaction point 和其 output 之间。
- UI 只暴露 stable event targets；compaction point 不是 target，因此正常交互
  不会产生非法边界。

## Local Migration

- 迁移只处理本机 root sessions，并要求 Kodex 进程已停止。
- 现有 remote checkpoint 与 `ContextCompaction` 位于同一旧 index。
- 每个 remote compaction 插入一个新 state index：
  - checkpoint 映射为新的 index compaction point。
  - 原 `ContextCompaction` 映射为其后的 work event。
  - 后续所有 timeline indexes 顺延一位。
- 对旧 index `x`，普通记录的新 index 为：
  - `x + count(remoteCompactionIndex < x)`。
- 对旧 remote compaction index `c`：
  - point 使用普通映射结果。
  - output 使用 `point + 1`。
- 旧 stable events 按 discriminator 分流到 `index` 或 `work`。
- 旧 checkpoint 丢弃 `prefix` 与 `historyBaseIndex`，保留 window lineage。
- Settings、token count、timestamp 与 unstable records 使用相同 index 映射。
- Compaction output 必须具有 exact timestamp；迁移时从旧 compaction index
  复制。
- 迁移使用 sibling staging directories、完整预检、原子目录替换和失败回滚。
- 迁移后重建所有 `latest.json`，并验证：
  - stable event 总数不变。
  - 每个非初始 point 与下一条 work event 正确配对。
  - 所有 timeline indexes 严格递增且无冲突。
  - global latest 等于旧 latest 加已插入的 remote point 数。
  - 第二次 dry-run 不产生修改。

## Implementation Boundaries

- Clean-model phase：
  - 建立 index entry、index event、work event 与 compaction point 类型。
  - 删除 persisted compaction prefix 模型。
  - 用 serializer tests 固定每个 subtype 的唯一分区。
- Storage phase：
  - 将 contract、in-memory、filesystem 与 cached storage 改为六条新 timeline。
  - 收窄 fork API 为 prefix-only。
  - 实现 stable-event merge、point lookup 与 point/output validation。
- Agent-state phase：
  - 路由所有 stable writes。
  - 重建 model input、state projection 与 remote compaction mutation。
  - 缓存 active point 与 derived retained prefix。
- Consumer phase：
  - 将 hooks、history actions 和其他 stable scans 切换到准确 timeline/helper。
- History phase：
  - 从 index/work indexes 构造 descriptors 和 groups。
  - 保留最多 128 个 children 的 group 上限。
  - 延迟 work payload 分类到独立 item 显示或 group expansion。
- Migration phase：
  - 新增一次性 `uv` Python 脚本。
  - 先 dry-run 本地全部 root sessions，再执行、复检并删除 staging。

## Primary File Scope

- Clean models：
  - `Kodex/agent-storage/clean-models/.../stable/`。
  - `Kodex/agent-storage/clean-models/.../CleanCompactionCheckpoint.kt`。
- Storage contract and backends：
  - `Kodex/agent-storage/contract/.../KodexAgentStorage.kt`。
  - `Kodex/agent-storage/contract/.../AgentStorageInitialization.kt`。
  - `Kodex/agent-storage/contract/.../CompactionStorage.kt`。
  - `Kodex/agent-storage/in-memory/`。
  - `Kodex/agent-storage/filesystem/`。
  - `Kodex/agent-session/filesystem/.../CachedAgentStorage.kt`。
  - prefix fork implementations under `Kodex/agent-session/`。
- Agent state and consumers：
  - `Kodex/agent-state/impl/.../KodexAgentStateImpl.kt`。
  - `Kodex/agent-state/impl/.../RemoteCompactionV2.kt`。
  - stable-history scans under `Kodex/agent-runtime/` and `Kodex/app/viewmodel/`。
- History：
  - `Kodex/app/viewmodel/history/.../AgentHistoryViewModel.kt`。
  - `Kodex/app/viewmodel/history/.../HistoryItemLoadContext.kt`。
  - `Kodex/app/viewmodel/history/.../WorkGroupHistoryItemViewModel.kt`。
- Migration：
  - `Kodex/scripts/migrate-index-work-timelines.py`。
  - 删除或明确废弃只适用于旧格式的 compaction migration script。

## Validation

- Clean models：
  - 每个 stable subtype 只能属于 index 或 work。
  - Compaction point 不能参与 OpenAI history projection。
- Storage：
  - initialization、set、latest、floor、ceil、revert 与 prefix fork。
  - point/output pairing 和非法截断边界。
  - filesystem reopen 与 cached index maintenance。
- Agent state：
  - 无 compaction、一次 compaction 与多次 compaction 的 model input。
  - retained prefix 顺序、64K truncation 和 reopen 后重建。
  - compaction mutation 的 point/output indexes、window lineage 与 token reset。
  - index/work 混合历史的 state reconstruction。
- History：
  - fold group 不读取 collapsed work payload。
  - index events、turn boundaries 与 compaction output 正确切组。
  - incremental append、pagination、revert invalidation 和 fork targets。
- Migration：
  - dry-run、成功迁移、故障回滚与幂等。
  - 使用真实本地 session 副本验证数量、顺序和模型投影。
- Performance：
  - 重跑 13,388-event session 的启动、完整遍历、userspace reads、CPU 与 RSS。
- Gradle：
  - 复用当前 daemon JVM。
  - 先运行受影响模块 checks，再运行根项目 `check`。

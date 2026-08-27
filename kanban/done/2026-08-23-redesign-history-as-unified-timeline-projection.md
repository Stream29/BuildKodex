# Task Tree

- [done] 将 History 重构为统一 timeline 增量投影
  - [done] 确认 folding 与 turn time marker 是正交投影
  - [done] 确认 StableItem 与 TurnMarker 的统一 timeline 语义
  - [done] 定义虚拟 timeline merge cursor 与分页边界
  - [done] 将静态 turn duration 统一为专用 item view model
  - [done] 实现 folding 与 turn time 两个独立线性 projector
  - [done] 将 HistoryItemWindow 迁移到统一投影结果
  - [done] 移除 marker change 触发的非失效全量 reload
  - [done] 保留 revert 的 generation 全量失效语义
  - [done] 补充投影、identity、分页和运行态回归测试
  - [done] 构建 release binary 并执行真实 session 端到端验证
  - [done] 与用户确认计划后再进入实现

# Details

- 状态：`done`。实现、回归测试、release 构建和真实历史交互验证均已完成。
- 当前 History 将 stable projection、历史 `TurnFooter`、最新 `HistoryTurnFooterState` 和
  active duration 分开编排。新 turn marker 单独到达时，
  `Kodex/app/viewmodel/history/src/commonMain/kotlin/io/github/stream29/kodex/cli/history/AgentHistoryViewModel.kt:325`
  会进入 `reload(invalidate = false)`；该 reload 仍会发布空 window、进入
  `Initializing` 并重建全部已加载 item：同文件 `:280`。
- 已使用真实 session `130` 在内存中复现。旧 turn 的最后一个 stable item 是
  `AssistantMessage(170)`，新 marker 是 `172`，新 user message 是 `173`。marker-only
  refresh 曾依次发布原 window、空 window 和重建 window，且原有顶层 item 实例保留数为
  `0`。

- 新的持久化 source timeline 是两个稀疏 timeline 的虚拟有序合并：
  - `StableItem(index)` 来自 stable timeline。
  - `TurnMarker(index)` 来自 settings timeline 中的 `turnId` change point；普通 settings
    change point 不是 marker。
  - 两者共享 storage index，按 newest-first 排序；同一 index 冲突时，
    `StableItem(index)` 排在 `TurnMarker(index)` 前。
  - timestamp timeline 不产生 source entry，只在投影 duration 时按 index join。
  - pending tools 和 streaming item 继续作为两个独立 tail projection，不进入该持久化
    timeline。
- 虚拟 merge 不复制完整 timeline。实现维护 stable full index 与 turn-marker index 的
  cursor，每次选择下一个较新的 source entry；正常 append 只推进 newest edge，older
  paging 只推进 older edge。
- older cutoff 不得留下无法定位的半个结构：
  - foldable run 继续沿用有界 edge extension。
  - cutoff 遇到 TurnMarker 时，延伸到或记录其前一 turn 的结束 stable item；在该结束 item
    尚未 materialize 时不得产生悬空 time marker。
  - 连续空 turn 只扫描 marker/index 元数据，不读取 stable payload。

- 同一次线性扫描维护两个互不控制的 projection state：
  - Folding projector 只关心 foldable stable run 是否仍是最后一个 stable foldable run，
    不读取 `runningTurn`。
  - Turn-time projector 只关心 TurnMarker、timestamp、timeline end 与
    `runningTurn`，不读取或修改 folding state。
  - Turn-time projector 产生的 item 不得反馈给 Folding projector 充当 breaker。
- Folding projector 保留当前已确认语义：
  - `Reasoning`、普通 `Tool` 和 `Patch` 可折叠。
  - `Message`、`RequestUserInput`、`PlanUpdate` 和 `ContextCompaction` 正常显示并打断
    stable foldable run。
  - 只有多于一个 child 才产生 `WorkGroup`。
  - 最新 foldable run 保持 open；它是否 open 与 active/history 状态无关。
  - TurnMarker-only 更新只产生或迁移 `TurnTimeMarker`，不得封口或重建 newest open run。
  - 新 StableItem 到来后，前一 open run 才可能不再是最后一个 stable foldable run；此时按
    正常 stable append 语义封口。
  - 新 StableItem 与前一 open run 分属不同 TurnMarker 时不得跨 turn 合并；该检查仍由
    stable append 触发，不允许 TurnMarker-only 更新提前改变 folding。
  - running 状态切换产生或移除的静态 time marker 不得封口该 run。
  - 封口时复用全部 child 实例，只允许新建最新 `WorkGroup` wrapper；任何既有 sealed
    suffix 及其 item 实例必须保持不变。

- 每个真实 `TurnMarker B` 投影前一个 turn 的静态 time marker：
  - 前一 marker 为 `A`。
  - `endIndex = stable.prevIndex(B.index)`，且必须满足 `endIndex > A.index`。
  - `duration = timestamp[endIndex] - timestamp[A.index]`。
  - 旧 session 的首个 marker 缺少精确 timestamp 时，继续以该 turn 的首个 stable item
    timestamp 作为兼容 fallback；其他缺失或非法 timestamp 不产生 time marker。
  - 第一个 marker 和空 turn 不产生 time marker。
- 没有后继 marker 的最新 turn 由 Turn-time projector 在 timeline end 处理：
  - `runningTurn == null` 时，以最新 stable item 为结束点，产生同一种静态 time marker。
  - `runningTurn != null` 时不产生静态 history item；live duration 继续在 composer
    分割线左侧显示，并维持现有秒级、turn-job child ticker 语义。
  - running 状态切换只能新增或移除这一个最新 time marker，不能改变任何 stable-derived
    item、WorkGroup 或 folding state。
- 将静态 duration 统一建模为
  `HistoryItemViewModel.TurnTimeMarker(turnMarkerIndex, endIndex, duration)`：
  - 替换现有 `HistoryItemViewModel.TurnFooter` 与 `HistoryTurnFooterState` 双重表示。
  - historical 和 latest non-running turn 使用同一个 item VM variant。
  - placement 由 source timeline 的投影位置持有，不在 VM 中保留 `positionIndex`。
  - item 不执行 stable `read`，不参与 item elapsed、WorkGroup child、context action 或
    revert target。
  - timeline-end 投影转为后继真实 TurnMarker 投影时，按
    `(turnMarkerIndex, endIndex)` 复用同一个 item VM 实例。
- `AgentHistoryViewModel` 对外只发布统一的 indexed history item window；移除单独的
  `historyTurnFooter` flow。`activeTurnDuration`、pending tools 和 streaming item 保持独立。
  `committedItems` 应重命名为 `historyItems`，避免继续暗示所有一级 item 都直接来自 stable
  storage。

- 正常 source append 必须只重投影 changed newest prefix，并将结果与原
  `HistorySequence` persistent suffix 拼接：
  - stable append 只处理新增 stable items 与既有 newest open run。
  - marker append 只产生或迁移前一 turn 的 `TurnTimeMarker`，不得改变 newest open run。
  - running 状态切换只更新最新 `TurnTimeMarker` 与 live duration。
  - 每个发布窗口必须是完整、可渲染且永久可索引的快照；不得发布作为实现中间态的空
    window。
  - 正常 append 和 marker handoff 保持 generation 不变。
- 只有 latest index 后退、revert 或无法增量描述的 storage replacement 才允许：
  - 增加 generation。
  - reset 两个 projector、marker index 与 paging cursor。
  - invalidate all 并重建 window。
- `markNewTurn()`、user-message append 和 runtime job 启动是相邻但独立的状态变化。
  “运行中”严格以 `runtime.runningTurn != null` 为准，不额外引入 admission 或
  runtime-operation 状态。实现在 user message 已写入但 runningTurn 尚未登记的间隙可以
  短暂产生最新静态 time marker，但该状态只能影响这一个 item，不得传播为 stable window
  reload 或 folding 变化。

- contract 与实现迁移至少涉及：
  - `Kodex/app/contract/history/src/commonMain/kotlin/io/github/stream29/kodex/app/history/contract/AgentHistoryViewModel.kt`
  - `Kodex/app/viewmodel/history/src/commonMain/kotlin/io/github/stream29/kodex/cli/history/AgentHistoryViewModel.kt`
  - `Kodex/app/view/history/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/history/AgentHistoryView.kt`
  - 对应 common、JVM、Native 和 Mosaic tests。
- 回归测试必须分别证明两个 projector 正交：
  - 在最新端放置多项 foldable run，切换 running、resume、cancel 后，stable item 实例、
    open run 形态和 generation 完全不变。
  - 追加真实 TurnMarker 后，前一 turn 产生一个专用 `TurnTimeMarker`；sealed suffix
    identity 保持不变。
  - timeline-end latest marker 在 running 时消失、停止后出现，并在后继 marker 到来时复用
    同一个 VM。
  - first marker、多个 turn、空 turn、sparse index、timestamp 缺失或倒退均有明确结果。
  - 首个 marker 缺少 timestamp 时使用首个 stable timestamp fallback；其他非法 timestamp
    不产生 time marker。
  - user message append 到 `runtime.runningTurn` 登记之间允许短暂静态 time marker，但
    stable item、open run、sealed suffix 和 generation 保持不变。
  - marker-only refresh 从不发布空 window 或 `Initializing`。
  - marker、user message、running job 连续交接期间，每个窗口都可安全完成 LazyColumn
    in-flight measure。
  - marker 位于 page cutoff、超长 foldable run、连续 older paging 和 1024-entry storage
    LRU 下不重复解析已投影 stable payload。
  - revert 增加 generation，旧窗口继续可索引，当前窗口不保留旧 generation target。
- 端到端验证使用包含长 turn 的真实 session 副本，至少覆盖：
  - history turn 停止与 resume 时只有 time marker/live duration 改变。
  - 输入下一条 user message 时无整屏闪动。
  - 最新 open foldable run 不因 running 状态变化而折叠。
  - 已加载大历史不因 marker handoff 被丢弃或重新读取。

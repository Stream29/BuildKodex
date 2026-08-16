# Task Tree

- 修复既有 History 的大历史扩展性
  - 建立保持现有展示语义的正确性与性能基线
    - 覆盖有限尾部加载、连续 append、向旧历史翻页、revert 和多 frontend
    - 记录 storage read、snapshot size、key 枚举、对象保留和 Native GC
  - 确定真正有限的 window contract
    - 确定每 frontend data window 与共享 source 的所有权
    - 确定 older/newer cursor、锚点恢复和 opposite-edge eviction
  - 确定现有 event-row 投影的增量更新边界
    - 复用 storage index/value cache，不新增 raw cache
    - 合并 refresh、后台读取解码投影和原子结果安装
    - 消除随完整已浏览历史增长的 list copy、validation 和 key provider 成本
  - 制定保持现有交互和 context action 的迁移计划
  - 确定性能回归测试与验收阈值
  - 与用户确认计划后再进入实现

# Details

- 状态：`discussion`。本任务只修复既有行为，不增加自动折叠语义。
- 保留每个 stable event 独立成行、现有 disclosure state、焦点恢复和 context action。
- 当前实现首次只加载 64 个事件，但 append 和 older paging 会持续把新列表拼接到已有列表，不能维持有限窗口：`Kodex/app/viewmodel/history/src/commonMain/kotlin/io/github/stream29/kodex/cli/history/AgentHistoryViewModel.kt:122`、`:157`。
- 当前 `Newer` 请求没有实现，无法在向旧历史翻页并回收另一端后返回较新区域：`Kodex/app/viewmodel/history/src/commonMain/kotlin/io/github/stream29/kodex/cli/history/AgentHistoryViewModel.kt:137`。
- 当前 snapshot 构造会扫描完整 entries 列表并重建派生集合；成本随已浏览历史持续增长：`Kodex/app/contract/history/src/commonMain/kotlin/io/github/stream29/kodex/app/history/contract/AgentHistoryViewModel.kt:123`。
- 当前 `LazyColumn` key provider 会为完整输入列表建立 key list 和 index map，不能抵消无界 ViewModel snapshot：`Kodex/app/view/components/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/components/LazyListScope.kt:126`。
- 当前 entry 直接持有 `StableCleanEvent`，已加载窗口会继续保留 raw payload；窗口必须按语义条目数和 payload 体积共同设限：`Kodex/app/contract/history/src/commonMain/kotlin/io/github/stream29/kodex/app/history/contract/AgentHistoryViewModel.kt:52`。
- 当前一个 Agent 只有一个共享 window，但 history viewport 属于 frontend-local state；无界增长暂时掩盖了多个 frontend 停留在不同历史位置时的所有权冲突。
- 当前命令入口使用 `Channel.UNLIMITED`，重复 refresh 和 paging demand 只在出队后判定；storage value 的 JSON 解码和 entry 投影也没有独立的后台 dispatcher 边界：`Kodex/app/viewmodel/history/src/commonMain/kotlin/io/github/stream29/kodex/cli/history/AgentHistoryViewModel.kt:31`。
- 现有测试只覆盖稀疏初始加载和 revert 基础行为，尚未覆盖连续 append、双向 paging、多 frontend 或大历史上界：`Kodex/app/viewmodel/history/src/commonTest/kotlin/io/github/stream29/kodex/cli/history/AgentHistoryModelsTest.kt:21`。
- 该修复必须形成 group-neutral 基础设施，避免后续 `WorkGroup` 再次替换窗口、cursor 和缓存机制。
- [大型 MCP payload 冻结](2026-08-10-prevent-large-mcp-results-from-freezing-cli.md)是并列的既有行为问题，不在本任务内合并处理。
- 本任务不构成进入规划或实现的授权。

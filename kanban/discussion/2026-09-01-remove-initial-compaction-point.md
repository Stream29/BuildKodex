# Task Tree

- 移除 initial Compaction point
  - 停止新 storage 写入初始 marker
  - 保留 index 0 的其他初始状态
  - 保持旧 storage 可读
  - 更新初始化与投影测试

# Details

- 当前状态：讨论 storage 初始化中的历史遗留 sentinel。
- 本阶段不修改实现。

## 已确定需求

- 新 storage 不再向 index timeline 的 index 0 写入 `CleanCompactionPoint`。
- Settings、timestamp 和 token count 继续在 index 0 初始化。
- 第一个真实消息或工具事件继续从下一个全局 storage index 开始。
- 真正发生 context compaction 时仍写入 `CleanCompactionPoint`。
- 不迁移或改写已有 storage。
- 读取旧 storage 时继续兼容 index 0 的 initial marker。
- 用户界面忽略旧 storage 中 index 0 的 initial marker。

## 当前实现边界

- `MutableKodexAgentStorage.initialize()` 当前显式写入 initial marker，见
  `Kodex/agent-storage/contract-ext/src/commonMain/kotlin/io/github/stream29/kodex/agentstorage/contract/ext/AgentStorageInitialization.kt:18`。
- In-memory storage 当前也预置 initial marker。
- `activeMessageWindowAt()` 已支持找不到 compaction point 的情况，不需要 initial marker
  作为投影起点。
- `latestIndex()` 从所有 timeline 取最大值，因此 index 0 的 settings、timestamp 和
  token count 足以维持初始化后的 snapshot index。

## 实现范围

- 删除持久化和 in-memory 初始化路径中的 initial marker 写入。
- 更新依赖 index 0 marker 的单元测试与集成测试。
- 增加没有任何 compaction point 时的消息窗口投影覆盖。
- 保留读取旧 initial marker 的兼容分支。
- 不改变真实 compaction marker 与其 `index + 1` output 的布局。

## 讨论产出

- 确认受影响的初始化、投影和测试范围。
- 讨论完成后移入 planning，再形成实现与验证步骤。

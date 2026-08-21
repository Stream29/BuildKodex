# Task Tree

- 在 History 中显示 turn duration
  - 识别 turn marker 与 stable 边界
  - 设计 completed 与 running footer 编排
  - 按需计算并缓存 completed duration
  - 在 turn job 下维护秒级 ticker
  - 渲染 Worked for 虚拟分隔行
  - 覆盖 revert、边界与 job 生命周期测试
  - 验证大历史性能与 release 行为

# Details

- 状态：`planning`。
- 现有 `HistoryItemViewModel` elapsed duration 保持不变；turn duration 是额外的一级展示。
- `markNewTurn` 会在 settings timeline 写入新的 `turnId`，并在同一 index 写入 timestamp。
  turn marker 应以 `turnId` change point 识别，不能把普通 settings change point 当成新 turn。
- 已结束 turn 的范围由当前 marker 和下一个 marker 确定。结束点是下一个 marker
  之前的最后一个 stable history item，duration 是结束点 timestamp 减去当前 marker timestamp。
- 在结束点后插入只存在于 History 展示层的虚拟 item；不写入 storage，也不进入 model history。
- 虚拟 item 显示为 `---Worked for <duration>---`。
- 运行中的最新 turn 使用当前时间减去最新 marker timestamp。每秒刷新一次，ticker
  必须是 turn job 的 child job，并随 turn job 结束或取消。
- completed footer 参与历史线性窗口；running footer 始终位于当前 turn 的可见末尾。
- 边界与 duration 只通过 index、settings 和 timestamp 元数据计算，不为此解析 stable payload。
- completed duration 一旦边界稳定即可缓存；每秒刷新只作用于一个运行中的最新 turn。
- duration 展示复用现有规则：舍入到最近的毫秒后使用 Kotlin `Duration.toString()`。
- 测试覆盖首个 turn、多个 turn、空区间、timestamp 缺失或倒退、折叠区间、revert、
  turn 完成、取消和 ViewModel 关闭。

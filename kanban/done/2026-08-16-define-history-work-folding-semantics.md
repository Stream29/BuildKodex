# Task Tree

- [done] 确定消息间工作记录的自动折叠语义
  - [done] 确定 message boundary 和 foldable event 集合
  - [done] 确定 plan update、context compaction、request-user-input 等例外
  - [done] 确定单项 group、首尾开放 group 和跨 batch 部分 group 的展示
  - [done] 确定折叠 summary 的有界内容与 incomplete 表达
  - [done] 确定展开后的 child 虚拟化、分页和 context action
  - [done] 确定 pending/streaming work 与 committed group 的衔接
  - [done] 确定 frontend-local expansion、焦点、键鼠和 generation 失效语义
  - [done] 汇总并确认完整行为

# Details

- 状态：`done`。最终语义已经用于
  [`History WorkGroup` 实现](../done/2026-08-18-implement-history-work-groups.md)并通过端到端验证。
- maximal foldable run 只包含 `Reasoning`、普通 `Tool` 和 `Patch`，且至少两个 item 才折叠。
- `Message`、`PlanUpdate`、`ContextCompaction` 和 completed `request-user-input` 是 breaker。
- 最新未闭合 run 保持逐项展示；旧端跨 batch 查找 breaker 时使用有界延伸，达到上限可强制切分。
- summary 固定为 `Take n actions`，不读取 payload；展开后复用 child renderer、状态与精确 context action。
- pending、streaming 与 committed 交接允许短暂闪烁；展开态由 frontend-local ViewModel 生命周期持有，
  revert 或 generation 变化时失效。

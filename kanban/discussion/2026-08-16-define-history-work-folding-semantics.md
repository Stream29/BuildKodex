# Task Tree

- 确定消息间工作记录的自动折叠语义
  - 确定 message boundary 和 foldable event 集合
  - 确定 plan update、context compaction、request-user-input 等例外
  - 确定单项 group、首尾开放 group 和跨 batch 部分 group 的展示
  - 确定折叠 summary 的有界内容与 incomplete 表达
  - 确定展开后的 child 虚拟化、分页和 context action
  - 确定 pending/streaming work 与 committed group 的衔接
  - 确定 frontend-local expansion、焦点、键鼠和 generation 失效语义
  - 汇总并让用户确认完整行为

# Details

- 状态：`discussion`。本任务只确定行为，不修改代码。
- 已确认目标是在相邻 message 之间自动折叠 tool call 和 reasoning。
- 语义必须覆盖历史窗口从 group 中间开始或结束，以及跨 storage batch 才能确认边界的情况。
- 折叠 summary 必须保持内容和计算成本有界，不得为生成摘要读取、解析或布局完整大型 payload。
- 展开态属于 frontend-local UI state，不进入共享 session state：`checklist/tui-interaction-components.md:100`。
- 既有设计要求 plan update 独立展示；除非用户明确改变该语义，自动分组不得隐式吞并 plan update：`kanban/done/2026-07-31-render-standalone-plan-updates.md`。
- 新行为必须保留 child event 的稳定定位、context action、revert 后失效和多 frontend 独立交互能力。
- 本任务结论将作为 `WorkGroup` 投影的输入；未确认前不得编写新行为。

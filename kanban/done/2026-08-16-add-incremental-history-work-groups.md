# Task Tree

- [done] 增加增量 History WorkGroup 投影
  - [done] 采用已确认的自动折叠语义
  - [done] 确定 semantic entry、group identity 与 storage span contract
  - [done] 实现跨 batch open-edge 与有界边界归并
  - [done] 实现有界投影缓存和失效语义
  - [done] 实现按需 child 读取、渲染与状态复用
  - [done] 覆盖 append、revert、transient handoff 与交互
  - [done] 完成自动化、性能和 release 验证

# Details

- 状态：`done`。增量投影、渲染交互、失效处理和验证均已由
  [`History WorkGroup` 实现任务](../done/2026-08-18-implement-history-work-groups.md)完成。
- 前置语义见
  [`确定消息间工作记录的自动折叠语义`](../done/2026-08-16-define-history-work-folding-semantics.md)；
  既有 History 扩展性基础见
  [`修复既有 History 的大历史扩展性`](../done/2026-08-16-stabilize-existing-history-scalability.md)。
- `WorkGroup` 顶层只持有 sparse storage index range、child count 和 expansion state，不保留完整 decoded event。
- 最新端只重新投影 open prefix；旧端每批读取 64 个 stable item，并在 foldable cutoff 后最多再有界延伸 64 个。
- 展开后按需读取 child，并复用 child renderer、expansion state 和精确 storage context action；并发读取受有界
  semaphore 限制。
- 第一阶段未引入持久化 sidecar index，也未复制第二套 History 数据源。
- 自动化验证覆盖 foldable/breaker、跨 batch、强制切分、append、revert、状态复用和键鼠交互。

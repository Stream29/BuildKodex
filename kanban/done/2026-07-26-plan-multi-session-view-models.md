# Task Tree

- [done] 支持 CLI 多 Session、按需 Agent ViewModel 与状态投影优化
  - [done] 收集现有状态与执行所有权信息
    - [done] 盘点 Session registry、selected-only 投影和后台执行路径
    - [done] 盘点应用级状态、命令、草稿和 popup 生命周期
    - [done] 盘点 frontend-local history 状态与 New Session 边界
    - [done] 盘点 aggregate StateFlow、完整 history snapshot 和 lazy list 数据路径
  - [done] 确定目标边界
    - [done] 定义 application、Session、Agent、New Session 与 frontend-local scope
    - [done] 固定 root Agent 常驻和 subagent ViewModel 按需 materialize
    - [done] 固定有限 history window、稳定 child handle 与明确生命周期
    - [done] 修正 application 导航与 child 状态所有权
  - [done] [完成 CLI 全层级 contract/ViewModel 迁移并收敛 SessionTreeCliScreen](2026-08-07-refactor-session-tree-cli-screen.md)

# Details

- 状态：`done`；本规划已由 2026-08-07 的正式迁移任务合并、修订并完成。
- 原规划中的 state slice 和通用 overlay 方案已按后续讨论修正为父子 ViewModel 各自拥有状态，以及明确的 application popup contract；以被引用的完成任务及当前 checklist 为准。
- `ApplicationNavigationState` 原子持有 `List<SessionViewModel>` 与 `selectedIndex`，消除了旧 `activeTab`、`activeNewSession()`、`selectedTree` 和独立 child registry 跨快照组合造成的标签页竞态。
- 新 CLI 已通过 Linux X64/JVM 回归、Compose compiler report、Native 链接和隔离 PTY smoke；原 `Required value was null` 崩溃路径不再存在。
- ViewModel 层级边界见 [`checklist/cli-session-view-models.md`](../../checklist/cli-session-view-models.md)，状态所有权与有限 history 见 [`checklist/cli-view-model-state.md`](../../checklist/cli-view-model-state.md)。

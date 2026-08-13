# Task Tree

- [done] 让 frontend 直接消费父子 ViewModel contract
  - [done] 审计父子状态复制与稳定 child handle
  - [done] 简化 Agent 自有状态 contract
  - [done] 简化 Session surface 与 child handle
  - [done] 简化 Application navigation child registry
  - [done] 清理 state-slice 命名并更新测试文档
  - [done] 使用 JDK 26 验证 contract 图

# Details

- 用户明确选择本轮只重塑尚未接入的 contract，不改造现有 ViewModel 实现或 frontend。
- 父 ViewModel 只发布自己拥有的状态和稳定 child ViewModel handle；frontend 直接订阅对应 child。
- 不复制 child mutable state，不保留只为 frontend 转字段的 presentation projection。
- 只有真实原子关系、状态机和未 materialize child 的轻量 topology 可以使用组合状态。
- `ApplicationNavigationState` 直接保存有序 `SessionSurfaceViewModel` handles 与 active handle，不再包装 tab projection。
- persisted Session 直接暴露稳定 root Agent 与 selected Agent handle；factory 完成打开后才返回。
- Agent settings capability 由 Agent 与 Session surface contract 直接复用，删除 summary/plan 对 settings 的重复投影。
- 修改范围为 `app-contract-agent`、`app-contract-session-v2`、`app-contract-application` 及对应 contract 测试与约束。
- 旧 `app-contract-session`、`app-contract-new-session` 与当前 settings/frontend adapter 保持不变，等待统一 contract 正式接入时删除。
- `app-contract-agent` 删除了从 settings 重复投影的 summary/plan state，并抽出可复用的 `AgentSettingsViewModel` capability。
- `app-contract-session-v2` 直接发布 root/selected Agent handle，factory 在 root 可用后返回；删除 name/update/selection/availability wrapper。
- `app-contract-application` 直接保存有序 `SessionSurfaceViewModel` 与 active handle，并改为依赖 `app-contract-session-v2`。
- JDK 26 下三个目标 contract 模块的 `allTests` 与 `check` 均通过；Session v2 的 JVM 与 Linux x64 测试实际执行通过。
- IntelliJ IDEA build 与错误级 inspection 均通过，仅报告既有 Native cross-compilation 与 deprecated property 警告。
- 已确认目标 contract 中不再存在旧 wrapper symbol 或 `*StateSlices.kt`，根仓库与 `Kodex/` 的 `git diff --check` 均通过。

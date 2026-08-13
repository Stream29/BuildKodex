# Task Tree

- [done] 用列表下标表达 Session 标签页导航
  - [done] 审计 Session surface key 调用面
  - [done] 收敛统一 `SessionViewModel` parent
  - [done] 建立 tabs 与 selectedIndex 联合 state
  - [done] 删除 key/id 与索引相关包装
  - [done] 更新测试、约束并使用 JDK 26 验证

# Details

- 用户明确要求删除 `SessionSurfaceKey`，按标签页列表顺序下标索引。
- Application navigation 只由 `List<SessionViewModel>` 与 `selectedIndex: Int` 组成一个不可变联合 state。
- `selected` 是 `tabs[selectedIndex]` 的派生值，不作为第三份状态保存。
- 统一 parent contract 从 `SessionSurfaceViewModel` 收敛为 `SessionViewModel`。
- `PersistedSessionViewModel.sessionIndex` 继续作为持久化领域身份，不再包装为标签页 key。
- New Session 以稳定 ViewModel 实例和列表位置区分，不再需要 `NewSessionId`。
- `ApplicationNavigationState` 删除 active handle 与 revision，只保存 `tabs` 和 `selectedIndex`，并通过下标派生 `selected`。
- Navigation registry 只禁止同一 ViewModel 实例重复出现，不建立另一种业务 key。
- `selectTab(index)` 直接更新 selected index；需要跨挂起边界精确寻址的 close/materialize/overlay 命令继续携带稳定 child handle。
- JDK 26 下 `app-contract-session-v2` 与 `app-contract-application` 的 `allTests` 和 `check` 均通过。
- IntelliJ IDEA build 与错误级 inspection 均通过，仅报告既有 Native cross-compilation 与 deprecated property 警告。
- 目标 contract 中不再存在 `SessionSurfaceKey`、`NewSessionId` 或 `SessionSurfaceViewModel`。
- 根仓库与 `Kodex/` 的 `git diff --check` 均通过。
- 本轮继续只重塑尚未接入的 session-v2/Application contract、测试和约束，不改现有实现或 frontend。

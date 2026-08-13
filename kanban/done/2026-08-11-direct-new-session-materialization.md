# Task Tree

- [done] 让 New Session 直接 materialize persisted Session
  - [done] 审计 materialization 状态与调用边界
  - [done] 确认直接返回 persisted child 的契约
  - [done] 删除 materialization state/request/result
  - [done] 对齐 Application 原子导航替换
  - [done] 更新测试、约束并使用 JDK 26 验证

# Details

- 用户明确要求 `NewSessionViewModel.materialize()` 直接返回 `PersistedSessionViewModel`。
- Materialization 是一次命令，不是需要长期发布的 New Session UI state。
- 创建失败由命令异常表达；不保留 `NewSessionMaterializationState` 或配套 failure/request/start-result wrapper。
- Application 仍拥有有序 child registry，并在命令返回后按原 New Session handle 做一次原子导航替换。
- `NewSessionViewModel` 只增加无参数 `suspend fun materialize(): PersistedSessionViewModel`；它在自身命令边界捕获并消费当前 settings/composer。
- `ApplicationViewModel.materializeNewSession(target)` 验证 exact child、调用该命令，并在同一 application command boundary 原位替换导航 child。
- Materialization 抛出时保留原 New Session child 和 draft；成功后 Application 发布一次替换后的 navigation state。
- 删除 `ApplicationNewSessionSubmissionResult`，不再以 Empty/Stale/Busy/Failed state 表达命令结果。
- 本轮继续只重塑尚未接入的 session-v2/Application contract、测试和约束，不改现有实现或 frontend。
- `NewSessionState.kt` 已删除；目标 contract 中不再存在 materialization state、failure、request、start-result 或 operation id。
- JDK 26 下 `app-contract-session-v2` 与 `app-contract-application` 的 `allTests` 和 `check` 均通过。
- IntelliJ IDEA build 与错误级 inspection 均通过，仅报告既有 Native cross-compilation 与 deprecated property 警告。
- 根仓库与 `Kodex/` 的 `git diff --check` 均通过。

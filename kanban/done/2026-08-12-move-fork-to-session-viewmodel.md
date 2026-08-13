# Task Tree

- [done] 将 fork 行为归还具体 Persisted Session
  - [done] 审计 Application fork API 与准确 source owner
  - [done] 从 `ApplicationViewModel` 删除两个 fork API
  - [done] 在 `PersistedSessionViewModel` 建立统一 fork contract
  - [done] 清理 Application 的直接 Agent 依赖
  - [done] 修订 Session/Application 所有权约束
  - [done] 使用 JDK 26 与 IDEA 验证

# Details

- 用户确认 fork 是具体 persisted Session aggregate 的行为，Application 外层不参与 fork。
- `forkSession()` 与 `forkHistory()` 都以某个 Session 内准确 Agent 的 committed-history boundary 为源，不应由 Application 重新解析 Session/Agent address。
- `PersistedSessionViewModel.fork(source, target)` 接收该 Session registry 中的 exact `AgentViewModel` handle 和 `AgentHistoryTarget`，创建新的 persisted root Session 并返回其 index。
- Fork 不修改 Application navigation；frontend 是否调用 `openSession(newIndex)` 属于独立导航选择。
- 本轮只修改尚未接入的 contract、模块依赖和架构约束，不改旧 implementation 或 frontend。
- 验证范围固定为 session-v2/application contract allTests/check、IDEA build/lint、依赖和符号审计以及 whitespace 检查。
- 修改路线和验证范围已经确定，可直接执行。
- 使用 `/home/stream/.jdks/openjdk-26.0.2` 执行 session-v2/application contract allTests 与 check，Gradle 构建成功。
- IDEA source build 成功且 error-level lint 无问题；仅有既存 Native cross-compilation 与 `kotlin.native.cacheKind` warning。
- Application contract 中的两个 fork API、Agent import 和直接 Agent Gradle 依赖审计为空；两个仓库与 untracked contract 文件的 whitespace 检查通过。

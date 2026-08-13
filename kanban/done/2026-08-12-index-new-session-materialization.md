# Task Tree

- [done] 用 tab index 定位 New Session materialization
  - [done] 审计 navigation registry 与 materialize 所有权
  - [done] 将 Application materialize 参数改为 `tabIndex`
  - [done] 固定索引校验、失败和原位替换语义
  - [done] 修订 navigation/materialization 约束
  - [done] 使用 JDK 26 与 IDEA 验证

# Details

- 用户确认 Application 负责父级 tab list 的结构变化，因此 `materializeNewSession()` 应按 tab index 定位槽位，而不是接收 child handle。
- `NewSessionViewModel.materialize()` 继续无参数捕获并消费自己的 settings/composer，返回 `PersistedSessionViewModel`。
- Application 在自身串行化边界内校验当前 `tabs[tabIndex]` 是 New Session，失败不改变 navigation，成功在同一位置一次替换并保持 `selectedIndex`。
- 参数命名为 `tabIndex`，避免与 persisted `sessionIndex` 混淆。
- 本轮只修改尚未接入的 contract 和架构约束，不改旧 implementation 或 frontend。
- 验证范围固定为 application/session-v2 contract allTests/check、IDEA build/lint、符号审计与 whitespace 检查。
- 修改路线和验证范围已经确定，可直接执行。
- 使用 `/home/stream/.jdks/openjdk-26.0.2` 执行 application/session-v2 contract allTests 与 check，Gradle 构建成功。
- IDEA source build 成功且 error-level lint 无问题；仅有既存 Native cross-compilation 与 `kotlin.native.cacheKind` warning。
- `materializeNewSession` 参数和约束审计只保留 `tabIndex: Int`，两个仓库与 untracked contract 文件的 whitespace 检查通过。

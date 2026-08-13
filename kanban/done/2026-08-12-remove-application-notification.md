# Task Tree

- [done] 删除未接入的 Application notification 并修正 popup contract
  - [done] 审计 producer、consumer 与 dismiss 调用面
  - [done] 删除 Application notification contract
  - [done] 修正 Application popup 语义
    - [done] 删除 overlay content/request-id 消息模型
    - [done] 建立独占 popup 实例与 child ViewModel contract
    - [done] 建立显式 open、dismiss 与 child 生命周期约束
  - [done] 修订 Application 状态所有权约束
  - [done] 使用 JDK 26 与 IDEA 验证

# Details

- 用户确认删除尚未接入的 Application notification。
- 用户确认 popup 可以由 Application ViewModel 持有，但必须表达当前交互 surface 与 child ViewModel 生命周期，不能继续使用近似 notification 的 `content + requestId` 模型。
- `ApplicationNotification`、`ApplicationNotificationLevel`、`ApplicationViewModel.notification` 与 `dismissNotification()` 只有 contract 声明，没有实现、producer、frontend consumer 或调用面。
- Application 命令通过返回值或异常报告结果，退出过程由 lifecycle 表达；准确 child 的通知不提升到 Application。
- 删除 `ApplicationState.kt` 中的两个 notification 类型，并删除 `ApplicationViewModel` 中对应 Flow 与 dismiss 命令。
- 将 Application overlay 改为独占 popup：每次打开产生具有对象身份的 open state，直接携带准确 child ViewModel；dismiss 使用 expected popup handle 防止旧回调关闭新 popup。
- Settings 与 Session Catalog popup 直接携带现有 child ViewModel；Rename/Delete 使用各自的 popup child ViewModel contract。
- 更新 `cli-session-view-models.md`、`cli-view-model-state.md` 与 frontend 边界，使 Application 所有权只包含 navigation、popup、lifecycle 和稳定 child handle。
- 验证目标模块 tests/check、IDEA build/inspection、符号清理与两个仓库的 `git diff --check`。
- JDK 26 Gradle daemon 位于 `/home/stream/.jdks/openjdk-26.0.2`；`:app-contract-application:allTests` 与 `:app-contract-application:check` 成功。
- IDEA source build 成功且 error-level lint 无问题；仅保留既有 Native cross-compilation、`kotlin.native.cacheKind` 与显式 `Unit` 风格 warning。
- 旧 Application notification/overlay 符号审计为空；根仓库、`Kodex/` 与 untracked Application contract 文件的 whitespace 检查通过。
- 删除路线与验证范围已确定，可直接执行。
- 本轮只修改尚未接入的 Application contract、测试和约束，不改现有实现或 frontend。

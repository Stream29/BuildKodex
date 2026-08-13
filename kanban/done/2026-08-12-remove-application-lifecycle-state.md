# Task Tree

- [done] 删除未接入的 Application lifecycle contract
  - [done] 审计显示、退出与 shutdown 调用面
  - [done] 删除 `ApplicationLifecycleState`
  - [done] 删除 `ApplicationViewModel.lifecycle` 与 `requestExit()`
  - [done] 修订 Application 所有权和 host 生命周期约束
  - [done] 使用 JDK 26 与 IDEA 验证

# Details

- 用户确认当前唯一退出方式是 Ctrl+C，frontend 没有 Exit/Quit 操作，也没有 `requestExit()` 调用。
- 新 `ApplicationLifecycleState` 与 `ApplicationViewModel.lifecycle` 只有 contract 声明，没有 implementation、producer、collector 或显示用途。
- Suspend factory 成功返回已经表达启动完成，`shutdown()` 成功返回已经表达关闭完成，失败继续通过异常表达。
- 本轮只删除尚未接入的 Application contract 和对应约束；旧实现中的 `exitRequested`、`requestExit()` 与 host watcher 留到 implementation/frontend 迁移阶段处理。
- 修改范围固定为 `app-contract-application`、相关 checklist 和主迁移规划；验证包含目标模块 allTests/check、IDEA build/lint、符号审计与 whitespace 检查。
- 删除路线和验证范围已经确定，可直接执行。
- 使用 `/home/stream/.jdks/openjdk-26.0.2` 执行 `:app-contract-application:allTests` 与 `:app-contract-application:check`，Gradle 构建成功。
- IDEA source build 成功且 error-level lint 无问题；仅有既存 Native cross-compilation 与 `kotlin.native.cacheKind` warning。
- 新 Application contract 中的 `ApplicationLifecycleState`、`lifecycle` 和 `requestExit()` 符号审计为空，whitespace 检查通过。

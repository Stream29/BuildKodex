# Task Tree

- [done] 移除 Application 根级数据源聚合
  - [done] 审计 settings、models 与 authentication 调用面
  - [done] 删除根级 Flow 与代理命令
  - [done] 清理 Application contract 依赖
  - [done] 固定 child constructor injection 边界
  - [done] 更新约束并使用 JDK 26 验证

# Details

- 用户明确要求 Application ViewModel 不直接依赖或暴露 settings、models 与 authentication。
- 准确的 child ViewModel 实现通过 Koin constructor injection 或 typed factory 获取对应 store/catalog。
- Contract 模块不依赖 Koin；ViewModel 内禁止使用 `getKoin()` 或从 Application 反查依赖。
- 删除 Application 的 settings/models/authentication Flow、refresh 代理与全局 settings transform 命令。
- 删除只服务于根级认证投影的 `ApplicationAuthenticationState`。
- 现有 `GlobalSettingsViewModel` 已拥有 settings、model options、sanitized authentication、account usage 与 MCP state，无需 Application 重复发布。
- 现有 `OpenAiLoginViewModel` 已拥有登录状态与 effect，无需 Application 提供认证刷新代理。
- Application contract 清理后不再依赖 `app-shared-settings-contract` 或 `openai-models`。
- `ApplicationViewModel.refresh()` 也删除；frontend 直接调用 selected Session 或具体 child 的 refresh。
- JDK 26 下 `app-contract-session-catalog`、`app-contract-application` 与 `app-contract-session-v2` 的 `allTests` 和 `check` 均通过。
- IntelliJ IDEA build 与错误级 inspection 均通过，仅报告既有 Native cross-compilation 与 deprecated property 警告。
- 目标 Application contract 中不再存在根级 settings/models/authentication Flow、认证投影或对应 refresh/update 代理。
- 根仓库与 `Kodex/` 的 `git diff --check` 均通过。
- 本轮继续只重塑尚未接入的 contract 和约束，不改现有实现或 frontend。

# Task Tree

- [done] 提取惰性 Session Catalog ViewModel
  - [done] 审计 catalog 所有权与启动加载
  - [done] 建立独立 session-catalog contract 模块
  - [done] 让 Application 持有稳定 catalog child
  - [done] 删除 Application catalog 聚合 state
  - [done] 更新测试、约束并使用 JDK 26 验证

# Details

- 用户明确要求将 Session catalog 拆为独立 ViewModel，并放入新的 `app/contract/session-catalog` 模块。
- `SessionCatalogViewModel` 对应 Select Session popup，直接发布 `StateFlow<List<SessionCatalogEntry>>`。
- ViewModel 构造和 Application 启动不得读取全量 catalog；首次打开 popup 时才调用 `refresh()`。
- `refresh()` 成功后原子替换列表；失败抛出并保留上一次成功列表，不建立 status/revision wrapper。
- Application 只持有稳定 `SessionCatalogViewModel` child，并继续负责 open/delete/navigation 等跨 child 操作。
- 删除 `SessionCatalogState`、`SessionCatalogStatus` 与 `ApplicationViewModel.refreshSessionCatalog()`。
- 新 contract 包名使用 `io.github.stream29.kodex.app.sessioncatalog.contract`；Application contract 增加单向模块依赖。
- `ApplicationViewModel.refresh()` 明确不触发 catalog 加载。
- Catalog child 由 Application 生命周期拥有，但实现可以 lazy 创建；无论何时创建，首次 `refresh()` 前不得进行 catalog I/O。
- JDK 26 下 `app-contract-session-catalog`、`app-contract-application` 与 `app-contract-session-v2` 的 `allTests` 和 `check` 均通过。
- IntelliJ IDEA build 与错误级 inspection 均通过，仅报告既有 Native cross-compilation 与 deprecated property 警告。
- 目标 contract 中不再存在 `SessionCatalogState`、`SessionCatalogStatus` 或 `refreshSessionCatalog()`。
- 根仓库与 `Kodex/` 的 `git diff --check` 均通过。
- 本轮继续只重塑尚未接入的 contract、测试和约束，不改现有实现或 frontend。

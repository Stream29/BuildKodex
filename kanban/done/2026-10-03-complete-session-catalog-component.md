# Task Tree

- `Inventory existing catalog spec and ViewModel plus remaining host UI`()
- `Specify interaction ports and exact Delete child ownership`()
- `Review lazy refresh, mutation failure, and popup lifecycle`()
- `Extend the existing spec and ViewModel and add the complete renderer`()
- `Add fake interaction and full renderer regression tests`()
- `Review owned diffs and publish exact integration handoff`()
- `Validate integrated component and downstream regression checks`()
- `Record acceptance and archive component task`

# Details

- 实施/集中验证完成；本组件 52 项 JVM 测试通过，真实 Application 导航/
  dismissal、menu admission、删除后刷新与资源生命周期验证见主任务。
  用户已验收，内层代码随批次提交为 `721ac0fe`。

## Scope and source map

- 属于 [五组件批次](2026-10-03-extract-session-configuration-and-browsing-components.md)，
  用户已审阅并授权实现，当前 executable。
- 已有 `app/component/session-catalog/spec` 和 `impl/viewmodel`；补
  `impl/view`，不新建另一 catalog controller/model/protocol。
- 基线 UI 在 SessionTreeCliScreen：Popup、header、row、loading indicator、
  context menu、timestamp/label/running 展示和 renderer-local deleteTarget。
  现已迁入 [完整目录 renderer](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-catalog/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/sessioncatalog/SessionCatalogPopup.kt#L60)。

## Component boundary and ports

- 复用 SessionCatalogDependencies/ViewModel/State/Entry，保持 snapshot shape、
  ordering、serialized command 行为及 backend mutations。补充实际会传播的
  dependency failure `@throws`，不只写 cancellation。
- 在同一 spec 声明 opening/dismissal 的 narrow host port 或 effect；
  open 绑定捕获 sessionIndex，dismiss 绑定准确 ApplicationPopupState open。
  组件不得引用 ApplicationViewModel、Session registry 或完整 RPC。
- 业务 deleteTarget/准确 Delete child 放已有 component VM 的 interaction
  所有权；child 创建明确接线 Session Delete spec/impl，不在 renderer 创建。
  旧 load snapshot 与新增 interaction 可分开暴露，但不能复制两份 sessions。
- context-menu geometry、anchor placement、focus、scroll、running 动画留 view；
  business callback 捕获准确 row/session target，过期 renderer 回调不操作新目标。
- 前端宿主保留 root tab/child disposal 与 registry failure reporting；
  Catalog 不打开 Session 来渲染行，不直接释放后端 Session 资源。

## Behavior and review gates

- 构造保持 Unloaded/无读取；打开时显式 refresh，避免 constructor + renderer
  双读。Loading 显示 progress，Loaded empty 才显示空目录。
- 列表保留后端顺序与 showArchived；行名、activity/running、时间戳从同一
  catalog snapshot 展示，不拼接独立缓存当作联合原子快照。
- filter 同值 no-op；mutation 完成后 reload 当前 filter，失败恢复原 snapshot
  并传播，不能把已成功 mutation 回滚或自动重放。fork 基线只 fork/reload，
  不顺带自动打开新 Session。
- createdAt 只取 snapshot 的 timestamp-zero，updatedAt 为采样最新值；
  不为菜单额外 backend I/O，不搜后续记录补 createdAt。
- open 成功才 dismiss 准确 popup；失败仍按 registry 规则报告，不表现为成功关闭。
  删除 false、取消、mutate-success/reload-failure 必须区分，不能重复删除当补偿。
- 移入 VM 的 Delete child 清理、替换/迟到 callback、popup owner close 与
  renderer unmount 分别声明；不因移动 deleteTarget 承诺后端删除可撤销。
- shared timestamp/running/label helper 若多处使用，只提取纯 renderer helper；
  不能让组件 view 反向依赖 app-view-application。

## Integration handoff (implementation-lane checkpoint)

- Host factory SAM now has two arguments:
  `SessionCatalogViewModelFactory.create(dependencies: SessionCatalogDependencies, interactions: SessionCatalogInteractions?): SessionCatalogViewModel`.
  [Authoritative factory and lifecycle contract](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-catalog/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/sessioncatalog/contract/SessionCatalogViewModel.kt#L198).
- Concrete constructor:
  `DefaultSessionCatalogViewModel(scope: CoroutineScope, dependencies: SessionCatalogDependencies, interactions: SessionCatalogInteractions? = null)`.
  Null ports support data-only tests; constructor performs no I/O.
  [Single extended implementation](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-catalog/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/app/sessioncatalog/DefaultSessionCatalogViewModel.kt#L25).
- `SessionCatalogInteractions` has
  `suspend fun openSession(sessionIndex: Int): Unit` and
  `fun dismissPopup(): Unit`. Inject the latter as a closure capturing the exact
  `ApplicationPopupState.SessionCatalog` opening, never the current popup.
  The host remains responsible for registry reporting, root tabs and backend cleanup.
  [Typed ports and exact child handle](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-catalog/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/sessioncatalog/contract/SessionCatalogInteractions.kt#L6).
- Existing VM gains
  `val deleteTarget: StateFlow<SessionCatalogDeleteHandle?>`,
  `suspend fun requestOpen(entry: SessionCatalogEntry): Unit`,
  `fun requestDelete(entry: SessionCatalogEntry): SessionCatalogDeleteHandle?`,
  `fun dismissDelete(handle: SessionCatalogDeleteHandle): Unit`, and `fun dismiss(): Unit`.
  Changed/missing row snapshots are rejected by captured-entry value comparison;
  equal refreshed values remain valid because StateFlow may conflate equal snapshots.
  Commands use the captured index/title, never the latest selection; exact child
  and menu handles use `===`. This is not a new permanent backend identity protocol.
- `SessionCatalogDeleteHandle(target: SessionCatalogEntry, viewModel: SessionDeleteViewModel)`
  exposes the exact child. VM explicitly composes
  `createSessionDeleteViewModel(sessionIndex: Int, threadName: String?, dependencies: SessionDeleteDependencies)`.
  Child dependency guards its exact handle, deletes/reloads once, and dismisses
  only that handle after true. False/reload failure retains confirmation.
  Success removes/closes the handle inside the VM; the Delete renderer's result
  callback performs no second dismissal or deletion.
- View entrypoint:
  `@Composable fun BoxScope.SessionCatalogPopup(viewModel: SessionCatalogViewModel)`
  in `io.github.stream29.kodex.cli.sessioncatalog`; mount refreshes once.
  Main registers `:app-component-session-catalog-impl-view`, adds host dependency,
  and replaces the old Application-bound popup with this renderer.
  Rendering is keyed to the exact VM; replacing it cancels old renderer callers
  rather than reusing their scope for a new popup.
  [Complete renderer](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-catalog/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/sessioncatalog/SessionCatalogPopup.kt#L44).
- Central integration deduplicated running animation and timestamp formatting into
  `app/view/components`; `sessionBrowserLabel` is public here and shared by tabs.
  Preserve terminal-cell ellipsizing, dim relative-time
  suffix, fallback `Session <index>`, backend-only running spinner (10 Braille
  frames / 100 ms), and timestamp `yyyy-MM-dd HH:mm:ss UTC±offset`.
  Menu dates are direct captured snapshot values, no asynchronous helper/I/O needed.
  [Catalog-specific visual helpers](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-catalog/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/sessioncatalog/SessionCatalogVisuals.kt#L25).
- Helpers available to this module's tests are `SessionCatalogHeader`,
  `SessionCatalogRow`, `SessionCatalogContextMenuPopup`,
  `SessionCatalogLoadingIndicator`, `sessionBrowserLabel`,
  `rememberRunningIndicatorFrame`, `formatPopupTimestamp`, and frame constants.
  Running/timestamp helpers are imported from `cli.components`; original Application
  helper suites have been moved to their owning modules.
- No Gradle, commit/push, branch switch or shared IDE operation performed.
  Centralized registration and host compilation are complete. Final regression
  status is recorded in the parent batch; scoped JVM checks:
  `:app-component-session-catalog-spec:jvmTest`,
  `:app-component-session-catalog-impl-viewmodel:jvmTest`,
  `:app-component-session-catalog-impl-view:jvmTest`.
  Non-JVM validation remains outside this run.

## Owned checks and validation status

- Existing serialization/Dependency suites are preserved; no wire entry or
  snapshot-union shape changed.
- [New fake interaction tests](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-catalog/impl/viewmodel/src/commonTest/kotlin/io/github/stream29/kodex/app/sessioncatalog/SessionCatalogInteractionTest.kt#L18)
  cover lazy construction, real exception identity, stale/equal rows, exact child
  replacement, queued/accepted old deletion, false/true, all mutation reload
  failures, caller/owner cancellation, no rollback/replay and fork without open.
- [Full renderer tests](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-catalog/impl/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/sessioncatalog/SessionCatalogPopupTest.kt#L31)
  cover once-per-mount refresh, progress-vs-empty, backend ordering/filter,
  pointer menu, captured dates, exact open/dismiss, failed open, owned Delete
  child/false/reload failure/Cancel, replacement, running animation and unmount
  cancellation without closing the host catalog.
- [Visual helper tests](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-catalog/impl/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/sessioncatalog/SessionCatalogVisualTest.kt#L29)
  cover terminal-width labels, dim suffix, nullable timestamp fields/offset format,
  Menu/Shift-F10, header filtering and loading animation.
- Command execution now retains the caller Job while linking catalog-owner
  cancellation. This enforces renderer-unmount/caller cancellation without
  cancelling the host/owner or undoing already accepted backend mutations.
- Static review and whitespace checks passed for owned tracked/untracked files;
  spec/VM have no renderer dependency and view has no app-view-application
  dependency. Central compilation and test results are tracked in the parent batch,
  not inferred from the implementation lane's static review.
- Shared hosts/contracts/old UI tests/common helpers/host Gradle were not edited.
  No temporary files or external resources were created.

## Remaining main integration checks

- 保留 SessionCatalogContractTest/DependencyTest；增加 typed host/deletion
  fake port 测试：lazy read、serialized operations、archive mode、
  open fail、exact dismiss、stale row/child、delete false/reload fail/close。
- Main 已迁移原 Catalog header/menu/loading/label 测试至组件 view，
  running 测试至共享视觉 helper；完整 renderer 回归纳入集中验证。
- Application 测试保留 tab 清理、registry failure 和准确 popup replacement；
  UI 测试不能替代 root fork/delete 后端语义。
- Catalog 线可改原两个 component 项目与新增 view，宿主/工厂/旧 tests/helper
  去重由整合负责人统一处理；需要的 spec 扩展在并行前先固定。

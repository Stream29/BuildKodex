# Task Tree

- `Inventory sparse scans, nonce validation, row reads, and both sidebar entrypoints`()
- `Specify borrowed data ports and per-interaction row/detail lifetimes`()
- `Review stale-result suppression and exact scroll routing`()
- `Extract component spec, scan/interaction ViewModel, and full browser renderer`()
- `Integrate stable Agent child and both sidebars without moving Shell UI`()
- `Validate sparse history, cache replacement, hover, and Check out`()
- `Record acceptance and archive component task`

# Details

- 实施/集中验证完成；本组件 45 项 JVM 测试通过，真实存储扫描/nonce、
  两侧栏、准确 Agent 滚动及离线 History 录制验证见主任务。用户已验收，
  内层代码随批次提交为 `721ac0fe`。

## Scope and source map

- 属于 [五组件批次](2026-10-03-extract-session-configuration-and-browsing-components.md)，
  用户已审阅并授权实现，当前 executable。
- 目标 `app/component/history-index/{spec,impl/viewmodel,impl/view}`。
- 原 contract：HistoryIndexWindow/Entry/Detail/Kind/ViewModel；
  原扫描/投影实现 `app/viewmodel/agent/.../HistoryIndexViewModel.kt`。
- renderer 在 SessionSidebar：Body/Row、hover detail、context menu、graph；
  SessionTreeCliScreen 管理左右侧 target/hover grace/menu/Check out 路由。
  现已迁入 [独立行读取与两侧 renderer](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history-index/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/historyindex/HistoryIndexComponent.kt#L102)。
- 不包含完整 History 虚拟列表/Composer/Shell 内容，也不迁移 Agent 的
  revert/fork confirmation；此 Index 菜单实际只有信息和 Check out。

## Proposed spec and dependencies

- 原 public types 迁入单一 spec，保留 FQCN；真实 factory 接受 typed deps/
  ownerScope。父 Agent 发布稳定 child，renderer 不从 Agent service locator 找存储。
- deps 给准确 Agent 的 lightweight index/timestamp 读取能力、latest index、
  cacheNonce 和 external-write 生命周期观察；借用现有领域值。
  不暴露完整 RpcSessionBinding/AgentState store 或供前端任意访问 Home。
- 扫描/映射留 component impl：稀疏 append、回退重扫、nonce replacement、
  同位置 external-write invalidation，支持摘要/detail/Message 时间戳准确读。
- row/detail 的 Loading/Ready/Failed 从宿主 renderer-local async 状态移入
  component handle；spec 明确 acquire/release、generation/index identity、
  重复 consumer/两侧栏隔离、caller cancellation 与关闭后的行为。
- renderer 保留 row placement/anchor、左右 popup 定位、pointer coordinate、
  scroll/followLatest、focus 与 hover grace；没有 Compose/Mosaic 类型进 spec。
- Check out 经绑定准确 Agent 的 narrow scroll port/effect；callback 使用
  captured generation/index/owner 并在准入时校验，不读取新选中的 Agent。

## Behavior and review gates

- indexes 保持 oldest-first/sparse，不用 count 推导连续范围；index=0 的
  初始 CleanCompactionPoint 按基线过滤，其余真实条目保留。
- generation 在生产依赖 cacheNonce；contains/read 前后核对 owner/window/nonce。
  同数字 index 可被替换，不能把 index 当稳定永远有效的 key。
- nonce 改变、latest 回退或 external-write 在同位置替换时 invalidation；
  stale row/detail/timestamp 响应不得重新发布到新 generation。
- row 加载中显示省略标记，失败显示安全 `[error]`，取消不当作失败；
  detail 保留 secret answer 隐藏、image/encrypted 占位及完整问题/计划呈现。
- 时间戳只读 exact Message 对应记录，非 Message/缺失不显示；
  不使用后续 timestamp、当前时间或解析 URI 作为补值。
- **Check out 只请求完整 History 滚动定位**，不恢复旧模型状态、
  不调用 revertHistory/fork，也不自动运行 Agent。
- 两侧栏可展示同一 child；各自 local interaction handle/anchor 不串目标，
  旧 Agent/nonce/row 消失及时隐藏 hover/menu，释放本地加载等待但不关闭 Agent。
- close/owner cancellation 只取消 component jobs，不能关闭 borrowed timeline/
  后端 Session。若旧无-close接口需新增局部生命周期能力，明确父级关闭顺序，
  不让 renderer unmount 误关两个侧栏共享的 child。
- 错误类型/`@throws` 区分失效目标、普通读取失败和 CancellationException；
  旧 internal HistoryIndexLoadException 不仅为填 spec 而升级为永久协议保证。

## Checks and handoff

- 保留 HistoryIndexViewModelTest 的增量稀疏扫描、回退、准确 Message timestamp、
  摘要与 secret detail；fake deps 增加 nonce same-index replacement、
  delayed read、external-write equal boundary、cancel/release/owner close。
- 完整 renderer 测试覆盖 Loading/error/Ready、graph、trim/ellipsis、hover grace、
  popup/菜单优先级、Check out/时间戳、scroll-away/follow-latest、两侧并存。
- Application/Agent 宿主验证选择更换、exact owner/generation/index 判断、
  scroll routing 与 Shell popup 共存；借用完整 History scroll port 不搬 History 状态机。
- Index 线只拥有新三个项目和专有测试；SessionSidebar/SessionTreeCliScreen、
  RpcAgentPresentation/Agent contract、旧 Timeline fixture/tests 由整合负责人处理。
- `app-view-history` 的 RequestUserInputHistoryRow 当前被 hover 使用；
  先确定纯 renderer helper 的公共所有者，避免新组件 view 反向依赖完整
  History/Agent UI 或复制第二份 stable event 投影。

## Integration handoff — History Index lane

- Implementation-lane checkpoint: changes were confined to the new
  `app/component/history-index/{spec,impl/viewmodel,impl/view}` trees and this handoff.
  No builds, commits, pushes, branches, IDE operations, or old/shared-source edits were performed.
- New Gradle projects are auto-discovered by the existing `includeModuleTree("app")`:
  `:app-component-history-index-spec`, `:app-component-history-index-impl-viewmodel`,
  `:app-component-history-index-impl-view`; conventional plugins are respectively
  `kodex.kmp-cli`, `kodex.kmp-viewmodel`, `kodex.kmp-view`.

### Specification and borrowed ports

- All existing `HistoryIndexWindow`, `HistoryIndexEntry`, `HistoryIndexEntryDetail`,
  `HistoryIndexEntryKind`, `HistoryIndexViewModel` retain
  `io.github.stream29.kodex.app.agent.contract` FQCNs.
  **Coordinator must remove the old contract/VM declarations before compiling.**
- [Authoritative spec](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history-index/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/agent/contract/HistoryIndexViewModel.kt#L19)
  defines the real factory/dependencies, state branches, exact read errors/cancellation,
  release/close, Check out, and renderer semantics.
- Production `window.generation` remains the exact observed index nonce, including decreasing
  or negative values. Null nonce is for local fixtures only: initial append uses generation 0,
  destructive rescans increment it. No backend nonce is synthesized.
- `HistoryIndexWindow(..., revision: Long = 0)` adds a **local renderer/read invalidation token**.
  It is not a backend nonce or history command credential. Replacement rescans advance revision
  even when production nonce and sparse indexes are unchanged; append retains revision.
  Rows re-key on `(generation, revision, index)`; host hover/menu must retain captured revision.
  `contains` rejects targets while a destructive replacement scan is pending.

```kotlin
// io.github.stream29.kodex.app.agent.contract
interface HistoryIndexDependencies {
    val timeline: IndexVersioned<CleanIndexEntry>
    val timestamp: IndexVersioned<Instant>
    val latestIndex: StateFlow<Int>
    val cacheNonce: StateFlow<Long>? // NON-null in production
    val externalWrite: StateFlow<Boolean>
    fun requestScrollToStorageIndex(index: Int)
}
fun interface HistoryIndexViewModelFactory {
    fun create(dependencies: HistoryIndexDependencies, ownerScope: CoroutineScope): HistoryIndexViewModel
}
// io.github.stream29.kodex.cli.agent — retained factory package, NEW signature
val historyIndexViewModelFactory: HistoryIndexViewModelFactory
fun createHistoryIndexViewModel(
    dependencies: HistoryIndexDependencies,
    ownerScope: CoroutineScope,
): HistoryIndexViewModel
```

- [Factory/implementation, L47 onward](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history-index/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/cli/agent/HistoryIndexViewModel.kt#L47)
  preserves the old sparse scan/mapping algorithm. Borrow the exact Agent's lightweight index/
  timestamp views and their metadata; project only `state == ExternalWrite` into the Boolean port.
  Do not pass the whole binding, Agent, state store, or RPC client.
- Bind the synchronous scroll closure to the **captured Agent's**
  `history.requestScrollToStorageIndex(index)`. Do not look up the latest selected Agent when called.
  VM `checkOut(generation, index): Boolean` validates current exact membership first; it does not
  revert, fork, run, resume, or add an RPC. Synchronous port failures/cancellation propagate.
- Keep one stable child on the materialized Agent; close it before releasing its borrowed
  timeline owners. Owner cancellation cancels only component jobs; renderer unmount must not
  close this shared child.

```kotlin
// HistoryIndexViewModel additions; original direct suspend reads remain available
val isActive: Boolean
fun acquireRow(generation: Long, index: Int): HistoryIndexReadHandle<HistoryIndexEntry>
fun acquireDetail(generation: Long, index: Int): HistoryIndexReadHandle<HistoryIndexEntryDetail>
fun acquireTimestamp(generation: Long, index: Int): HistoryIndexReadHandle<Instant?>
fun checkOut(generation: Long, index: Int): Boolean
fun close()
// Each handle: captured generation/index, StateFlow<HistoryIndexReadState<T>>, release()
// State branches: Loading, Ready(value), Failed, Closed
```

- Each acquisition owns an independent wait/handle. Release, nonce/revision invalidation,
  close, and cancellation are terminal Closed; late/non-cooperative results cannot refill it.
  Failed exposes no raw exception. Timestamp Ready(null)/Loading/Failed omit the optional field.
- `HistoryIndexLoadException` remains internal implementation detail. Adapt old scanner tests
  to the public dependency factory/public VM, not an internal class across module boundaries;
  assertions outside the new module should use the documented `IllegalStateException` contract.
  Keep null-nonce fixtures where the old tests asserted local fallback generation.

### Renderer API and host routing

- All renderer entrypoints/types are under **`io.github.stream29.kodex.cli.historyindex`**;
  no entrypoint accepts Agent/Application types.
  [Renderer API](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history-index/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/historyindex/HistoryIndexComponent.kt#L68).

```kotlin
enum class HistoryIndexSide { Left, Right }
class HistoryIndexInteractionRequest(
    side: HistoryIndexSide, viewModel: HistoryIndexViewModel,
    generation: Long, index: Int, anchor: TuiPopupAnchor,
    revision: Long = viewModel.window.value.revision,
) // mutable pointerPosition: IntOffset?; isCurrent(): Boolean
class HistoryIndexMenuRequest(target: HistoryIndexInteractionRequest, clickPosition: IntOffset?)

@Composable fun HistoryIndexSidebarBody(
    viewModel: HistoryIndexViewModel, columns: Int, rows: Int,
    side: HistoryIndexSide = HistoryIndexSide.Left,
    onHoverChanged: (HistoryIndexInteractionRequest, Boolean) -> Unit = { _, _ -> },
    onOpenMenu: (HistoryIndexMenuRequest) -> Unit = {},
    listState: LazyListState = remember(viewModel) { LazyListState().apply { requestScrollToEnd() } },
)
@Composable fun BoxScope.HistoryIndexHoverPopup(
    request: HistoryIndexInteractionRequest?, contentColumns: Int, contentRows: Int,
    onHoverChanged: (Boolean) -> Unit, onDismissRequest: () -> Unit = {},
)
@Composable fun BoxScope.HistoryIndexContextMenu(
    request: HistoryIndexMenuRequest?, onDismissRequest: () -> Unit,
)
@Composable fun BoxScope.HistoryIndexContextMenuPopup(
    anchor: TuiPopupAnchor, clickPosition: IntOffset?, index: Int,
    onDismissRequest: () -> Unit, onCheckOut: () -> Unit, timestamp: String? = null,
)
@Composable fun HistoryIndexRowContent(
    state: HistoryIndexReadState<HistoryIndexEntry>, graph: String,
    hovered: Boolean = false, pressed: Boolean = false,
)
@Composable fun BoxScope.HistoryIndexHoverContent(
    anchor: TuiPopupAnchor, side: HistoryIndexSide, pointerPosition: IntOffset?,
    state: HistoryIndexReadState<HistoryIndexEntryDetail>,
    contentColumns: Int, contentRows: Int, onHoverChanged: (Boolean) -> Unit,
)
```

- Body owns row acquire/release and independent viewport/follow-latest; hover owns the 300ms
  presentation delay and detail acquire/release; menu owns exact-opening timestamp acquire/release.
  Geometry, focus, anchors and pointer are renderer-only. Existing host grace timer and shared
  Shell/index hover/menu priority stay in `cli.app`.
- Host must filter requests by `selectedAgent?.historyIndex === target.viewModel`, captured
  revision and `target.isCurrent()`. This is presentation routing, not a selected-Agent lookup
  inside Check out. Full component menu directly invokes the captured VM's Check out port;
  there is no `selectedAgent` or `onCheckOut` argument on `HistoryIndexContextMenu`.
- Either use new request types directly or keep old `cli.app` wrappers and convert **once per
  exact opening**, preserving VM/generation/index/revision/anchor/pointer/clickPosition.
  Map old `SessionSidebarSide.Left/Right` to `HistoryIndexSide.Left/Right`; do not rebuild an
  opening on every renderer frame. Low-level popup helper retains an explicit callback for
  pure host/renderer tests.
- View has only two project dependencies: its spec and `app-view-components`; no full History
  view dependency or duplicate request-user-input projection. It borrows the relocated
  `RequestUserInputHistoryRow`, `RequestUserInputHistoryRowModel`, and
  `requestUserInputHistoryRows` under their existing `cli.history` FQCNs.
- **Required coordinator build export:** `app-view-components` must expose its existing
  `:utils-terminal-text-spec` dependency as `api`, not `implementation`, so the component can
  borrow `String.terminalCellWidth()` transitively without a third project dependency.
  Shared helper/source/build edits remain coordinator-owned. Timestamp formatting is a private
  renderer-local pure helper; ordinary formatting failure omits the timestamp. The coordinator's
  `api` export was already present at this lane's final source review.
- Remove old index row/detail/menu business loaders and index renderer duplication from the
  old host, while keeping Shell UI and shared popup arbitration in place.

### Checks delivered and pending

- [Fake-dependency VM suite](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history-index/impl/viewmodel/src/commonTest/kotlin/io/github/stream29/kodex/cli/agent/HistoryIndexComponentTest.kt#L41):
  sparse incremental scan/index-zero filtering, revert, local-vs-production equal-boundary
  external write, same-index nonce replacement, delayed scan/nonce mismatch, exact timestamps,
  late row/detail/timestamp release and nonce invalidation, cancellation, owner close, independent
  handles, safe failures, secret/image/encrypted/plan projection.
- [Renderer suite](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history-index/impl/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/historyindex/HistoryIndexComponentTest.kt#L45):
  all read branches/classification titles, sparse graph, terminal ellipsis, shared question form/
  secret hiding, delayed hover/release, both-side geometry, menu information/Check out/timestamp
  branches, revision invalidation, independent sidebar handles, scroll-away/follow-latest.
- Source/ownership/dependency and whitespace review performed; centralized results
  are recorded in the parent batch. Coordinator completed original scanner-test adaptation (its transplanted
  `HistoryIndexStorageRegressionTest.kt` was present at final review), exact captured-Agent scroll/
  selection/late-callback tests, both host sidebars, Shell popup priority/hover-grace integration,
  and centralized component/downstream/platform validation.

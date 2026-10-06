# Task Tree

- `Load the approved History boundary and exact baseline algorithms`()
- `Replace renderer coupling with meaningful MVVM operations and effects`()
- `Preserve bounded paging, identity and renderer-only state`()
- `Add fake-port and genuine-renderer regression evidence`()
- `Hand off the exact API for centralized integration`()
- `Receive independent scoped acceptance with the parent batch`()

# Details

- [主任务](2026-10-07-cutover-ui-foundations-and-home-bootstrap.md)；用户已授权实施。
- Own History spec/impl-viewmodel/impl-view source/tests，Patch 新目录除外；
  所有 Gradle、AgentRuntimeScreen、Root cache、Docs/host callers 由协调者统一处理。
- 保留原 AgentHistoryViewModel/window/item/factory，不恢复 Aggregate/Timeline、
  第二 VM、resolver、opaque handle 或 compatibility overload。
- spec/VM 删除 androidx.compose、Mosaic、LazyList/scroll event 类型。
  原唯一 bounded/window/cache/pending/streaming/seek 算法留在 VM。
- portable viewport 报告捕获原 window/实际 item 身份；stale 不改变保留策略。
  follow-latest 是 StateFlow；用户滚动判定留 View，VM 不监听 snapshotFlow。
- 标准 pending scroll effect 需支持同目标重复请求、未挂载等待、准确消费；
  late ack 不清除 newer intent，generation/target 失效不能寻址新 owner。
- renderer-only `AgentHistoryViewState` 只保存 LazyList/interaction/focus 类 UI 状态，
  由宿主按准确 VM 保留；不可含第二份业务 window/child registry/持久化 ID。
- 对 transient prefix、Work Group 行、visible chunk 淘汰及 key anchor 给出完整测试。
  原 VM 测试的真实滚动部分迁到 View 测试，纯分页测试改上报，不删除原不变量。
- 给所有公开操作/state/render/lifetime/error 写规范 KDoc/@throws。
- 早期 handoff 列出精确签名和 Root/Agent/Docs 改法；实施完成后 centralized build
  仍 pending，不运行 Gradle/IDE，不提交/推送。只修改本任务交接段落。

## Integration handoff

### Historical coordinator feedback — first central run

- `:app-component-history-impl-viewmodel:jvmTest` passed 42 cases.
- The first History View run compiled but failed three cases; log is
  `/tmp/kodex-ui-home-matrix-419.log`, XML under the View `build/test-results/jvmTest`.
  Please finish your owned implementation/tests with these exact failures in mind:
  - `AgentHistoryBoundedWindowTest.kt:100`: wheel scroll then settled snapshot still
    leaves followsLatest true; verify real item loading/input collection, not just
    weaken the assertion.
  - `AgentHistoryRendererStateTest.kt:264`: unmount/remount changes output by one
    line; compare settled output and exact offset, distinguishing fresh measure
    timing from real anchor drift.
  - Same suite `:289`: actual Work Group click does not expand; inspect the real
    rendered row/button coordinate (reverse layout), preserve real input testing.
- Root state caching, consumer Gradle retargets and Docs Boolean→Flow call sites
  are integrated by coordinator. No lane build; central rerun after your handoff.
- For the bounded-window failure, specifically check asynchronous follow reset:
  the renderer's auto-follow snapshot may jump back before the interaction
  collector flips intent, and restarting a no-replay collector per window can
  miss committed input. `MutableScrollInteractionSource(onInteractionCommitted)`
  already offers synchronous classification **in the View**; it need not be
  retained in VM or add a new bus. If used, bind only during exact renderer mount
  and clear on disposal, with latest displayed-window validation. Do not change
  LazyList's migrated algorithms to hide this race.
- Recheck after the Unconfined collector change still has those same three
  failures (`/tmp/kodex-ui-home-history-recheck-419.log`). The Work Group failure
  is actually the **Think child output** assertion at line 289, not the expanded
  header (line 288 passed). Also check whether the bounded test's first wheel
  fires before async payloads have any scrollable height: zero consumed input
  correctly leaves follow true, so wait for real loaded/measured rows before
  the gesture rather than manually setting follow or dropping the assertion.
- Once test readiness is corrected, consider ablating `Dispatchers.Unconfined`:
  the original fake-model genuine-input cases already passed without it. Retain
  a scheduling override only if a deterministic production-meaningful test
  proves it necessary, not just because a pre-payload zero-consumption gesture
  failed an unsuitable assertion.
- Latest central run of the synchronous mount listener and settled tests
  (`/tmp/kodex-ui-home-final-regression-419.log`) now passes bounded navigation
  and Work Group; **remount still drifts exactly one line**. The settled before/
  after output failure is unchanged. Do not declare that case frame timing
  resolved; coordinator will finish its root cause/fix if outside your source
  ownership. No original LazyList algorithm has been changed yet.

### Lane completion — central rerun pending

- Coordinator now owns post-handoff central fixes. A standalone LazyList
  ablation with the retained state passes exact partial-row remount without
  History/VM. The History round-trip fixture now keeps a real outside focus
  target, matching the root tab/composer: otherwise Mosaic activates the first
  partially visible History row and its existing FocusRelocation scrolls one
  line. No scrolling primitive or projection algorithm has been changed.
  This explanation remains subject to the central rerun and real root tab input.

### Coordinator closure

- Final VM 42 / View 71 JVM cases passed. The exact remount output/anchor/index/
  offset test passes with the retained outside focus owner, and the two standalone
  LazyList ablation cases confirm the distinction. No primitive algorithm changed.
- Root JVM 54 cases passed, including identity-not-equality retention and hidden
  binding withdrawal; original Docs Check out/latest/revert input assertions remain.
- Final native CLI actual pointer scrolling → other draft tab → original Agent tab
  gives identical full plain screens, retaining the older viewport and latest button.
  Clicking latest reaches the newest message and removes the button.
- Host/current Native/JVM matrices and all limitations are recorded by the
  [parent](2026-10-07-cutover-ui-foundations-and-home-bootstrap.md).
  No Aggregate/Timeline/resolver/compatibility API or extra Gradle project added.
  Independent scoped acceptance passed under conditional commit authorization;
  VM 42 / View 71 / Root 54 cases were freshly rerun. Local commits only.

- Owned source/test implementation is complete; no lane build/Gradle/IDE, commit,
  push, branch or shared-resource operation was performed. No Gradle, host, Docs,
  root cache or Patch API/package edits were made by this lane.
- Responded to the first central failures without weakening the original assertions:
  - Follow classification now uses the existing input source's **synchronous**
    committed-input callback, bound once per exact renderer mount with the latest
    displayed window. Paging does not restart a no-replay follow collector.
    The Unconfined scheduling override was removed.
    Disposal removes the exact listener and clears viewport retention; the retained
    ViewState has no business window/cache and no listener after unmount.
  - Tab round-trip compares settled snapshots and asserts exact anchor child,
    terminal offset, first-visible index and scroll offset.
  - Work Group still uses real pointer press/release at the measured row; assertions
    now inspect the settled release output (including actual Think children) and
    the actual Expanded state, not an earlier expanded-header-only draw.
  - Bounded-window wheel evidence likewise uses settled output; the original
    1,000-item round trip, peak-window bound and timeout assertions remain.
    Before the gesture, real frames are driven until the exact current provider
    has loaded visible message states and measured scrollable height. This fixes
    the coordinator's pre-payload zero-consumption readiness finding without
    manually changing follow intent or weakening the assertion.
- Original VM/window/item/factory and sparse/chunk/cache/release algorithms remain.
  Eviction consults the latest accepted exact visible-item report after structural
  reads, retains the original neighboring chunks, and ignores stale/foreign reports.
  Navigation retains one identity-distinct immutable effect; exact acknowledgment,
  nonce/revert/close withdrawal and dynamic UI prefix are implemented.
- Added `HistoryViewportAndScrollEffectTest` (repeat target/latest, unmounted slot,
  stale ack, nonce/revert/close, sparse visible retention and Work Group chunks) and
  `AgentHistoryRendererStateTest` (real pointer/keyboard/focus, late mount, dynamic
  stream/pending/newer prefix, tab lifetime and Work Group input/output).
  Existing paging readiness/identity tests report portable viewports; original
  forward/backward anchor and large-history assertions are preserved.
- Static review: scoped `git diff --check` is clean; History spec/VM source and tests
  contain no Compose/Mosaic/cli.components imports or `notifyContentChanged`.
  New tests and fixes have **not** been rerun by this lane. Coordinator's earlier
  42-case VM pass is historical evidence; the current full JVM/Native/Docs/CLI
  matrix and scoped acceptance are recorded in the coordinator closure above.

### Fixed integration API

- Exact signatures (source/test implementation complete; no lane builds):
  - Original `createAgentHistoryViewModel(source: AgentHistorySource,
    ownerScope: CoroutineScope, running: StateFlow<Boolean>): AgentHistoryViewModel`
    and Koin factory remain unchanged.
  - Original `historyItems`, `loadState`, `pendingTools`, `streamingItem`,
    `activeTurnDuration`, window/item interfaces and navigation commands remain.
  - `followsLatest: StateFlow<Boolean>` replaces the Compose Boolean.
  - `pendingScrollEffect: StateFlow<HistoryScrollEffect?>` is the single pending slot.
    `HistoryScrollEffect(generation: Long, target: HistoryScrollTarget)` has identity
    equality; `HistoryScrollTarget.Latest` / `HistoryScrollTarget.Item(item:
    HistoryItemViewModel)` use the exact materialized child, never a UI row index.
  - `reportViewport(window: HistoryItemWindow, visibleItems:
    List<HistoryItemViewModel>)`, `setFollowsLatest(window: HistoryItemWindow,
    followsLatest: Boolean)`, `acknowledgeScrollEffect(effect: HistoryScrollEffect)`.
    Reports/follow commands validate exact current window; ack validates exact effect.
  - Remove VM `listState`, `scrollInteractionSource`, `notifyContentChanged` outright.
  - Public renderer-local `io.github.stream29.kodex.cli.history.AgentHistoryViewState()`
    exposes `listState: LazyListState`, `scrollInteractionSource:
    MutableScrollInteractionSource` only; no model/window/cache fields.
  - `@Composable AgentHistoryView(model: AgentHistoryViewModel,
    shellSessions: AgentShellSessionRegistry,
    viewState: AgentHistoryViewState = remember(model) { AgentHistoryViewState() },
    onOpenEntryContextMenu: ((Long, Int, HistoryItemViewModel, TuiPopupAnchor,
    IntOffset?) -> Unit)? = null)`. Existing callback argument names unchanged.
- Coordinator: Root retains ViewState by **exact History VM identity**, passes through
  AgentRuntimeScreen; discard on actual binding replacement/close, not tab unmount.
  Docs/host fake models directly implement portable flow/report/follow/ack operations;
  scrolling fixtures own a ViewState and pass it into the renderer. Gradle retargeting
  and final centralized build/test/CLI evidence were completed by the coordinator.

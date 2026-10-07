# SCOPED REPAIR SEALED FOR E2E — full E2E acceptance pending

# Task Tree

- `Read actual History renderer and original failure evidence`()
- `Trace model publication, measured provider and one-shot demand gates`()
- `Publish deterministic actual-renderer fixture race gates`()
- `Repair confirmed fixture navigation boundary and statically review delta`()
- `Hand off centralized validation and independent review`()
- `Receive authorization to seal the scoped fixture repair with residual limits`()

# Details

## Coordinator validation of final bytes

- 作者下文的 SOURCE ONLY / PENDING 是历史 handoff，运行证据由协调者补记。
  最终两个测试的 manifest：
  `e6840e8e13625b268d7306caa561e298a3f6036769b44e3c83aa668d82cab1f1`。
  bounded 最终 SHA `19aa78e36084e3c3e8e44158cd2427f4ad5552f83424e17254598bed6681338a`；
  renderer SHA `bc05d5f75d007012b054a109b9a0e1f59ea33442bf89e1ba65078e2d2d91ad3b`。
  不使用作者早期 bounded 哈希覆盖中央消融后准确恢复的字节。
- payload/height 条件消融后新六条门闩全部红；恢复后完整 View 79 项通过。
  当前 continuous input 压力同源三次 fresh 各 1 项通过；最终 35 目标矩阵
  再运行完整 View 79 项通过，XML 总 1152 项零 failure/error/skipped。
- [最后两文件独立复审](../done/2026-10-07-recheck-history-continuation-tests.md)
  已完成并支持限定范围测试修复；不继承前次 547 审查覆盖本次修改。
  [主记录](2026-10-07-plan-spec-impl-closure-and-consistency.md#final-scoped-closure-evidence)
  保留两次旧压力失败、actual VM admission B2 与历史唯一根因的限制。
  生产 History View/VM 没有因本次 fixture 调查改动，原预算与断言保留。

- 用户授权调查、最小修复并复审；本线只拥有 `AgentHistoryView.kt`、专用 renderer
  测试、`AgentHistoryBoundedWindowTest.kt` 和本记录。History spec/VM、构建与其他线不改。
- 工作树为共享 dirty `BuildKodex/Kodex`，用户给定基线 `6b7129fa`；不提交、不切换。
  不运行 Gradle/runtime/IDE/设备/网络，不控制共享进程或资源。
- 原始证据：[本次日志](file:///tmp/kodex-closure-history-remainder-green.log#L1474) 是
  **FAILED**：History view 71 项中 1 项失败；VM 50 / RPC 85 通过为主线给定证据，
  不是本线新增执行。[失败 XML](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/impl/view/build/test-results/jvmTest/TEST-suite_io.github.stream29.kodex.cli.history.agentHistoryBoundedWindowTest.xml#L5)
  记录 Ready、newest 509、oldest 502、size 8、count 10、visible 7:503 到 2:508，
  older loop L117 的 `awaitWindowProgress` 超时。历史 flaky 风险不改写成通过。

## EARLY REDTESTREADY — historical source checkpoint

- [Renderer gate source](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/impl/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/history/AgentHistoryRendererStateTest.kt#L64)：
  `published older window cannot borrow old provider readiness before anchored edge input`；
  `published newer window cannot borrow old provider readiness before anchored edge input`。
- 实际 `AgentHistoryView` / `LazyListState`，真实 WheelUp 消费后冻结于下一帧之前。
  fixture 发布同 count、部分同 child 的新窗口；旧 measured provider 的所有 visible
  key 仍匹配旧 candidate。要求旧 candidate 不得借此授权新 model 的下一步导航；
  新 candidate 同样要等真实 provider key/index 匹配。恢复帧后 anchor key/offset
  保持，真实 list edge positioning 才产生窗口 demand；测试不调用 VM 分页命令。
- checkpoint 的 test-only helper 保留原等待器校验逻辑，所以旧 candidate readiness
  的 `assertFalse` **预期 red**。本线没有执行该预测；主线若未保留 red 控制，不得
  把静态断言或后续 green 称作实测 red。尚无生产 renderer 修复。

## Actual conditional call chain

- **Fixture publication gap：**原 `awaitWindowProgress` 在 Default 上 `combine` 独立的
  window、load state 与 Compose layout flow；Mosaic caller 同时 pump frame。组合值
  不是 owner/provider 的原子快照。候选旧 window 的 visible key/index 全部匹配时，
  owner `.value` 可能已经更新；原等待器返回 Boolean，caller 又读取更新后的 `.value`
  作为滚动索引及 progress baseline。它还可能在等待器返回后、frame pump 取消/join
  期间再次更新。这是可静态构造的 fixture 条件缺陷，不是已测生产分页失败。
- **真实 list 消费：**当前方法名是
  [`scrollToItem`](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/view/components/src/commonMain/kotlin/io/github/stream29/kodex/cli/components/LazyListState.kt#L148)，
  只设置 numeric Position；不携带 History window 或 provider 身份。
  [测量](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/view/components/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/components/LazyColumn.kt#L184)
  对当次 provider 解析/clamp 请求，随后
  [发布测量](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/view/components/src/commonMain/kotlin/io/github/stream29/kodex/cli/components/LazyListState.kt#L221)
  清空 request；后续 provider 则
  [按原 key/offset 恢复](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/view/components/src/commonMain/kotlin/io/github/stream29/kodex/cli/components/LazyListState.kt#L194)。
  key lookup 是现有 provider 的 nearest-key map，不重新读 model
  [provider source](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/view/components/src/commonMain/kotlin/io/github/stream29/kodex/cli/components/LazyListScope.kt#L134)。
- **与原失败相容，但不是唯一根因证明：**如果 request 被旧 provider 消费，锚点留在
  503，新窗口补入 502 后保留 503 key，502 可处于实际 viewport 上方；随后等待
  “比已经读取的 502 更旧”就可能超时。XML 的 visible 503..508 与此相容，但 XML
  没有记录前一 candidate/provider、请求消费帧或需求接受情况，不能认定这次恰是该时序。
- **实际 renderer demand：**
  [Ready 分支](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/history/AgentHistoryView.kt#L284)
  compose marker；[one-shot effect](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/history/AgentHistoryView.kt#L338)
  等准确 edge child 可见后，先 report 实际 visible children，再 request。
  compose/overscan 不等于可见；原失败的 502 不在 visible，单凭 Ready 不能要求这条
  effect 已触发。新窗口、marker remount 或 list 实例替换才会重启该 effect。
- **实际 VM admission：**
  [older/newer gates](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/cli/history/AgentHistoryViewModel.kt#L175)
  要求 exact current window、Ready、has-edge、pending CAS，并允许满 buffer 拒绝。
  [reportViewport](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/cli/history/AgentHistoryViewModel.kt#L216)
  拒绝过期窗口及 foreign child，不能把旧 layout 当新窗口 retention。
  当前 [loadOlder 尾部](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/cli/history/AgentHistoryViewModel.kt#L643)
  先 publish window，finally 清 pending，最后 Ready；并非 Ready 先发布而旧 pending
  尚未清除。这里的 VM/source 属主是主线，本线没有更改它。

## Final source delta

- [Bounded fixture L92–149](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/impl/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/history/AgentHistoryBoundedWindowTest.kt#L92)：
  等待器返回实际通过校验的 window；循环用同一捕获 window 作为进度比较基线，
  不再重新读取 latest `.value` 产生另一套地址/基线。older/newer 使用现有 list
  `requestScrollToEnd/Start`；这是原边缘定位意图，不是分页 retry 或 VM request。
  logical intent 仍可能在旧 provider 上被消费，因此**保留捕获 progress baseline
  同样必要**，不声称 logical intent 本身消除了所有并发。
- [Await/provider barrier L196–271](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/impl/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/history/AgentHistoryBoundedWindowTest.kt#L196)：
  保留 count、非空 stored rows、每行 exact child/index 和初始 payload/scrollability
  校验；增加当前 `.value === candidate` 和当前 Ready 的前后检查。
  helper 只在 `mosaicTest`；message-only、无 transient 的 fixture 专用，不是生产
  viewport controller、全 provider identity token 或原子 snapshot 锁。
- [Renderer gate L65–148](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/impl/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/history/AgentHistoryRendererStateTest.kt#L65)：
  两个方向都保留真实 WheelUp、anchor key/offset、obsolete-window rejection、
  current Loading 拒绝借用旧 Ready、同 count 不足以确认 provider、
  middle viewport 零 demand、真实渲染 edge 一次 demand、programmatic 不恢复 follow。
  复用现有 `RendererHistoryModel`/window fixture，仅让窗口暴露 demand 回调计数。
  **这两个测试是真实 renderer + 可控 contract fixture，不是实际 VM 的 pending/CAS
  实验**；原 1000-item 测试仍使用真实 VM/repository/Default payload 与并发 frame pump。
- 保留原 1000-item 双向到边界、initial <5s、每步 5s、300s navigation budget、
  360s hang guard、viewport-derived peak <=48、真实 pointer 退出 follow 与 stored
  message 输出断言。没有加 timeout、retry test、屏蔽 assertion 或手调 VM 分页。
- [Cleanup L53–184](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/impl/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/history/AgentHistoryBoundedWindowTest.kt#L53)：
  仓库打开/填充也进入 finally；持有并 cancel/join 原 model 专用 parent scope，
  然后 join repository；frame pump 在 finally cancel/join。NonCancellable 仅用于
  必要收尾。新 renderer gate 与 bounded block 在 finally `cancel()` Mosaic，因为
  [现有 runMosaicTest](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/Mosaic/mosaic-testing/src/commonMain/kotlin/com/jakewharton/mosaic/testing/TestMosaic.kt#L38)
  只在 block 成功返回后 cancel；未修改共享 Mosaic harness。
- 仅修改这两个测试和本报告；**`AgentHistoryView.kt` 没有本线增量**。
  无新增生产 API/owner/VM/controller/journal；spec、VM、Gradle、其他线和不可变
  snapshots/reports 未改。没有创建临时文件或运行 fixture，无待释放运行资源。

## R / B1 / B2 / U verdict

- **R — 静态支持的修复：**fixture 不再把已验证旧 candidate 与随后新 model 的数字
  地址/进度基线混用；当前 owner/window/Ready 与 measured rows 的屏障更严格。
  新确定性门闩与旧 invariants 均已交接；这不是运行验收。
- **B1 — 已确认的条件缺陷：**旧等待器/caller 的非原子交接允许上述错配；
  分类为测试 fixture defect。本线没有确认新的生产 B1，也不把 original failure
  的原因认定为已解决。
- **B2 — 仍待区分的生产疑点：**one-shot demand 若因实际 current-window/Ready/
  pending/full-buffer gate 被拒绝，而相同 composition 未重启，可能失去后续边缘需求。
  需要在实际 VM/renderer 下记录 exact request 的接受/拒绝及之后 remount/新窗口，
  不能仅让假 model 计数证明该生产问题，也不能用 VM 测试绿色否定它。
  Default payload 高度改变与真实 pointer/key anchor 同样仍由实际 view 回归判别；
  fixture 修正不准 renderer 在新窗口发布时强制重定位或改变原 visibleChunks 淘汰。
- **U — 主线门槛：**本线没有编译/执行 red 或 green。最新实际整轮仍为
  **71 view / 1 failure；command FAILED**。VM 50 / RPC 85 的通过不覆盖该压力失败。
  Native/真实 CLI/性能、全部闭合与新独立增量审查均未验证。

## Central handoff — author lane finished, execution pending

- 模块 `:app-component-history-impl-view:jvmTest`，先运行上述
  `agentHistoryRendererStateTest` 两个新 names，再运行
  `agentHistoryBoundedWindowTest / long History navigation keeps only a viewport-derived local window`。
  单次 green 不作 flaky/root-cause 闭合；保留每次真实失败及对应 source。
- 保留 `agentHistoryAnchorTest` 的四个 late-provider/offset 组合，以及现有
  `agentHistoryRendererStateTest` 的 pointer/keyboard、focus relocation、late mount/exact
  ack、transient prefix 和 tab remount controls；随后完整 view suite。
- 主线实际 VM 的 Ready/pending/CAS、nonce/completion、visibleChunks 与 RPC controls
  不由本线重写。若原 bounded gate 仍失败，先区分请求前 capture、消费 provider/
  measured edge、payload height、真实 request admission；不提高 5s 或制造 VM 分页。
- 对这两个测试与本记录做独立 delta review；不改旧审查结论。
  scoped `git diff --check` 静态检查通过；这不代表 Kotlin compilation 或 tests 通过。
  本线完成源码/文档交接，不等待中央构建，不提交/推送/切换。

## Follow-up — EARLY REDTESTREADY, central execution PENDING

- 用户给定新证据：完整 History View 73 green 后，相同字节两次 fresh pressure，
  第一次 green、第二次 [pressure-2](file:///tmp/kodex-closure-history-pressure-2.log#L975)
  FAILED；Ready、422→415、size 8、count 10、visible 416..421，
  older loop 等进度。前次 capture/provider 修复不是完整 flaky 根因闭合。
- 本次仅拥有同两个 mosaicTest 和本节追加；547 独立审查冻结的前次两文件。
  旧报告/快照不编辑。主线唯一执行构建，本线只读真实生产调用链。
- 先加入 bounded timeout 诊断：窗口每个 child 的 state / lazy Job 状态，
  visible offset/height、数值 anchor、viewport、scrollability、follow/pending effect，
  最后 16 个 frame 检查点。仅 peek，无 payload activation。
  私有 list request 与 VM pending/CAS/buffer 不从 public API 可读，明确标为 unavailable，
  不用反射或复制 VM authority。
- 新门闩计划（实际 AgentHistoryView / LazyColumn，不是模型分页测试）：
  `delayed older payload preserves the consumed edge anchor until real pointer input`；
  `delayed newer payload preserves the consumed edge anchor until real pointer input`；
  同两方向的 `real keyboard input` 版本。
  保持 payload lazy Job 未完成，先 consume logical End/Start；记录 get 激活、
  Loading edge 无 visible height、归一化 anchor。解除 payload 后保留真实 anchor，
  不断言 programmatic intent 必须持续 pin edge。然后真实 wheel/Page key 才恢复
  positive-height edge visibility 与一次 demand，检查 consumed input 分类。
- RED 控制计划：新增 fixture target-payload + measured-positive-height 屏障若只保留
  count/key/current-window，则 Loading 阶段或 Ready 尚未测量阶段的拒绝断言应 red。
  这是静态预期，不冒充执行结果，也不宣称 415 此次必然走了该路径。
- **源码实现中；central compilation / red / gated green / pressure / delta audit PENDING。**

## Follow-up — SOURCE ONLY READY; central tests / delta audit PENDING

### Follow-up Task Tree

- `Read pressure-2, current fixtures and actual activation/measure/anchor sources`()
- `Add failure observations before changing fixture input`()
- `Add deterministic blank-row and late-payload physical-input gates`()
- `Repair continuous-input fixture and review source delta`()
- `Hand off source-only evidence to the sole central runner`()

### Static proof: correct the zero-height premise

- 真实 message 状态名是 **Loading**，不是 Initializing；后者是结构 load state。
  [Message VM L21–50](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/cli/history/MessageHistoryItemViewModel.kt#L21)
  建立 LAZY Job，只有 ensureLoaded 启动；window
  [get L1015](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/cli/history/AgentHistoryViewModel.kt#L1015)
  启动同一 child；peek 不激活。实际 View 的 indexed item content 调 get。
- [Loading renderer L540–545](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/history/AgentHistoryView.kt#L540)
  用 `Text("")`；[Mosaic TextLayout L36–54](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/Mosaic/mosaic-runtime/src/commonMain/kotlin/com/jakewharton/mosaic/text/TextLayout.kt#L36)
  按 `split("\n").size` 计算高度，空字符串为 **1 行**。
  无内容 demand marker 才是零高。不能把零高 marker 的归一化当 Loading message
  的零高，也不能由旧 XML 推定 payload/measure 时序。
- [真实 measure L177–248](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/view/components/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/components/LazyColumn.kt#L177)
  对消费请求的 provider 解析 logical Start/End，越过零高 marker，
  正高 Loading row 可以进入 visible 并发出 demand，**不需要先 Ready**。
  [anchorAt L310](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/view/components/src/commonMain/kotlin/io/github/stream29/kodex/cli/components/LazyListState.kt#L310)
  只选择正高 child；[publish/restore L194–243](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/view/components/src/commonMain/kotlin/io/github/stream29/kodex/cli/components/LazyListState.kt#L194)
  清 request 并保存准确 key/offset。晚到窗口/高度恢复 key，不续订旧 logical intent。
- 因而“同一 provider 的 Loading edge 被 End 消费却因零高跳过到 416”不符合源码。
  仍可构造的不同路径：End 已在包含 416 的 provider 消费，随后发布 415，
  当前 anchor 416 被忠实恢复；415 的 blank/Ready 高度都在 viewport 上方。
  [one-shot effect L338–354](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/history/AgentHistoryView.kt#L338)
  等准确 edge 可见，overscan get 与 Ready 都不代替 visibility。
  该路径与 pressure-2 相容，**不是这次唯一根因的经验证明**。

### Six actual-renderer gates

- [两方向 blank-row gates L68–160](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/impl/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/history/AgentHistoryRendererStateTest.kt#L68)：
  `delayed older payload has a real blank row when logical edge is consumed`；
  `delayed newer payload has a real blank row when logical edge is consumed`。
  actual renderer get 启动 gated LAZY Job；consume logical edge 后断言 Loading row
  size=1 且 demand=1。完成 payload、尚未 pump frame 时，Ready 仍不能借 blank
  geometry；下一测量 size=2，anchor key/offset 保留，one-shot 不因高度重发。
- [四个 continuation gates L161–298](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/impl/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/history/AgentHistoryRendererStateTest.kt#L161)：
  上节列出的 older/newer × real pointer/keyboard 名称全部已实现。
  old provider 消费 logical intent **后**才发布含 delayed edge 的 replacement。
  overscan get 激活 Job，Loading 和晚到 Ready 均不可见，anchor key/offset 不变，
  demand=0；既有 count/key 屏障甚至 loaded-visible-row 屏障都可通过。
  正确的 target barrier 在 Loading、Ready-before-frame、Ready-offscreen 均拒绝。
- 后续真实 WheelUp/Down 或 PageUp/Down 消费 movement，target size=2 可见且
  demand 恰一次；键盘通过真实点击 fully-visible neighbour 获取 focus，单独确认
  focus acquisition 没有产生 edge demand/改 anchor，再发送 Page key。
  有新的 input-source/consumed-sign 断言，不靠 enqueue 或 snapshot 文本冒充输入。
- 复用原 RendererWindow/Message；仅 get 启动已有 lazy Job、记录 accessed child。
  没有在测试 body 调 get/start/VM paging 强制加载，peek diagnostics 不激活。
  model 是可控 contract fixture，不复制真实 VM 队列/状态或框架布局；
  demand counter **不是实际 VM pending/CAS/full-buffer admission 证据**。
  Job/gate、interaction collector 与 Mosaic 都在 finally cancel/join 相应所有者。

### Pressure fixture delta and limitations

- [bounded L116–155](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/impl/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/history/AgentHistoryBoundedWindowTest.kt#L116)
  用持续真实 WheelUp/Down 驱动每个原 5s progress wait；捕获同一 window 比较方向
  进度，仍不另读 latest 作为地址/基线。不是重复执行测试或 VM 分页 retry：
  一个未完成的用户连续滚动操作会继续发物理输入，而单次已消费的 logical intent
  没有必须 pin 后续 UI 的契约。新独立 gates 保留旧 programmatic 行为控制。
- [readiness L312–355](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/impl/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/history/AgentHistoryBoundedWindowTest.kt#L312)
  每个 pressure handoff 都要求可见 message Ready 且测得完整 2 行，保留原
  current window/Ready 前后校验与 count/key/index、初始 scrollability。
  2 行只适用于本 fixture 的单行 User header+content，**不是通用最终高度协议**。
  新可选 inputTarget+expectedTargetHeight 用于受控门闩断言，生产未使用。
- 不增加 offscreen-edge Ready 的被动 await：只有测量/get 才可能激活它。
  受控门闩已把“Ready-before-measure”“Ready-offscreen”和 actual physical
  continuation 分开，不能只凭 visible rows Ready 或 provider identity 判断目标就绪。
  真实 continuous pressure 与独立 geometry gates 都须中央执行才有运行证据。
- [诊断 L201–308](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/impl/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/history/AgentHistoryBoundedWindowTest.kt#L201)
  保留最多 16 个变化的 frame checkpoint，含 captured baseline/方向/input button、
  实际 interaction requested/consumed/source、每个窗口 child Loading Job 状态、
  visible offsets/heights、anchor、scroll flags、follow 与 pending effect。
  当前公开 API 不暴露 list pending request 或 VM request admission，明确 unavailable。
  这些是非原子观察，不是新 window/provider authority 或可靠全量事件 journal。
- 原 1000 items 双向 storage 边界、12 行 viewport、peak≤48、initial<5s、
  per-wait 5s、总 300s、360s hang guard、初次真实 pointer 退出 follow/output
  断言与必要 cleanup 均保留。没有增加 timeout、吞 assertion 或重新运行失败测试。
  peak 仍是检查点采样，不把它升级宣称为全时段峰值监控。
- 没有发现需本线擅改的确认生产 View/VM 错误；生产源码/契约/anchor 行为全部未改。
  实际 admission rejection 后同 mount 不重启的既有 B2 仍未关闭；若真实新门闩
  或 pressure 证明该生产路径错误，主线应保留 exact evidence 再单独取得修改授权。

### Central handoff / ablation

- **SOURCE ONLY READY；central compilation / red / green / pressure / independent
  delta audit 全部 PENDING。** 本线完成静态源码核对，不等待中央运行。
- 在 `:app-component-history-impl-view:jvmTest` 集中运行上述新增 **6** names，
  再保留前次两条 publication gates、anchor 四组合及完整既有 renderer/View controls。
  不继承给定 73 green 覆盖本次新字节；单次 pressure green 也不关闭 flaky root。
- RED 控制仅消融新 `requireReadyRows` 与 inputTarget/height 条件，保留新 tests：
  Loading、Ready-before-measure 拒绝断言预期失败。这个 red 证明 helper 新责任，
  不证明 VM admission bug 或唯一压力根因；本线没有执行 red。
- 另独立核对四个 continuation 的 old-consumed → replacement → late-height →
  physical input 条件；去掉物理输入应无法通过 target geometry / demand=1 阳性。
  不把 counter=0 的合法 invisible edge 当 production red；不要求 programmatic
  Start/End 在之后新窗口或 payload 下继续 pin。
- 中央 fresh pressure、失败诊断与完整 suite 记录 actual hash/log/XML/count；
  旧 logical-pressure 路径/547 的冻结审查仍是独立历史证据，不修改或冒充覆盖。
  本次 delta 请另做独立审查，特别核对 continuous input 对预算和 retention 的影响。
- 若主线需要 **diagnostic-only 原路径 control**，只恢复 547 冻结两文件中的
  bounded older/newer logical 请求及等待器的 initial-only readiness 调用，保留本次
  observations/interaction diagnostics；不恢复旧 Boolean/caller 重读缺陷。
  对照运行必须单独记录 source/hash，不将新 physical-pressure green 当旧路径 green。
  新门闩继续存在；不在生产或测试 API 添加两套运行模式/重试开关。
- Scoped source `git diff --check` 已通过；未编译/运行，无 Gradle、IDE、网络、
  设备/共享进程控制或 git commit/switch/push。仅两个测试和本任务追加有增量。
  没有创建临时文件或占用待释放运行资源。
- 本次源码 SHA-256：
  - bounded：`1f3fdcb23103f5bea2c1435422d901c7125d4dae9d2472f3c2c89801373a1789`。
  - renderer：`bc05d5f75d007012b054a109b9a0e1f59ea33442bf89e1ba65078e2d2d91ad3b`。

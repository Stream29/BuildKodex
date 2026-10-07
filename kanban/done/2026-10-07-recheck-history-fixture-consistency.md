# REVIEW READY — two-file static re-review; runtime acceptance pending

- 审查工作已归档；历史两文件和当时给定运行结果不回写为当前测试字节。
  当前中央证据见 [主任务](2026-10-07-plan-spec-impl-closure-and-consistency.md#final-scoped-closure-evidence)。

# Task Tree

- `Independently compare the two frozen renderer test deltas`()
- `Check readiness, edge input, progress assertions and resource teardown`()
- `Report confirmed fixture repair separately from unproven production root cause`()

# Details

- 独立 reviewer，未参与实现；用户授权修复后复审，本线只编辑此报告。
- 已完成固定字节比较、实际调用链和最小消融审查。**R：支持 fixture 源码修复；
  未发现这两文件新增的确认 B1。原条件 fixture 缺陷与压力失败唯一根因分开。**
- 最新给定整轮仍为 **71 View / 1 failure，command FAILED**：Ready，窗口
  509→502，可见 508→503。VM 50 / RPC 85 通过为主线报告，不是本线执行；
  也不证明该旧压力风险被 completion 修复解决。
- [作者 SOURCE READY 记录](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-07-investigate-history-bounded-navigation-stall.md#L1)
  和[主线验收计划](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-07-plan-spec-impl-closure-and-consistency.md#L1)
  已读；作者的 red 预测不是实测 red，源码 ready 不是经验根因闭合。

## Fixed evidence and original-path mapping

- 两个 manifest 的 SHA-256 均实读核验匹配，且 `sha256sum --check --quiet`
  核验 **2/2 当前文件、1292/1292 第三快照文件**，无 mismatch。
  当前 manifest：
  `784e4eddacc6879a6c19a1729012cdb1e51fea934d5fd27f9625c8a4f4234be6`；
  第三快照 manifest：
  `fff68f99f942a9bf201e26cf3ce852718a4df29f26e33757d3352cbdd7ec0c1e`。
- [当前 manifest](file:///tmp/kodex-closure-history-fixture-review-20261007/REVIEW_SOURCE_MANIFEST.json#L1)；
  [before/生产 manifest](file:///tmp/kodex-closure-remainder-review-20261007/REVIEW_SOURCE_MANIFEST.json#L1)。
  第三快照两测试均实际存在，并用于 `diff -u`，不是用 Git 基线冒充 before。

| 文件与原路径导航（非工作树字节证据） | 本次依据与行号 | 当前 / before SHA-256 |
| --- | --- | --- |
| [AgentHistoryBoundedWindowTest.kt](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/impl/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/history/AgentHistoryBoundedWindowTest.kt#L92) | [fixed L92–184、196–271][bounded]；[before L107–161、173–233][before-bounded] | `a3b1ba5d97882463c68be98c843c5b70dc0a8099f754adfde761980df69630a3` / `52b6a2ec9fde95b763464a95c51b1754581c7e51e8696e0ab168f0cd04c929f9` |
| [AgentHistoryRendererStateTest.kt](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/impl/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/history/AgentHistoryRendererStateTest.kt#L65) | [fixed L63–148、413–425、455–498][renderer]；[before L62、325–412][before-renderer] | `309f342c26c28acdc4eb928aee2a3adb7dcc02bdaa79d4c5cd5c4d015ebe6347` / `3341c5a3c513836889f2eb77a29b628df407b5fdd6ea1697eef9250e1b387c67` |

- 后文测试行号均对应当前两文件 fixed bytes；生产行号均对应第三快照。
  对应原 `Kodex/` 路径为快照相对路径原样加该前缀，未读其变化中源码。
  原路径导航：
  [View L338](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/history/AgentHistoryView.kt#L338)、
  [VM L175](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/cli/history/AgentHistoryViewModel.kt#L175)、
  [spec L46](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/history/contract/AgentHistoryViewModel.kt#L46)、
  [list L148](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/view/components/src/commonMain/kotlin/io/github/stream29/kodex/cli/components/LazyListState.kt#L148)；
  这些行的生产判断依据是后文链接的第三快照同路径/同行字节。
- 共享 dirty 工作树基线 `6b7129fa` 只作历史背景；本次生产上下文为第三快照，
  不声称 manifest 覆盖外部子模块。加载了 change/kanban/planning/ask-user/
  document workflows，以及 frontend、lifecycle、spec/impl 规范和实际 History spec。

## R / B1 — confirmed conditional fixture defect, not unique failure causality

- [before waiter L186–214][before-waiter] 组合独立 window、loadState、Compose layout，
  只返回 Boolean。可构造条件：组合中的旧 window 与旧 provider 全部 visible
  child/index/count 匹配，但 owner `.value` 已前进；caller 随后在 L113–115 /
  L133–134 重新读取 latest window 的 size/edge 作为 numeric 地址和 progress baseline。
  等待返回后的 frame-pump cancel/join 也允许 owner 继续前进。
- [修后 waiter L196–271][waiter] 在校验前后检查
  `model.historyItems.value === candidate` 和当前 Ready；保留组合 Ready、marker
  count、非空 stored rows、每行 exact child/index，初始 `requireScrollable` 时仍
  要求真实可滚动高度和所有可见 message payload Ready。返回的是通过检查的
  **exact window**，不是 Boolean；L92 / L113 / L127 / L146 接住它。
- [循环 L115–152][loops] 使用同一 captured window 比较方向进度，分别发出现有
  list `requestScrollToEnd/Start`，不再由新 `.value` 的 size 计算旧 provider 地址。
  latest `.value` 仅留在超时/预算诊断，不用作另一套 progress baseline。
- [实际 list L148–168、194–241][list]：numeric Position 与 logical Start/End 都是
  下一测量请求；[实际 measure L177–193][measure] 对**消费请求的 provider**解析，
  publishMeasureResult 清掉 request，再按原 key/offset 恢复 anchor。
  logical edge 可以仍被旧 provider 消费，所以保留 captured baseline 是独立必要项。
- 这确认的是**旧 fixture 允许错配的条件缺陷**，修正方向静态成立。前后检查不是
  原子锁，也不保证返回后 owner 不变；exact capture 和 logical intent 避免的是
  跨快照地址/基线混用，不是消灭所有并发或保证每次一步一页。
- 509→502 / visible 508→503 与“请求被前一 provider 消费后保留 503 anchor”
  相容，但原失败未记录 candidate、消费帧或 VM admission。**不能据此认定
  此次 timeout 恰由该条件引起，更不能确认唯一 1000-item 根因已经解决。**
  此 test-only delta 没有生产 View/VM/source 修复，不能归为新的 completion bug。

## R — real renderer gates and retained pressure invariants

- [新两方向 gate L65–148][renderer] 挂载实际 AgentHistoryView/LazyList：
  两次真实 WheelUp 后断言退出 follow，捕获实际 visible anchor key/offset。
  在不 pump 下一帧的区间发布同 count、部分共享 child 的 replacement；
  L113 要求 layout 仍为同实例，L114 拒绝旧 candidate，L117 拒绝尚未测量的新
  candidate。新窗口的 shared child 位置整体 ±1，因此同 count 不足以通过 exact
  index/key 屏障。
- L95–99 是**可控 contract owner 的 Loading guard**，不是实际 VM pending 实验。
  恢复帧后 L123 要求新 candidate 真正 measured；L127–130 要求 anchor key/offset
  不变、旧方向 demand 不增加、当前中部 viewport 零 demand。L134–140 通过真实
  list edge positioning 使 exact edge child 可见，方向回调恰一次，且不恢复 follow。
  没有手调 requestOlder/Newer、setFollowsLatest、手造 layout 或仅 assertToString。
- [RendererWindow L413–425 / RendererHistoryModel L455–498][fixture] 的计数回调
  是合法的 renderer→contract 边界观察。它不模拟真实 Ready/current/pending CAS/
  full-buffer admission，`get()` 的 payload 已 Ready；一次 callback **不等于
  实际 VM 接受、完成一页**。这些 gate 也没有控制 Default publication 的完整
  combine/caller handoff；该 handoff 的修复另由静态数据流及完整压力用例判别。
- [原压力 L43–52、82–161][pressure] 保留 1000 items、12-row viewport、双向走到
  hasOlder/hasNewer 边界、初始实际 elapsed <5s、每次等待 5s、总 navigation
  300s、testScope 360s hang guard、peak ≤48。**300s 是总导航预算，不是每步预算**；
  没有提高任何原预算。真实 pointer 退出 follow、输出不含 999 / 含 User 和
  最终 elapsed 断言保留，未新增直接 VM 分页、测试重试或忽略失败。
- 原 renderer tests 除 fixture 私有参数/回调外未改；新增参数有保持旧行为的默认值。
  既有 pointer/keyboard、focus relocation、late mount/exact ack、tab remount、
  transient 和 work-group controls 不被新 fake gates 取代。

## R — failure teardown and cancellation is not assertion suppression

- [bounded L53–68、173–184][ownership] 在 try 内 open/fill/create model，保存
  dedicated modelOwner。正常、setup failure、断言失败或取消均先 close model，
  再在 NonCancellable 中 cancel/join 该 owner，finally cancel/join repository。
  [真实工厂 L920–932 / close L276–282][model-owner] 使用传入 dedicated scope；
  [repository L173–205][repository] 拥有自己的 supervisor child，Session/runtime
  归其结构化子树。不是只 cancel VM 留一个正常返回时不会自行结束的 parent Job。
- [waiter L201–245][waiter] 的 frame pump 是 coroutineScope child，finally cancel，
  NonCancellable join；throw 路径不会跳过 pump 等待。必要收尾才屏蔽取消。
- bounded L168–171、新 gate L141–146 均 finally `cancel()` Mosaic；fake model
  close 无独立 Job。[共享 runMosaicTest L38–45][mosaic-test] 原来仅在 block
  **成功返回后** cancel，所以本地 finally 是必要的失败路径修补，共享 harness 未改。
- 已核对 cancel receiver：TestMosaic L125 转发给 Mosaic；
  [Mosaic L152–154、477–482][mosaic-runtime] 建立 `Job(parent)`，cancel 的是该
  renderer child/recomposer，不是 TestBalloon current TestScope 或 caller scope。
  finally 无 return、catch AssertionError 或空成功替代；一次成功的重复 cancel
  不改变 assertion 的传播。因此未发现“显式 cancel 令失败假绿”的源码机制。
  `cancel()` 本身不 join；renderer 完成由其结构化 caller parent 在离开
  coroutineScope / runTest 时等待，不把 finally cancel 单独宣称为同步完成。
- 核对的是 **catalog 对应的缓存 sources.jar**，不是把本地 testBalloon
  `1.0.2-dev-1` 当实际 `1.0.1-K2.4.0`：
  [Test L80–105][balloon-test] 经 currentCoroutineContext 运行 action；
  JVM runTestAwaitingCompletion L30–34 直接调用 runTest(timeout)；
  [TestElement L412–418][balloon-report] 捕获 Throwable 并 registerFailure。
  [coroutines-test 1.11.0 TestBuilders L337–372、564–573][coroutines-test]
  withTimeout 取消 **test scope** 后仍 join、收集 completion/uncaught failures 并
  throwAll；这里的 `(scope as Job).cancel` 不等于测试内 Mosaic child cancel，
  也没有把 AssertionError 自动当通过。source 阅读不是故障注入执行。
- Mosaic 外部子模块只读静态 library，两个源 SHA：
  TestMosaic `f4437c369490972ce25887989e5ebbd2b4d3e1b668ab5ba62700028d1357d13b`；
  mosaic runtime `a8849fb72eec34f8f47e9c309dfbd43e0220f15e0e1171cd97a667f355fa7c51`。
  精确 TestBalloon/coroutines-test sources.jar SHA：
  `62455c90e7ff2513ebe13cb4a4c9b7d5d0ab58b5a4ee07de6b1de462fda44ce7` /
  `7159352cfc943f007fcc0003d986656538644cac3dcd8af1c9c775ca0f73f87c`。

## B2 — concrete actual-demand rejection / missing-remount gate remains

- [生产 renderer L236–293、338–354][view] 只在 Ready compose edge marker；
  one-shot LaunchedEffect 等 exact edge child visible，report 实际 children 后 request。
  overscan composition 不等于 visible。502 不在原失败 viewport，仅有 Ready
  不证明该 effect 已发请求；同 window/edge/list 的 effect 不自动循环重试。
- [真实 VM L175–201][admission] 验证 current window、Ready、edge、pending CAS；
  trySend full-buffer 失败清 pending 并拒绝。L216–223 拒绝 stale/foreign viewport。
  L644–653 / L695–704 顺序为 publish window→finally 清 pending→Ready，不能
  把“Ready 总先于旧 pending 清除”当这份生产源码的确认故障。
  [规范 requestOlder/Newer L46–62][spec] 允许拒绝，无 retained paging/retry。
- 所需实际 gate：保持真实 VM + mounted renderer，记录 exact window/edge、
  visible layout、请求时 current/Ready/pending/buffer 和接受/拒绝；
  用实际 command/read 门闩控制一次拒绝（full buffer 或明确 Ready/pending 时序），
  不靠 fake callback 宣称 reject。解除门闩后保持相同 window/marker/list mount，
  核验是否仍停住；再用真实离开/返回 edge 或 marker remount 作阳性控制，
  验证后续 demand 是否被接受并实际推进。
- 若控制期间 publication/remount 已发生，不满足“拒绝后无重启”条件，不能
  把该运行当此 B2 复现。无拒绝控制的实际 admission/推进阳性也必须保留。
  该疑点是 renderer demand delivery 与规范允许拒绝之间的具体缺口，
  **尚非确认生产 B1，也不是本次 fake 门闩证明的行为**。

## Minimal ablation / D / U

| 最小消融 | 会失去的实际责任 |
| --- | --- |
| 去掉 current identity / live Ready 前后检查 | 旧 candidate 或 lagging Ready 可借旧 measured rows 授权；双检查缩窄校验中 publication，不构成锁 |
| 只留 count，删非空 rows / child identity / index / 初始 payload+scrollability | 相同 count 的旧 provider、foreign rows 或 blank-height wheel 零消费可被误认 ready |
| 返回 Boolean 或 caller 重读 `.value` | 再次丢失测量对应的 progress baseline |
| 仅捕获 baseline，恢复新 model size 的 numeric index | 请求地址仍属于另一 provider；logical edge 的消费责任丢失 |
| 删真实 WheelUp / anchor / middle zero / exact edge once 断言 | 只剩 helper 自证，失去真实输入、稳锚和 visible-edge contract 的边界回归 |
| 删 renderer finally 或 owner/pump join | 断言失败可绕过共享 harness cancel，或离开测试时未完成结构化收尾 |

- **D（证据能力限制，不是放宽许可）：**helper 仅适用无 transient 的 message
  fixture，不是通用 provider token；peak 仍为完成测量检查点的采样峰值，不是
  全时段每次 publication 的资源监测。fake gate 的 payload 高度固定，实际 Default
  payload/物理输入/anchor 时序仍须原压力与 anchor suite 判别。
- NonCancellable joins 没有各自新 deadline；360s runTest guard 不能强行终止
  不合作的 shielded cleanup。本次对象为已取消的结构化 in-memory/test Jobs，
  未见具体不退出路径；不声称通过了 cleanup fault/deadline 运行矩阵。
- **U：**未编译/运行。新增 imports 均有实际用途，未见新增 unused import；
  [现行 convention L13–18][compiler] 启用 Werror，静态检查不能替代 Kotlin/Compose
  编译。主线安排在 CLI link 后集中编译/执行，本线不另起 build。
- 中央证据分三份记录：①旧 readiness 逻辑 + 新 gate 的 deterministic **red**
  控制（只回退相应 helper 责任，不能只使用不含新 tests 的 before 文件）；
  ②固定两文件的新 gate **green**，同时保留 anchor 四个 late-provider/offset
  controls 与既有 renderer controls；③原完整双向 1000-item **pressure** 与完整
  View suite。gate green、单次 pressure green、源码 ready 都不是唯一根因证明。
- 报告每次实际 source hash、测试选择/count、日志/XML、失败和成功分开。
  VM/RPC、completion gates、真实 CLI/Native/其他平台和全仓 203 项目的闭合属于
  主线独立门槛，**本报告不作 blanket pass**，不继承旧运行通过覆盖新字节。
- 本线只写本报告（apply_patch）；未修改源码/其他文档/快照，未运行 Gradle、
  runtime UI、网络/设备/IDE 或控制共享进程/资源，未提交/推送/切换。
  未创建临时文件或运行 fixture，无本线待释放资源；共享快照保留给主线使用。

[bounded]: file:///tmp/kodex-closure-history-fixture-review-20261007/app/component/history/impl/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/history/AgentHistoryBoundedWindowTest.kt#L92
[before-bounded]: file:///tmp/kodex-closure-remainder-review-20261007/app/component/history/impl/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/history/AgentHistoryBoundedWindowTest.kt#L107
[renderer]: file:///tmp/kodex-closure-history-fixture-review-20261007/app/component/history/impl/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/history/AgentHistoryRendererStateTest.kt#L65
[before-renderer]: file:///tmp/kodex-closure-remainder-review-20261007/app/component/history/impl/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/history/AgentHistoryRendererStateTest.kt#L62
[before-waiter]: file:///tmp/kodex-closure-remainder-review-20261007/app/component/history/impl/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/history/AgentHistoryBoundedWindowTest.kt#L186
[waiter]: file:///tmp/kodex-closure-history-fixture-review-20261007/app/component/history/impl/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/history/AgentHistoryBoundedWindowTest.kt#L196
[loops]: file:///tmp/kodex-closure-history-fixture-review-20261007/app/component/history/impl/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/history/AgentHistoryBoundedWindowTest.kt#L115
[list]: file:///tmp/kodex-closure-remainder-review-20261007/app/view/components/src/commonMain/kotlin/io/github/stream29/kodex/cli/components/LazyListState.kt#L148
[measure]: file:///tmp/kodex-closure-remainder-review-20261007/app/view/components/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/components/LazyColumn.kt#L177
[fixture]: file:///tmp/kodex-closure-history-fixture-review-20261007/app/component/history/impl/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/history/AgentHistoryRendererStateTest.kt#L413
[pressure]: file:///tmp/kodex-closure-history-fixture-review-20261007/app/component/history/impl/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/history/AgentHistoryBoundedWindowTest.kt#L43
[ownership]: file:///tmp/kodex-closure-history-fixture-review-20261007/app/component/history/impl/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/history/AgentHistoryBoundedWindowTest.kt#L53
[model-owner]: file:///tmp/kodex-closure-remainder-review-20261007/app/component/history/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/cli/history/AgentHistoryViewModel.kt#L920
[repository]: file:///tmp/kodex-closure-remainder-review-20261007/agent-session/impl/in-memory/src/commonMain/kotlin/io/github/stream29/kodex/agentsession/inmemory/InMemoryKodexSessionRepository.kt#L173
[mosaic-test]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/Mosaic/mosaic-testing/src/commonMain/kotlin/com/jakewharton/mosaic/testing/TestMosaic.kt#L38
[mosaic-runtime]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/Mosaic/mosaic-runtime/src/commonMain/kotlin/com/jakewharton/mosaic/mosaic.kt#L152
[balloon-test]: jar:file:///home/stream/.gradle/caches/modules-2/files-2.1/de.infix.testBalloon/testBalloon-framework-core-jvm/1.0.1-K2.4.0/8504c37134cf81dd583ce05e212e0ceb9a73c92f/testBalloon-framework-core-jvm-1.0.1-K2.4.0-sources.jar!/commonMain/de/infix/testBalloon/framework/core/Test.kt#L80
[balloon-report]: jar:file:///home/stream/.gradle/caches/modules-2/files-2.1/de.infix.testBalloon/testBalloon-framework-core-jvm/1.0.1-K2.4.0/8504c37134cf81dd583ce05e212e0ceb9a73c92f/testBalloon-framework-core-jvm-1.0.1-K2.4.0-sources.jar!/commonMain/de/infix/testBalloon/framework/core/TestElement.kt#L412
[coroutines-test]: jar:file:///home/stream/.gradle/caches/modules-2/files-2.1/org.jetbrains.kotlinx/kotlinx-coroutines-test-jvm/1.11.0/4fb14a79175c320cf05cc31e8e573d9f2a253343/kotlinx-coroutines-test-jvm-1.11.0-sources.jar!/commonMain/TestBuilders.kt#L337
[view]: file:///tmp/kodex-closure-remainder-review-20261007/app/component/history/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/history/AgentHistoryView.kt#L338
[admission]: file:///tmp/kodex-closure-remainder-review-20261007/app/component/history/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/cli/history/AgentHistoryViewModel.kt#L175
[spec]: file:///tmp/kodex-closure-remainder-review-20261007/app/component/history/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/history/contract/AgentHistoryViewModel.kt#L46
[compiler]: file:///tmp/kodex-closure-remainder-review-20261007/buildSrc/src/main/kotlin/KodexHostKmp.kt#L13

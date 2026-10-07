# REVIEW READY — independent final two-file review; scoped R, residual B2/U

- 独立审查已完成并归档；当前运行和限定范围交付见
  [主任务](2026-10-07-plan-spec-impl-closure-and-consistency.md#final-scoped-closure-evidence)。

# Task Tree

- `Independently review final two-file continuation test deltas`()
- `Verify real input, payload geometry and retained assertions`()
- `Separate confirmed test repair from residual admission and root-cause limits`()

# Details

- 用户授权修复后复审；本线只编辑本报告，不实施或执行运行验证。
- [调查 handoff](2026-10-07-investigate-history-bounded-navigation-stall.md)；
  [统一证据](2026-10-07-plan-spec-impl-closure-and-consistency.md)。
- 当前两文件快照 `/tmp/kodex-closure-history-continuation-review-20261007`；
  manifest SHA-256
  `e6840e8e13625b268d7306caa561e298a3f6036769b44e3c83aa668d82cab1f1`。
  Before 是前次两文件快照；生产上下文是第三个 1292 文件快照。
- “Loading message 零高”假设被实际 TextLayout 反证：空 Text 为 1 行；
  空 marker 才为零高。不由旧失败 XML 推断唯一根因。
- 新六条实际 renderer 门闩区分 consumed logical intent、后续窗口/高度、
  captured anchor、Ready-before-measure/offscreen 与真实 pointer/keyboard。
  Pressure 改成持续真实 wheel 操作；保留 1000 双向、48 峰值、原 5s/300s/360s
  预算和 viewport/输出断言，无 VM 手调分页。
- 中央已实际消融 payload/height guard：6 项均红。恢复后完整 79 项通过；
  当前 pressure 相同源三次 fresh 通过。运行结论不能由 reviewer 转述成为自己的
  实测；不证明所有历史 timeout 唯一根因或 full-buffer admission 已关闭。
- 最终 bounded SHA-256
  `19aa78e36084e3c3e8e44158cd2427f4ad5552f83424e17254598bed6681338a`，
  renderer `bc05d5f75d007012b054a109b9a0e1f59ea33442bf89e1ba65078e2d2d91ad3b`。
  以此快照为准，不用作者早期 handoff 的 bounded 哈希替代。

## Independent verdict and evidence identity

- Session 549 is the independent reviewer, not implementer 548.
  **R:** final test-only delta is supported by source review; no new confirmed
  B1 was found in these two files. Continuous-input green is not proof that
  old logical-only pressure, every historic timeout, or actual admission is closed.
- Loaded AGENTS, change/checklist/kanban/planning/ask-user/document workflows,
  frontend/lifecycle/SPEC checklists, actual History contract, the investigation,
  main plan, and [preceding independent review][preceding].
  Earlier source-only/pending checkpoints and failures remain historical evidence.
- Read and verified all three manifest hashes and every listed file:
  **2/2 final, 2/2 before, 1292/1292 production-context files**, no mismatch.
  Shared dirty `Kodex/` baseline `6b7129fa` is background, not current review code.

| Evidence | Manifest SHA-256 |
| --- | --- |
| [Final two files][manifest] | `e6840e8e13625b268d7306caa561e298a3f6036769b44e3c83aa668d82cab1f1` |
| [Before two files][before-manifest] | `784e4eddacc6879a6c19a1729012cdb1e51fea934d5fd27f9625c8a4f4234be6` |
| [Production context][production-manifest] | `fff68f99f942a9bf201e26cf3ce852718a4df29f26e33757d3352cbdd7ec0c1e` |

| Original navigation, not mutable worktree evidence | Fixed-byte comparison |
| --- | --- |
| [Bounded test L116](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/impl/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/history/AgentHistoryBoundedWindowTest.kt#L116) | [Final L116–188, 201–355][bounded] versus [before L115–184, 196–271][before-bounded] |
| [Renderer test L68](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/impl/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/history/AgentHistoryRendererStateTest.kt#L68) | [Final L68–298, 651–685][renderer] versus [before L65–148, 413–437][before-renderer] |
| [Production View L338](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/history/AgentHistoryView.kt#L338) | [Production snapshot same path/lines][view]; also L250–255, 540–548 |
| [Production VM L175](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/cli/history/AgentHistoryViewModel.kt#L175) | [Production snapshot same path/lines][vm]; also L644–653, 695–704, 1013–1018 |
| [List L194](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/view/components/src/commonMain/kotlin/io/github/stream29/kodex/cli/components/LazyListState.kt#L194) | [Production snapshot same path/lines][list]; also L157–168, 221–241, 310–328 |
| [Measure L177](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/view/components/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/components/LazyColumn.kt#L177) | [Production snapshot same path/lines][measure]; also L333–367 |

- Before bounded SHA:
  `a3b1ba5d97882463c68be98c843c5b70dc0a8099f754adfde761980df69630a3`;
  before renderer:
  `309f342c26c28acdc4eb928aee2a3adb7dcc02bdaa79d4c5cd5c4d015ebe6347`.
  The author's early bounded
  `1f3fdcb23103f5bea2c1435422d901c7125d4dae9d2472f3c2c89801373a1789`
  is **not** the final `19aa78…` file and is not used for acceptance.
- Test findings below use final frozen bytes; production findings use the 1292-file
  snapshot. External Mosaic/library source references are separate evidence,
  not claimed to be included in that manifest.

## R — counterproof and six actual-renderer gates

- **Loading Text is height 1, not 0.** View L543 emits `Text("")`;
  [Mosaic TextLayout L36–54][text-layout] measures the number of split lines,
  so the empty string has one line. Empty demand/loading markers have no visual
  content. Measure L337 excludes zero-height items from visible rows; List L310
  selects positive-height anchors. Loading message and zero-height marker cannot
  be conflated, nor can payload Ready be required for production edge demand.
- **Real renderer access activates the lazy read.** [Message VM L21–51][message-vm]
  creates a LAZY Job; window get at VM L1015 calls ensureLoaded. View L255 uses
  indexed get, while key/content-type/effect access uses peek. Final RendererWindow
  L661–668 records get and starts the same existing Loading Job only there.
  No new test body calls get/start/requestOlder/requestNewer to force loading.
  Releasing a gate permits an already renderer-started read to finish.
- **Two blank-row gates, older/newer** (Renderer L68–160): consume logical
  Start/End, require active LAZY payload, exact measured Loading edge height 1
  and demand=1. Count/key readiness passes; loaded-row and target guards reject.
  Completion without frames retains the same layout: Ready cannot borrow
  one-line geometry. After measure, height=2, anchor key/offset are retained,
  demand remains one and follow remains false.
- **Four continuation gates, older/newer × pointer/keyboard** (L164–298):
  consume the old logical request first, then publish a replacement with the
  delayed edge. Overscan get activates it, but Loading remains invisible and
  demand=0. Ready-before-frame and measured-but-offscreen Ready remain insufficient.
  Anchor key/offset survive both replacement and late height; tests do not
  incorrectly require consumed logical intent to repin future UI.
- **Physical continuation is causal** (L251–283): WheelUp/Down is real pointer
  input; keyboard first clicks a fully visible neighbour. Focus acquisition must
  separately leave anchor/demand unchanged before PageUp/Down. Positive assertions
  require exact target height/intersection, demand=1 and actual Pointer/Keyboard
  interaction with consumed sign (older <0, newer >0). Further frames must not
  repeat demand. Enqueue or focus relocation alone cannot satisfy these checks.
- Removing physical input would leave the asserted target invisible with zero
  demand: this is a static responsibility check, **not reviewer-run ablation**.
  Controlled RendererHistoryModel callbacks are not actual VM admission evidence.

## R — bounded continuous input and retained invariants

- Bounded L116–155 replaces one consumed logical End/Start plus passive wait with
  continuous WheelUp/Down. Each step still captures the checked window and uses
  that same oldest/newest progress baseline; latest `.value` is not a new address.
- L228–260 emits input from the frame child and settles actual Mosaic frames
  inside the **original Default-dispatcher 5s progress wait**. No whole-test retry,
  restart after timeout, direct VM paging, enlarged budget or assertion suppression.
  Frames stop/join on exit. Consumed programmatic behavior remains covered by
  the six new gates and unchanged earlier publication/anchor controls.
- Guard L312–355 retains current-window/live Ready checks at both ends, marker
  count, nonempty stored rows, exact child/index and initial scrollability.
  Every pressure handoff additionally needs visible messages Ready and measured
  at two rows. Optional exact target/height/intersection is controlled-gate-only.
- **D:** two rows is valid for these single-line User header+content fixtures
  (widths 60/40, no transients), not a general final-height protocol, provider
  identity token, production controller or atomic snapshot lock. Pressure does
  not pass an offscreen target and passively await an unactivated lazy payload.
- Before/after diff retains 1000 items, both directions/storage boundaries,
  12-row viewport, sampled peak ≤48, initial elapsed <5s, each progress wait 5s,
  total navigation/elapsed 300s and test hang guard 360s. Real pointer exits
  follow; output excludes `999` and contains `User`; all boundary assertions stay.
  Peak is initial/completed-handoff sampling, **not global heap/all-time peak**.
  New pressure green must not be relabelled old logical-pressure green.

## R — diagnostics, threading, measure/frame and teardown

- Bounded L279–308 uses window **peek**, state/Job inspection and public
  layout/follow/effect observations; no payload activation or mirrored VM authority.
  Private list request/VM pending admission is explicitly unavailable.
  At most 16 differing checkpoints are kept, not an exact full event journal.
- Deque/lastInput/observe belong to the original test dispatcher: both launch
  children inherit it; timeout catch resumes it after withContext(Default).
  Default combine/readiness never accesses the deque. Bounded L46 enables
  TestScope; matching [TestBalloon L96–114][balloon-test] removes non-Test
  dispatchers, and [TestScope L165–180][test-scope] supplies a TestDispatcher.
  No concurrent Default ArrayDeque mutation was found. Live state/layout reads
  can span publications; observations and double-check guards remain non-atomic.
- Geometry is observed, not manually manufactured. Gates assert layout identity
  before pumping, then positive measured height afterward. Actual measure publishes
  intersecting positive-height rows and clears the consumed request
  (Measure L333–367, List L221–241).
- Bounded L178–188 retains model close, dedicated model-owner cancel/join, then
  repository cancel/join in nested necessary NonCancellable cleanup; setup remains
  inside try. Waiter L269–275 cancels/joins frame and input collector children.
  Renderer gates cancel payload gate/Job, await Job and close model; continuation
  collector is cancel/joined. [Repository factory L198–205][repository] owns a
  structured child with Sessions/runtime below it, not detached resources.
- **finally cancel is child-only, not assertion cancellation into false green.**
  [TestMosaic L38–45, 125][mosaic-test] skips its ordinary cancel when a block
  throws; local finally repairs that gap. It forwards to
  [Mosaic L152–154, 477–482][mosaic-runtime], cancelling its own `Job(parent)` and
  recomposer, not the asserting caller/TestScope. No catch-success, finally return
  or caller cancellation replaces an assertion. Structured parents await child
  completion; cancel alone is not synchronous join.
  [TestBalloon L412–418][balloon-report] records Throwable failure;
  [runTest L337–372][test-builders] joins and rethrows failure.
- External sources read/hash-checked: TestMosaic
  `f4437c369490972ce25887989e5ebbd2b4d3e1b668ab5ba62700028d1357d13b`;
  Mosaic runtime `a8849fb72eec34f8f47e9c309dfbd43e0220f15e0e1171cd97a667f355fa7c51`;
  TextLayout `47747495e21a8bcf1dbf8a88aed040b47ae02496c08ea636b03f840084cc5fac`.
  Matching TestBalloon 1.0.1-K2.4.0 / coroutines-test 1.11.0 source jars:
  `62455c90e7ff2513ebe13cb4a4c9b7d5d0ab58b5a4ee07de6b1de462fda44ce7` /
  `7159352cfc943f007fcc0003d986656538644cac3dcd8af1c9c775ca0f73f87c`.
- **D:** necessary NonCancellable joins have no new individual deadlines.
  The 360s guard cannot force uncooperative shielded cleanup to end.
  No concrete new nonterminating Job path was found; no cleanup fault matrix run.

## B1 / B2 — fixture repair does not establish historical production causality

- **B1 historical fixture:** the earlier Boolean/caller latest reread permitted
  cross-window address/baseline mismatch; the preceding review remains evidence.
  This final before already contains exact capture. New gates additionally show
  count/key, and even all-visible Ready rows, cannot authorize offscreen-target
  payload/geometry. No new confirmed B1 was found in this delta.
- **No unique historic timeout proof:** retaining an anchor while a later edge is
  offscreen is legal. Neither 509→502 nor later 422→415 identifies the exact
  consumed-provider/payload/measure/admission sequence. Counterproof rules out
  the claimed same-provider “Loading message zero-height” mechanism, not all causes.
- **B2 remains:** View L348–354 emits one demand per mounted window/edge/list;
  actual VM L175–201 can reject current/Ready/pending/full-buffer admission.
  Controlled actual-VM + mounted-renderer evidence is still needed: capture exact
  rejection, release the gate without window/marker/list remount, verify progress,
  then real leave/return or remount positive control. Fake callback counts and
  unfaulted continuous pressure do not close this gate.
- VM L644–653 / 695–704 publishes window, clears pending, then Ready; do not claim
  this source always publishes Ready before clearing pending.
  [Contract L46–62][spec] permits rejection without automatic paging retry.
  No production View/VM/API/dependency change or new abstraction is in this delta.

## Meaningful ablation / central evidence / U

| Minimal removal | Responsibility lost |
| --- | --- |
| Ready-row state/height guard | Loading or Ready-before-measure may pass a handoff |
| Exact target/intersection/height guard | Loaded neighbours may falsely authorize an offscreen edge |
| Capture/current Ready/count/key/index checks | Stale provider or cross-window baseline may masquerade as measured handoff |
| Physical continuation, leaving assertions intact | Legal preserved anchor leaves later target invisible; logical intent is not retained |
| Focus isolation/source/sign/once assertions | Enqueue, focus side effects or repeated demand may masquerade as real continuation |
| Peek-only diagnostics / frame, collector, owner finally cleanup | Observations may change activation, or failures leave children unfinished |

- Central guard-only ablation retains the new tests while removing payload/height
  guards. [Red log L988–1006][red-log] names all six failures. This proves the
  helper's readiness responsibility, **not a production View/VM red reproduction**.
- Restored **79-green** count and **three fresh unchanged-source pressure greens**
  are supplied user/coordinator evidence. Read-only log inspection corroborated
  [suite command success][green-log] and [pressure 1][pressure-1],
  [pressure 2][pressure-2], [pressure 3][pressure-3] success, with jvmTest executed
  rather than UP-TO-DATE. Logs alone do not attest source hashes or independently
  establish the 79 XML count; correlation is attributed to the central handoff.
- **U:** reviewer ran no build/test/red/green injection, IDE/UI/PTY/network/device
  operation. Actual admission, unique historic root cause and Native/full-platform
  gates are not established here. **Not all-203 approval.**
- Only this report changed (apply_patch). Earlier reports, snapshots and current
  code untouched; no process/resource control, commit/push/switch, temporary file
  or retained running resource. Shared snapshots remain for coordinator use.
  Independent lane finished; no waiting for further builds.

[preceding]: file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-07-recheck-history-fixture-consistency.md#L1
[manifest]: file:///tmp/kodex-closure-history-continuation-review-20261007/REVIEW_SOURCE_MANIFEST.json#L1
[before-manifest]: file:///tmp/kodex-closure-history-fixture-review-20261007/REVIEW_SOURCE_MANIFEST.json#L1
[production-manifest]: file:///tmp/kodex-closure-remainder-review-20261007/REVIEW_SOURCE_MANIFEST.json#L1
[bounded]: file:///tmp/kodex-closure-history-continuation-review-20261007/app/component/history/impl/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/history/AgentHistoryBoundedWindowTest.kt#L116
[before-bounded]: file:///tmp/kodex-closure-history-fixture-review-20261007/app/component/history/impl/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/history/AgentHistoryBoundedWindowTest.kt#L115
[renderer]: file:///tmp/kodex-closure-history-continuation-review-20261007/app/component/history/impl/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/history/AgentHistoryRendererStateTest.kt#L68
[before-renderer]: file:///tmp/kodex-closure-history-fixture-review-20261007/app/component/history/impl/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/history/AgentHistoryRendererStateTest.kt#L65
[view]: file:///tmp/kodex-closure-remainder-review-20261007/app/component/history/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/history/AgentHistoryView.kt#L338
[vm]: file:///tmp/kodex-closure-remainder-review-20261007/app/component/history/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/cli/history/AgentHistoryViewModel.kt#L175
[message-vm]: file:///tmp/kodex-closure-remainder-review-20261007/app/component/history/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/cli/history/MessageHistoryItemViewModel.kt#L21
[spec]: file:///tmp/kodex-closure-remainder-review-20261007/app/component/history/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/history/contract/AgentHistoryViewModel.kt#L46
[list]: file:///tmp/kodex-closure-remainder-review-20261007/app/view/components/src/commonMain/kotlin/io/github/stream29/kodex/cli/components/LazyListState.kt#L194
[measure]: file:///tmp/kodex-closure-remainder-review-20261007/app/view/components/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/components/LazyColumn.kt#L177
[repository]: file:///tmp/kodex-closure-remainder-review-20261007/agent-session/impl/in-memory/src/commonMain/kotlin/io/github/stream29/kodex/agentsession/inmemory/InMemoryKodexSessionRepository.kt#L198
[text-layout]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/Mosaic/mosaic-runtime/src/commonMain/kotlin/com/jakewharton/mosaic/text/TextLayout.kt#L36
[mosaic-test]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/Mosaic/mosaic-testing/src/commonMain/kotlin/com/jakewharton/mosaic/testing/TestMosaic.kt#L38
[mosaic-runtime]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/Mosaic/mosaic-runtime/src/commonMain/kotlin/com/jakewharton/mosaic/mosaic.kt#L152
[balloon-test]: jar:file:///home/stream/.gradle/caches/modules-2/files-2.1/de.infix.testBalloon/testBalloon-framework-core-jvm/1.0.1-K2.4.0/8504c37134cf81dd583ce05e212e0ceb9a73c92f/testBalloon-framework-core-jvm-1.0.1-K2.4.0-sources.jar!/commonMain/de/infix/testBalloon/framework/core/Test.kt#L96
[balloon-report]: jar:file:///home/stream/.gradle/caches/modules-2/files-2.1/de.infix.testBalloon/testBalloon-framework-core-jvm/1.0.1-K2.4.0/8504c37134cf81dd583ce05e212e0ceb9a73c92f/testBalloon-framework-core-jvm-1.0.1-K2.4.0-sources.jar!/commonMain/de/infix/testBalloon/framework/core/TestElement.kt#L412
[test-scope]: jar:file:///home/stream/.gradle/caches/modules-2/files-2.1/org.jetbrains.kotlinx/kotlinx-coroutines-test-jvm/1.11.0/4fb14a79175c320cf05cc31e8e573d9f2a253343/kotlinx-coroutines-test-jvm-1.11.0-sources.jar!/commonMain/TestScope.kt#L165
[test-builders]: jar:file:///home/stream/.gradle/caches/modules-2/files-2.1/org.jetbrains.kotlinx/kotlinx-coroutines-test-jvm/1.11.0/4fb14a79175c320cf05cc31e8e573d9f2a253343/kotlinx-coroutines-test-jvm-1.11.0-sources.jar!/commonMain/TestBuilders.kt#L337
[red-log]: file:///tmp/kodex-closure-history-payload-barrier-red.log#L988
[green-log]: file:///tmp/kodex-closure-history-continuation-green.log#L875
[pressure-1]: file:///tmp/kodex-closure-history-continuous-pressure-1.log#L875
[pressure-2]: file:///tmp/kodex-closure-history-continuous-pressure-2.log#L875
[pressure-3]: file:///tmp/kodex-closure-history-continuous-pressure-3.log#L975

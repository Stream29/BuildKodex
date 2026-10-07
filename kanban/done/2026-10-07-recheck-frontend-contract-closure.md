# REVIEW READY — Independent frontend/contract re-review

- 审查脚本已完成并归档；本报告对应第一固定快照，后续增量另审，
  当前状态见 [主任务](2026-10-07-plan-spec-impl-closure-and-consistency.md)。

# Task Tree

- `Independently review fixed frontend ownership and contract ablation()`
- `Compare production callbacks and retained UI tests with baseline()`
- `Report incomplete cutover, duplicate state and unwarranted layers()`

# Details

- 用户授权修复后重新审查；本任务独立只读，不授权改代码或提交。
- 固定快照与证据见
  [修复主任务](2026-10-07-plan-spec-impl-closure-and-consistency.md)。
- 独占审查：History/catalog 错误出口、Draft 父 scope、Settings queue/CAS、
  原 Application/Session 准入、History 饱和；Session Settings 旧 API 消融、
  Home Path 投影、测试转发项目、Patch 真实操作契约和 codec KDoc。
- 区分真实组件状态与 widget buffer、原捕获目标与平行模型；特别核对原 UI
  录制/真实根输入和闭包测试，不将手动 VM 调用或 fixture 调度修正称作 UI 通过。
- `SafeRw` 有 Session 缓存生产调用；应核验后保留，不能沿用“未使用”旧断言。
- 只改本文件报告；不构建/操作 UI/设备，不改原报告。核对新编译支持仅为真实
  根测试服务，无生产 widget 权威泄漏；每项列 URI/行号及最小消融建议。

## Independent report

- 独立只读复审完成；**已审查不等于运行通过**。本报告没有新增实测生产失败，
  但保留下面两个有具体时序的疑点，不能据此无条件关闭全部前端准入/失效门槛。
- 分类：`reviewed` = 固定源码/调用链与测试正文已核对；
  `confirmed bug` = 本复审确认的当前故障；`suspicion` = 有复现方案但未执行；
  `debt` = 结构/既有政策债务；`U` = 本复审未验证的运行或平台结论。
- 本轮 `confirmed bug`：**无新增运行复现**。两个 `suspicion` 不冒充已测失败；
  基线已有错误吞并、孤立 draft owner、死队列准入、整份 draft 配置覆盖及
  equal-twin 错选的修正，分别按下文接受静态修复证据，不沿用旧结论。

### Provenance and limits

- 唯一当前源码：`file:///tmp/kodex-closure-review-6b7129fa-20261007/`。
  对 `REVIEW_SOURCE_MANIFEST.json` 重新计算 SHA-256，得到
  `3d661ec7abdf3e33a8de2b103ad7a19b8504cfd2b8a1168f00980fbd7765335e`；
  **1290/1290 文件内容 hash 匹配，缺失/不匹配 0**。
- 基线比较仅使用原仓库 Git 历史
  `6b7129fa0e719dde8f88a0034b9d572679984a03` 的 `show`/`grep`/`ls-tree`；
  未以 dirty 工作树源码补充当前证据。
- 下列 `file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/...#L...` 是
  **原文件地址**；全部行号与正文取自上述固定快照，而非原目录当前内容。
  快照对应路径为去掉 `Kodex/` 前缀后的同一路径。
- 只修改本 child report。没有构建、Gradle、IDE、UI、PTY、浏览器/网络、
  实际用户 Home 操作或 commit/push。没有创建额外源码副本、测试输出或临时文件；
  未删除仍供其他审查线使用的共享快照。
- `U`：协调者的 JVM/JS/Linux 矩阵、真实终端录制产物、其他 Native 平台以及
  外部子模块/resolved dependencies 不由本只读复审重新执行或独立背书。
  不宣称“206 项目全部通过”。

### 1. Original owners and parented draft — reviewed

- 原 `ApplicationViewModel`、`NewSessionViewModel`、`AgentHistoryViewModel`、
  `SessionSettingsViewModel` 的 spec/FQCN 与真实实现仍为一套。生产根直接构造
  原 factories/VM，Settings 只绑定 exact Session/draft source；没有新 Aggregate、
  Timeline VM、Facade、平行 mutable model 或为配对新增的 DI 项目。
  根的 tabs/popup、binding 的 Agent child、History 的窗口、renderer 的 widget state
  各自承担实际责任。[生产装配][composition]
- 与基线相比，`RpcSessionDraft` 的无父 `CoroutineScope(SupervisorJob())` 已变为
  显式必需 Job 的 `SupervisorJob(parent)`，继承 owner dispatcher/context；
  completion 关闭 Composer，edit/materialize 的边界检查 owner。
  `RpcNewSessionViewModel` 将自身 local scope 传入这个**原 draft**，
  settings/composer 仍直接借用同一实例，不复制状态。[draft][draft-owner]、[New Session][new-session]
- 实际 Composer 的三个观察 job 都挂在 supplied scope 下；
  completion/close 发布 Closed 并取消观察。父 `cancelAndJoin` 等待的是这棵真实树，
  不是测试仅调用 `draft.close()` 后推测清理。显式 draft close 只取消其子 owner，
  没有 Stop、delete、close views/backend 的调用。[Composer][composer-owner]
- 新测试分别覆盖 factory parent-only cancellation、直接 draft parent-only cancellation、
  fixture `closeAndJoin`，以及 backend 正在运行时单 draft close 后 sibling 仍可编辑。
  门闩 resume 改为 `async` 后显式放行，避免在同一调用栈等待自己释放的 gate。
  这是 fixture 调度修正，不是产品取消修复的替代证据。
  [factory tests][new-session-tests]、[真实 draft/backend tests][draft-tests]
- 最小消融：**不删除 draft owner 或 Composer observations**；它们承担父取消与
  本地 child disposal 的真实责任。保留 creation/append 分离与 `createdIndex/submitted`
  记忆，不用通用 retry 或失败自动删除已创建 Session。

### 2. History/catalog error authority — reviewed, host runtime U

- 基线三个 History 捕获点及 catalog open/fork 的普通异常被注释声称“下层已报告”，
  实际下层相应挂起命令只抛错。现在 root 的 `reportOperationFailure` 和 catalog
  `onFailure` 接到同一个宿主 callback；普通 Throwable 交付一次，CE 先行 rethrow。
  没有建立 Failure VM、通知镜像或业务命令重放。
  [root callbacks/helper][root-errors]、[catalog open][catalog-open]、[catalog fork][catalog-fork]
- 真实责任链：
  `revertAndEdit → Agent.revertHistory → binding.revertHistory → runtime RPC`；
  composer 仅在 revert 成功后更新。
  `Session.fork(exact Agent, captured generation) → binding.forkHistory → global RPC`，
  再独立 `Application.openSession`；open-after-fork 失败不撤销成功 fork。
  `Catalog.requestOpen → captured interactions.openSession → original Application/registry`，
  成功才 dismiss，reload 失败恢复旧目录并重抛。
  [revert/edit][revert-edit]、[Agent 命令权威][agent-errors]、[exact Session fork][session-fork]、
  [binding RPC][binding-commands]、[catalog VM][catalog-vm]
- 要区分不抛给 caller 的 `confirmHistoryRevert`：它由原 Agent owner launch，
  验证/失败进入原 Agent notification；renderer 不再次报告。
  `requestHistoryRevert` 的同步验证和挂起 revert/fork 则走上述宿主出口。
  因此仍是一个底层 business command、对应的一个 error authority。
  [Agent confirm/command][agent-errors]、[dialog consumer][revert-dialog]
- 实际 CLI 在 `runBlocking(CliCoroutineExceptionLogger)` 内通过 `withKodexApplication`
  挂载 `SessionTreeCliScreen`，未覆盖 throwing default。
  所以产品默认是原 coroutine/host 异常边界，**不是新增可见错误弹窗**；
  新测试注入的 `MutableList<Throwable>` 只是记录器。
  [CLI mount][cli-mount]、[host logger][cli-errors]
- 最小消融：保留准确 caller outlet 和 CE 分支。不能删 catch 后改成静默失败，
  也不把同步/挂起抛错路径改成额外 Agent notification，使宿主再次报告。

### 3. History navigation buffer and renderer — reviewed; S2 open

- 基线 `trySend(Jump/Seek)` 的失败会遗失最新目的地；现在原 History VM 增加
  **单个** `pendingNavigation` 槽，原 bounded channel 只提供 `Navigate` wake。
  每个消费迭代后取出最新槽值；满 channel 自身保证还有消费迭代。
  这只是 latest-intent coalescing，不是第二窗口或导航 journal。
  [admission/withdrawal][history-admission]、[consumer][history-consumer]、
  [真实 spec][history-spec]
- 非导航消息没有被机械改成 latest-only：
  refresh/nonce/external-write 等内部 producer 仍 `send`；paging `trySend` 失败会清掉
  demand flag，等待另一次需求。后者不是承诺无损的 business write admission，
  也不能把本修复解释为全 channel 压力、全部 paging 或旧压力 timeout 已解决。
  [producers/page admission][history-producers]
- 窗口、visible child、nonce/generation 和 effect acknowledgment 都校验准确目标。
  View 保留 LazyList/测量/输入分类，真实 pointer/keyboard scrolling 才改 follow；
  高度变化局部跟随，effect 在两帧与真实目标测量后准确 ack。
  root 按 referential History child 保留 widget state，hidden binding 替换也清理；
  tab unmount 不关闭业务 VM。[renderer][history-renderer]、[root retention][history-retention]
- 新饱和测试确实阻塞原 `getExact(7)`，提交 256 次混合导航后核验最后
  Latest/Item 及 stale ack；不是 fake effect setter。已有 renderer tests 保留
  mount-before/after-effect、真实输入、viewport/paging/nonce 与 tab 状态。
  **饱和测试没有同时驱动 nonce 改变**，这点留作 S2。
  [saturation/nonce tests][history-tests]、[renderer tests][history-render-tests]
- 最小消融：保留单槽、intent identity、nonce/window guards、exact ack。
  不将其抽成导航服务，也不以 coalescing 丢掉 revert/fork/settings 等业务命令。

### 4. Settings queue and draft field CAS — reviewed; S1 open

- 原 Settings queue 现在返回真实 Boolean admission；worker 终止会 cancel channel、
  释放待执行 closures，后续 `submit` 检查 worker/channel 并报告普通拒绝。
  command-local CE 不被当成普通持久化错误，也不伪称正常 drain；
  正常 close 在 application scope 存活时排空已接受 FIFO。
  `drained` 只表示正常结束队列，不表示每项 write 成功。
  [queue][settings-queue]、[queue tests][settings-queue-tests]
- 原两组消费者实际使用返回值：Global editor 返回 `updates.submit`，
  defaults 映射 Accepted/Rejected；均用原 global operationFailure。
  Session Settings 则有意不同：close 取消自己的 FIFO/CAS、关闭 exact source，
  不排空、不关闭 models/draft/backend；其 worker CE 后 finally close，
  不留下死 consumer 接受新工作。
  [global consumer][global-editor]、[defaults consumer][defaults-queue]、
  [Session Settings worker/admission][session-settings-vm]
- draft source 不再 `withConfiguration(capturedWholeConfiguration)` 覆盖其他字段。
  它捕获 requested-field baseline，在**原 draft mutex 内的 latest settings**
  校验 changed fields，保留未请求配置及 instructions/title 等 runtime-owned 值；
  同字段竞争返回 false。rename 也核验原名称 baseline。
  persisted source 仍用完整 backend CAS 和 `editField`，不修改观察值冒充成功。
  [draft/persisted sources][draft-source]
- 新 delayed-observer 测试暂停真实 source dispatcher，调用实际 Session Settings VM
  FIFO 与原 draft mutex；覆盖竞争 model 被拒、非目标 tier/instructions/title 被保留。
  真实 global/defaults queue 测试让 operation CE 后 parent 继续存活，验证后续仅拒绝、
  RPC 写次数仍为 1；close-CAS 测试阻塞原 materialize mutex，关闭 popup 后无迟到修改。
  [actual source/queue tests][draft-tests]
- 最小消融：保留字段 baseline、原 lock 与真正 backend CAS；
  不能用 copied snapshot overwrite、projection setter 或泛化错误 retry 替代。
  queue terminal cancellation 不承诺保全未执行的 admitted writes，不能把
  cleanup callback 当成功持久化回执。

### 5. Exact root identity and actual test boundaries — reviewed

- 删除后 selection 从值相等 `in/indexOf` 改为 `indexOfFirst { it === selectedChild }`；
  select/close/materialize 仍在原 mutex 校验捕获 child。
  root tab identity registry 从 equality map 改为 referential pair list；
  Compose collection key 用 `SessionRenderKey`，清理也是 `===`。
  delete 业务仍按原 sessionIndex，不新增身份 wire 或跨删除/重建政策。
  [Application delete][root-delete]、[tab keys][tab-keys]、
  [root identity mapping][root-identity]
- `ApplicationRootBoundaryTest` 挂载实际 root renderer、原 Application owner、
  实际 registry/RPC services/backend 与 JSON transport。装饰器只在指定 RPC
  注入故障，equal-twin wrapper 只改变 equality 或调用原 Agent 的 stale-generation
  验证；不复制真实业务算法。[root test assembly][root-test-assembly]
- tests 实际 mouse Press/Release 菜单输入后断言 RPC 次数、准确 tab、
  composer text/revision、Agent identity、backend catalog 与一次 caller report；
  equal twins 的中间 tab 删除保护最后选中的实例。
  Compose compiler 与 `commonMain.compileOnly(mosaic.runtime)` 用于 test-lambda
  lowering，renderer/testing 依赖在 jvmTest；生产 Application source 没有 Compose/
  Mosaic/widget 状态导入，spec 未新增 UI 依赖。不是新 DI 空工程。
  [test cases][root-tests]、[build boundary][root-test-build]
- `awaitBoundaryCondition` 明确不要求 MutableList reporter 触发 redraw；
  frame settlement、padded menu text 命中与受阻 resume gate 的等待属于测试载体。
  root test 也只在放行原操作后核验最终 exact selection，
  **没有把“操作等待时禁止切 tab”发明成产品 policy**。
  [input/condition helpers][root-test-input]
- 三个原 JVM recordings 文件与基线逐字节相同：
  `DocsRecordingTest`、`DocsHistoryRecordingTest`、`DocsWorkspaceRecordingTest`。
  原 running/idle、pending patch、question/suggestion、History/index/revert、
  overflow tabs/shell 状态用例仍在；本审查不声称生成或观看了新的录制产物。
  [原 UI states][recording-states]、[History states][recording-history]、
  [workspace states][recording-workspace]

### 6. Contract ablation and utility responsibility — reviewed, debt retained

- **Session Settings hard cut**：snapshot 搜索无 `SessionSettingsEffect`、
  compatibility effect channel、picker forwarding getter 或旧 factory overload；
  基线生产 renderer 本就消费 `rename`/`selection`，旧 effects 只有声明、发出及
  compatibility tests，无真实生产接收者。现生产装配只调用 dependency-object factory，
  overlays 直接渲染准确 child。
  [surviving spec][session-settings-spec]、[original VM][session-settings-vm]、
  [actual overlays][session-settings-view]
- VM tests 14→14、renderer tests 7→7、spec tests 3→3（静态 test 声明计数，不是执行数）。
  唯一换名的 VM 用例从 compatibility-buffer receipt 改为 public close 释放准确
  rename child 且不 rename；field/revision/FIFO、foreign handle、editable、
  hidePage、source-close 和 child cleanup 断言保留。
  [VM regression body][session-settings-tests]、[renderer body][session-settings-render-tests]
- **Home path**：原 `io.github.stream29.kodex.utils.kodexhome.KodexHome` 仅一个真实
  top-level Path 声明，保留 host home + `.kodex` 路径语义及明确 failure KDoc；
  Provider/interface/impl project 已移除，CLI/Application 主依赖改为 spec。
  此纯查询不是 Home handle/lease，更不执行 migration。
  [real path][home-path]、[path test][home-path-test]、[CLI mount][cli-mount]
- **SafeRw 必须保留**：实际 `CachedIndexVersionedImpl` 的 indexes 以它包住
  list，查询走 readSession，publication/revert 走 writeSession。
  旧“无生产调用”断言错误，不能按该断言删锁/测试。
  [真实使用][safe-rw-consumer]、[原 implementation][safe-rw]、
  [保留 tests][safe-rw-tests]
- **Patch**：借用 explicit CoroutineFileSystem 的原同包/FQCN 扩展与完整
  algorithm 移到 spec；从 `private applyHunksToFileSystem` 起与基线逐字节相同。
  impl 只绑定 `SystemCoroutineFileSystem` 的默认 overload；
  tool 的实际 ApplyPatchToolClient 传自身 filesystem 到同一算法。
  没有 PatchApplier、新 model 或克隆算法。非事务/按序/borrowed FS/异常 KDoc
  与真实实现相符，保留行为测试含绝对路径、move、部分成功及无匹配。
  [real operation][patch-operation]、[host binding][patch-host]、
  [tool consumer][patch-consumer]、[retained behavior tests][patch-tests]
- **codec/transformer KDoc**：明确 header inspection 不保证完整 decodability，
  byte/data URL/file 三种入口的 byte bound 不混淆；transform 仅按 plan 必需时调用，
  检查 metadata 不重新 decode output；借用 FS/transformer、不承诺同步 codec
  prompt cancellation、append 是 raw bytes、write 非事务。
  exception 条件与 require/throw/委托路径一致，未把其他平台/供应商异常声称
  全部 normalized。[codec operations][codec-spec]、[transformer contract/algorithm][image-spec]、
  [IO tests][codec-tests]、[copy/metadata/failure tests][image-tests]
- 实际 codec/FS 链也已核对：file helper 委托 `CoroutineFileSystem.readBytes/useSource`，
  raw `use` 在 NonCancellable 收尾关闭并保留 primary/suppressed；
  JVM ImageIO 的 graphics/writer 在 finally dispose，Sharp await 的 CE 不包装，
  Skiko 按实际取得的 source/surface/paint/output handles 清理。
  这些支持 borrowing/异常与 cancellation caveat，不是跨平台运行验收。
  [raw IO responsibility][codec-raw-io]、[JVM actual][codec-jvm]、
  [JS actual][codec-js]、[Skiko actual][codec-skiko]
- `debt D1`：`utils-host-test-support-impl` 空 forwarding project 已删除，
  但剩余 `utils-host-test-support-spec` **也只有 build.gradle.kts**，
  仅 `api(libs.ktor.client.mock)`；唯一消费者是 OpenAI commonTest。
  它不是有行为的 fixture，不能记为“全部空 host-fixture 层已消融”。
  最小替代责任：消费者直接 test-depend `libs.ktor.client.mock`，
  再删除这个纯坐标转发（需原任务按授权决定，复审不实施）。
  [remaining empty spec][host-fixture-spec]、[only test consumer][host-fixture-consumer]
- `debt D2`：原 History/New Session `@Module/@Configuration/@ComponentScan`
  marker 文件仍在，快照没有其装配消费者或扫描到的业务 bean annotations；
  真实生产使用 typed factories。它们是基线保留的空 DI markers，
  **不是本修复新造的项目，也不是第二个 VM owner**。
  最小消融仅移除无责任 markers；是否收窄通用 viewmodel convention 要另核消费者，
  不为本复审扩大到全仓构建重构。
  [History marker][history-di]、[New Session marker][new-session-di]

### Concrete suspicions — not executed production failures

- **S1 — draft becomes nonwritable after source admission, before original mutex acquisition.**
  当前 source 在 `change` 入口检验 editable/revision，然后等待 `draft.edit`；
  editable/createdIndex 的最终检验先于 source transform，故锁等待期间 materialize
  成功可能抛 `IllegalStateException`，而非 source spec 的 nonwritable `false`。
  source owner 仍活着，VM 会把这个拒绝当普通失败交给 reporter。
  [source admission][draft-source]、[draft locked check][draft-owner]、
  [required false semantics][session-settings-spec]
  - 最小复现：复用真实 close-CAS test 的 `createSession` gate；让原
    `draft.materialize()` 持锁，启动 `source.tryUpdateConfiguration` 或 VM update
    等锁，**不要 close VM/source**；release creation 并等 materialize 成功。
    断言 source 拒绝为 false、reporter 0、原/持久 settings 未被迟到更新。
    已有测试关闭了 VM 再 release，不能证明这个 interleaving。
  - 最小责任修正方向：在**原 draft mutation lock** 内表达 nonwritable rejection，
    由 source 返回 false；不泛捕所有 IllegalStateException、不新增 draft/队列 owner。
    本复审未运行此方案，不将预期异常记为实测结果。
- **S2 — queued nonce invalidation can withdraw a newer-generation navigation.**
  nonce collector 先 withdraw，再 enqueue `Invalidate`；
  满-buffer retained navigation 可在该 queued invalidation 之前消费。
  `replaceWindow(invalidate=true)` 又无条件 withdraw，不区分新 effect 的 generation。
  因此有“新 nonce 已到达 → 新 Latest 被处理/发布 → 迟到 Invalidate 清掉它”的具体疑点。
  [nonce producer][history-producers]、[withdraw/admission][history-admission]、
  [replacement invalidation][history-replacement]、[consumer order][history-consumer]
  - 最小复现：真实 History read gate 阻塞 seek，256 次请求灌满 wake buffer；
    推进真实 cacheNonce observer 到已 withdraw/queued Invalidate，再发新的 Latest；
    release read，排空 queued commands。
    核验最终窗口是新 nonce 且最新 effect 仍待 exact ack，而非短暂出现后消失。
    同时检查 released old child 不以旧 load context进入新 generation。
  - 最小责任修正方向：仍由原 History owner 保持失效与 intent 的 generation/顺序关系；
    不新增 renderer route、nonce resolver 或通用 retry。
    已有“饱和无 nonce”与“nonce 改变撤旧 pending”两个分离用例不能关闭它。

### Test-quality gaps separate from product findings

- root cancellation case 是 decorator 主动 throw CE，证明 reporter 分支与草稿保留，
  **不等价于真实 caller Job 被取消时 backend admitted work 继续**；
  catalog 既有 [unmount/replacement tests][catalog-cancel-tests] 和静态 CE call chain 提供另外证据。
  尚缺同一 root call chain 中 caller-cancel/transport-cleanup 的联合门闩回归。
- root 测试注入非抛出 reporter，未执行实际 CLI throwing default/logging。
  factory-parent tests 验证 Closed 与 parent 无 child，但没有单独 observer
  subscribe/unsubscribe 计数；生产三条观察链已核对。
- delayed source 用例主要覆盖 model；多目标 tuple、workingDirectory/rename 的
  delayed observer 同字段竞争尚未组成同样的真实-source 矩阵。
- Patch preserved tests 实际传 explicit SystemCoroutineFileSystem；
  新 host default overload 的接线经静态核对，没有独立调用该 overload 的行为测试。
  codec bound 用例验证参数与异常委托，不是实际超出 `MaxInputBytes` 的大文件 IO/
  全部平台 codec 故障测试。
- 这些是覆盖限制，不是已证明的 UI、CAS、FS 或平台生产失败。

### Handoff conclusion

- 本 child **REVIEW READY**；静态修复链条及有责任的契约消融已核对，
  没有把 fixture 绿色、声明计数或主依赖图当作生产验收。
- 保留 S1/S2 的确定性实验入口与 D1/D2，协调者可异步读取；
  没有等待/修改其他审查报告，也没有替其关闭 parent 或迁移总任务。

[composition]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/application/src/commonMain/kotlin/io/github/stream29/kodex/cli/app/Application.kt#L89-L143
[draft-owner]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/rpc/src/commonMain/kotlin/io/github/stream29/kodex/cli/rpc/RpcSessionDraft.kt#L28-L132
[new-session]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/new-session/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/cli/newsession/NewSessionViewModel.kt#L14-L60
[composer-owner]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/composer/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/cli/agent/ComposerViewModel.kt#L61-L77
[new-session-tests]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/new-session/impl/viewmodel/src/commonTest/kotlin/io/github/stream29/kodex/cli/newsession/NewSessionViewModelTest.kt#L14-L49
[draft-tests]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/rpc/src/commonTest/kotlin/io/github/stream29/kodex/cli/rpc/DraftAndSettingsBoundaryTest.kt#L25-L205
[root-errors]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/SessionTreeCliScreen.kt#L931-L1045
[catalog-open]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-catalog/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/sessioncatalog/SessionCatalogPopup.kt#L58-L115
[catalog-fork]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-catalog/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/sessioncatalog/SessionCatalogPopup.kt#L143-L175
[revert-edit]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/SessionTreeCliScreen.kt#L1332-L1341
[agent-errors]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/rpc/src/commonMain/kotlin/io/github/stream29/kodex/cli/rpc/RpcAgentViewModel.kt#L216-L309
[session-fork]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/session/src/commonMain/kotlin/io/github/stream29/kodex/cli/session/SessionViewModels.kt#L96-L124
[binding-commands]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/rpc/src/commonMain/kotlin/io/github/stream29/kodex/cli/rpc/RpcSessionBinding.kt#L54-L80
[catalog-vm]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-catalog/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/app/sessioncatalog/DefaultSessionCatalogViewModel.kt#L39-L54
[revert-dialog]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/SessionTreeCliScreen.kt#L1281-L1291
[cli-mount]: file:///tmp/kodex-closure-review-6b7129fa-20261007/app/cli/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/Main.kt#L11-L51
[cli-errors]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/cli/src/commonMain/kotlin/io/github/stream29/kodex/cli/app/CoroutineFailureLogging.kt#L7-L14
[history-admission]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/cli/history/AgentHistoryViewModel.kt#L204-L273
[history-consumer]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/cli/history/AgentHistoryViewModel.kt#L742-L804
[history-spec]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/history/contract/AgentHistoryViewModel.kt#L121-L232
[history-producers]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/cli/history/AgentHistoryViewModel.kt#L97-L201
[history-replacement]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/cli/history/AgentHistoryViewModel.kt#L471-L527
[history-renderer]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/history/AgentHistoryView.kt#L109-L199
[history-retention]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/HistoryRenderState.kt#L12-L46
[history-tests]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/impl/viewmodel/src/commonTest/kotlin/io/github/stream29/kodex/cli/history/HistoryViewportAndScrollEffectTest.kt#L40-L203
[history-render-tests]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/impl/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/history/AgentHistoryRendererStateTest.kt#L160-L238
[settings-queue]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/rpc/src/commonMain/kotlin/io/github/stream29/kodex/app/settings/SettingsUpdateQueue.kt#L10-L90
[settings-queue-tests]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/rpc/src/commonTest/kotlin/io/github/stream29/kodex/app/settings/SettingsUpdateQueueTest.kt#L16-L95
[global-editor]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/rpc/src/commonMain/kotlin/io/github/stream29/kodex/cli/rpc/RpcGlobalEditor.kt#L97-L125
[defaults-queue]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/rpc/src/commonMain/kotlin/io/github/stream29/kodex/cli/rpc/RpcNewSessionSettings.kt#L33-L69
[session-settings-vm]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-settings/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/app/settings/SessionSettingsViewModel.kt#L28-L174
[draft-source]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/rpc/src/commonMain/kotlin/io/github/stream29/kodex/cli/rpc/RpcSessionSettings.kt#L70-L208
[root-delete]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/application/src/commonMain/kotlin/io/github/stream29/kodex/cli/app/ApplicationViewModel.kt#L403-L420
[tab-keys]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/SessionTabRenderState.kt#L35-L66
[root-identity]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/SessionTreeCliScreen.kt#L138-L163
[root-test-assembly]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/application/src/jvmTest/kotlin/io/github/stream29/kodex/cli/app/ApplicationRootBoundaryTest.kt#L214-L311
[root-tests]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/application/src/jvmTest/kotlin/io/github/stream29/kodex/cli/app/ApplicationRootBoundaryTest.kt#L46-L211
[root-test-build]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/application/build.gradle.kts#L1-L17
[root-test-input]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/application/src/jvmTest/kotlin/io/github/stream29/kodex/cli/app/ApplicationRootBoundaryTest.kt#L319-L356
[recording-states]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/view/src/jvmTest/kotlin/io/github/stream29/kodex/cli/app/DocsRecordingTest.kt#L65-L311
[recording-history]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/view/src/jvmTest/kotlin/io/github/stream29/kodex/cli/app/DocsHistoryRecordingTest.kt#L63-L164
[recording-workspace]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/view/src/jvmTest/kotlin/io/github/stream29/kodex/cli/app/DocsWorkspaceRecordingTest.kt#L33-L95
[session-settings-spec]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-settings/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/settings/contract/SessionSettingsViewModel.kt#L51-L236
[session-settings-view]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-settings/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/sessionsettings/SessionSettingsComponent.kt#L100-L145
[session-settings-tests]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-settings/impl/viewmodel/src/commonTest/kotlin/io/github/stream29/kodex/app/settings/SessionSettingsComponentViewModelTest.kt#L16-L311
[session-settings-render-tests]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-settings/impl/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/sessionsettings/SessionSettingsComponentTest.kt#L32-L249
[home-path]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/kodex-home/spec/src/commonMain/kotlin/io/github/stream29/kodex/utils/kodexhome/KodexHome.kt#L1-L14
[home-path-test]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/kodex-home/spec/src/commonTest/kotlin/io/github/stream29/kodex/utils/kodexhome/KodexHomeTest.kt#L8-L12
[safe-rw-consumer]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-session/impl/filesystem/src/commonMain/kotlin/io/github/stream29/kodex/agentsession/filesystem/CachedAgentStorage.kt#L139-L316
[safe-rw]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/read-write-mutex/impl/src/commonMain/kotlin/io/github/stream29/kodex/utils/SafeRw.kt#L8-L34
[safe-rw-tests]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/read-write-mutex/impl/src/commonTest/kotlin/io/github/stream29/kodex/utils/SafeRwTest.kt#L1
[patch-operation]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/patch/spec/src/commonMain/kotlin/io/github/stream29/kodex/utils/applypatch/PatchFileSystem.kt#L6-L102
[patch-host]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/patch/impl/src/commonMain/kotlin/io/github/stream29/kodex/utils/applypatch/HostPatchFileSystem.kt#L1-L8
[patch-consumer]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/apply-patch/src/commonMain/kotlin/io/github/stream29/kodex/tool/applypatch/ApplyPatchToolClient.kt#L11-L19
[patch-tests]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/patch/impl/src/commonTest/kotlin/io/github/stream29/kodex/utils/applypatch/ApplyPatchBehaviorRustTest.kt#L81-L362
[codec-spec]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/images-codec/spec/src/commonMain/kotlin/io/github/stream29/kodex/utils/images/codec/PromptImageCodecIo.kt#L11-L67
[image-spec]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/images/spec/src/commonMain/kotlin/io/github/stream29/kodex/utils/images/PromptImageProcessing.kt#L19-L182
[codec-tests]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/images-codec/impl/src/commonTest/kotlin/io/github/stream29/kodex/utils/images/codec/PromptImageCodecIoTest.kt#L82-L163
[image-tests]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/images/spec/src/commonTest/kotlin/io/github/stream29/kodex/utils/images/PromptImageProcessingTest.kt#L80-L117
[host-fixture-spec]: file:///tmp/kodex-closure-review-6b7129fa-20261007/utils/host-test-support/spec/build.gradle.kts#L1-L11
[host-fixture-consumer]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/impl/client/build.gradle.kts#L20-L25
[history-di]: file:///tmp/kodex-closure-review-6b7129fa-20261007/app/component/history/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/cli/history/HistoryViewModelKoin.kt#L1-L10
[new-session-di]: file:///tmp/kodex-closure-review-6b7129fa-20261007/app/component/new-session/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/cli/newsession/NewSessionViewModelKoin.kt#L1-L10
[codec-raw-io]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/kotlinx-io-coroutines/spec/src/commonMain/kotlin/io/github/stream29/kodex/utils/kotlinxiocoroutines/CoroutineRawIo.kt#L39-L98
[codec-jvm]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/images-codec/impl/src/jvmMain/kotlin/io/github/stream29/kodex/utils/images/codec/JvmImageIoPromptImageTransformer.kt#L21-L90
[codec-js]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/images-codec/impl/src/jsMain/kotlin/io/github/stream29/kodex/utils/images/codec/SharpPromptImageTransformer.kt#L20-L45
[codec-skiko]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/images-codec/impl/src/skikoNativeMain/kotlin/io/github/stream29/kodex/utils/images/codec/SkikoPromptImageTransformer.kt#L24-L88
[catalog-cancel-tests]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-catalog/impl/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/sessioncatalog/SessionCatalogPopupTest.kt#L255-L328

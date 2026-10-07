# Task Tree

- `Inventory the accepted repository and remaining real boundaries`()
- `Specify closure work from the independently reviewed findings`()
- `Receive authorization to repair and independently re-review`()
- `if (authorizedToExecute()) {`
  - `Confirm repair gates and turn doubts into controlled experiments`()
  - `coroutineScope {`
    - `async { Remove obsolete compatibility and correct actual contracts() }`
    - `async { Repair lease handoff and awaited cleanup() }`
    - `async { Preserve settings accepted during compaction() }`
    - `async { Repair approved primary-error and frontend-owner boundaries() }`
  - `}`
  - `Resolve platform and policy gates within separately approved scope`()
  - `Integrate consumers and run centralized regression`()
  - `Close remaining independent-review findings and recheck the exact delta`()
  - `Record scoped acceptance without closing unrelated policy risks`()
  - `Receive authorization to seal and commit the scoped repair`()
- `}`

# Details

- 用户现授权分批提交并封存修复现场；本任务及九份实施子任务归档，
  仅表示已复审修复范围进入固定验收基线，**不表示完整 E2E 已通过**。
  内层五批提交为 `13288e08`、`f3d1f833`、`93e304ce`、`0f17fb6e`、`7b23c481`。
  [二进制封存与 E2E 准备](../executable/2026-10-07-seal-repair-baseline-and-build-e2e-cli.md)
  接管后续；下文“未提交/等待验收”保留为当时交付 checkpoint。

- 当前汇总：[Final scoped closure evidence](#final-scoped-closure-evidence)；
  下文保留逐轮实现、失败与固定快照审查记录，不能把历史 pending 当当前状态。

## Authorization and baseline

- 用户授权本批独立验收、通过后提交，并规划下一批；本文件是下一批提案，
  原先只授权规划；用户现明确“修，修完重新审查”，授权修复确认缺陷与
  对具体疑点做确定性实验，不包含提交、推送、版本发布或新产品能力。
- UI/Home 已范围验收并本地提交；下一基线为内层 `6b7129fa0e719dde8f88a0034b9d572679984a03`。
  [上一批证据](../done/2026-10-07-cutover-ui-foundations-and-home-bootstrap.md#independent-scoped-acceptance--2026-10-07)。
- 用户随后要求立即并行审查全仓：[八线终验](../done/2026-10-07-final-reaccept-spec-impl-repository.md#coordinator-final-verdict--2026-10-07)
  已完成静态审查并归档，206 个实际项目覆盖完整。结论并非全仓通过；
  本计划据此扩大收尾门槛；修复授权已收到，政策冲突仍独立评审。
- 约束：[唯一契约](../../checklist/spec-impl-module-boundaries.md)、
  [资源生命周期](../../checklist/coroutine-resource-lifecycle.md)、
  [State 串行化](../../checklist/agent-state-mutation-serialization.md)、
  [冻结 Home](../../checklist/kodex-home.md)。

## Inventory and ablation conclusion

- 206 个实际 build 入口的静态主依赖图无缺失/环；spec 没有直接指向 impl 或
  旧混合项目的主依赖。该检查不替代实际调用链、外部依赖或平台实现验收。
- 旧 app/shared、旧业务 contract/ViewModel 根和 UI/Home 三个退役入口已接管；
  不再安排第二轮 OpenAI、RPC、Hook、Agent/Session 的机械目录迁移。
- 未按 spec/impl 配对的实际项目只有：Integration、CLI 装配、通用 UI components、
  RPC fixture、AgentState fixture、AgentSession fixture。分别是测试、入口或 UI
  基础设施；不为它们新增空 spec、Service/Controller 或第二套状态。
- 本轮收尾应从“剩余目录迁移”转为“真实契约与所有权闭环”。
  Home/State 两个已有复现并不是全部剩余工作；终验另外确认了兼容 API、
  错误出口/父 owner/主异常保留和契约差异，疑点实验与政策矛盾另列。

## Proposed batch

| 工作单元 | 实际职责与范围 | 不包含 |
| --- | --- | --- |
| 迁移终验 | 当前主图与消费者；真实 spec→实现→装配→测试；退役源唯一性；测试 fixture 仅被 test 使用 | 新造接口满足形式、外部子模块重构 |
| 最小结构消融 | Session Settings 闲置 effect/channel/getter；Home path Provider、SafeRw/fixture 转发逐项确认；实际 API KDoc 校正 | 第二套 VM、为纯函数/DTO 强造接口 |
| [Home/Lease](2026-10-07-plan-home-lease-handoff-and-cleanup.md) | 原 lease 获取/释放和 Home 等待清理；取消交接、主异常与清理异常 | 新 registry、冻结 migration、版本升级 |
| [AgentState](2026-10-07-plan-compaction-settings-consistency.md) | 原 compact checkpoint 在 Mutex 内保留期间成功提交的 settings | 阻塞整个网络等待、新配置合并服务、retry 政策 |
| 主异常/前端 owner | Runtime pending 清理、backend auth 临时文件、draft Composer 父 scope、History/catalog 原错误出口 | 通用清理框架、Failure VM、第二个 draft owner |
| 平台契约 | Shell close/exit 保证、Node 和 Native curl 的 owner 外清理、SearchIndex 输入快照 | 把 JVM 绿色作为 Native 已测、未经验证的资源泄漏断言 |

- 文件所有权分为六线：Home/lease、State/storage、前端真实 owner、
  平台资源、旧契约消融、Backend/MCP admission。协调者独占 Runtime cancellation、backend auth cleanup、
  SearchIndex 快照、Home Path Provider 消融、主 Gradle/宿主适配及集中验证。
  各线只改所属源码/测试和自己 handoff，不构建、不操作 IDE/用户设备、不提交。
  [前端](2026-10-07-repair-frontend-error-and-owner-boundaries.md)、
  [平台](2026-10-07-repair-platform-resource-lifetimes.md)、
  [消融](2026-10-07-remove-obsolete-contract-surfaces.md)补齐具体闭包；
  [Backend/MCP](2026-10-07-repair-backend-startup-and-mcp-admission.md)
  接管按需 repository 与原名字归一化的唯一 admission；
  State/storage 的写入故障疑点先复现，不扩大为新存储事务。
- 先检查并把确定性故障用例变成期望行为失败，再做最小修正；不以重跑或放宽断言
  当修复，不把“characterization 测试绿色”视为错误已解决。
- 此提案新增项目数为 **0**；优先消融确认闲置路径，保留原 `FileSystemLease`、`KodexHomeHandle`、
  `KodexAgentState` 和工厂身份。有意义的等待清理操作直接补在实际资源契约上。

## Closure gates

- 所有静态依赖债务和非配对项目逐项解释；真实工厂返回 spec 类型、生产装配使用
  该类型；不存在只在声明里自引用的替代接口、隐藏旧 owner 或转发壳。
- 显式 project 主图不包含 kRPC 全局 substitution 的外部/transitive 结果，
  resolved 边界证据独立记录；不能用“无直接边”宣称所有 spec 完全无实现支撑。
- Home 确定性取消/清理故障矩阵通过，原共享引用/互斥/进程测试保留；
  AgentState 并发 CAS/compaction、失败/取消和 RPC 观察通过。
- 受影响 JVM 测试、CLI/Integration 编译、Linux x64 主源码与隔离 CLI 回归；
  其他平台只按实际结果记录，不把一台机器编译当全部 actual 验收。
- 冻结 Home 16 文件的内容/路径、持久化与 RPC serializer 形状不变，
  原名称与唯一声明仍成立，主依赖图无新环；不新增空层。
- 主异常/取消、parent-only close、真实 UI 失败出口与平台资源矩阵按实际 owner
  验收；Session Settings 消融后保留全部有效 revision/child/UI 用例。
- 旧重验风险按证据更新而不是整体关闭。当前进入 executable，
  当前源基线 `6b7129fa` 干净；IDE 在 BuildMemoh，未干预窗口或触发同步。
- 修复完成后由不参与相应实现的审查线重验，协调者核对生产调用链与实际测试。
  实现者 handoff 不等于独立通过；无新运行结果的目标标明待验证。

## Independent backlog

- 优先实验：最新 History navigation buffer 饱和、真实 draft observer/CAS
  竞争、队列 operation cancellation 后仍存活 owner 的 admission。当前是具体疑点，
  不是已运行故障；不能以顺序 fake 测试或父 scope 取消替代相应证据。
- 另有 equal Session twins、account A→B→A、MCP refresh/reconnect、
  disk publication+cleanup/cache、settings append+timestamp、Native EOF、
  进程获取交接与日志 fixture 隔离，见终验各线的 B2/U。
- remote compaction 与工具宣称按下文用户决定执行；Backend 按需启动和
  MCP 歧义 admission 已纳入独立实施线。原 raw name、协议和冻结 wire 不改变。
- 窄屏双侧栏、pending-tool late-success 和 suggestion 非事务语义保留为已有
  明确债务；History 饱和疑点不等于历史压力 timeout 根因。
- [原重验记录](../executable/2026-10-05-reaccept-spec-impl-refactor.md)是固定历史证据，
  不把其中已在后续任务修正的 root registry 问题重复列为当前确定故障。

## Policy boundaries and execution checkpoint

- 用户已确认保留现行 20 次 remote compaction 默认，修正 SOP 冲突；
  工具能力选择“先收紧宣称”，原 wire 字段保留，历史图片和自动回答不增加产品
  算法，模型侧说明准确标注未支持。
- MCP 归一化冲突先用实际投影/路由离线用例复现并提出准确 admission 修正；
  不改变持久化 raw name、冻结工具 wire 或生成第二 dispatcher。
- eager Session root 先按实际 Home 规范补 startup 回归，明确唯一 backend
  repository 的按需装配方案；不要恢复 per-tab repository。
- 集中 JVM 与实际 JS/Linux x64 行为覆盖按资源安排；已有 Daemon 检测并显式
  JVM 装配，不能让多线竞争构建。检查冻结 Home 路径/字节与原 serializer。
- 本轮无提交/推送授权。独立复验仍失败时保留具体阻塞，不整体关闭迁移总任务。

## Coordinator integration checkpoint

- 下列是首轮实现/回归的历史 checkpoint，当前状态以
  [集中证据与末项门槛](#central-evidence-and-remaining-review-gates)为准。

- `SafeRw` 的“无生产调用”结论被实际证据否定：Session 文件缓存的
  `CachedAgentStorage.kt` 使用它管理索引读写。保留其实现和回归测试，
  不为消融而删真实责任；原审查报告保留为历史记录。
- SearchIndex 的调用方可变列表修改确已复现为失败，工厂捕获输入列表快照；
  backend auth 的 IO/取消与临时删除双故障已复现，原异常和清理原因保留。
- State 五个实际文件故障用例已执行并失败于持久化成功后的旧 `latestIndex`，
  因此在原 settings append 失败边界有界重读真实尾部。不会返回 CAS 成功、
  回滚已写记录或改变当前 `Compacting`/`RequestResponse` 状态。
- 调用方 Job 取消及 Deferred await 的调试栈恢复可能复制异常对象；
  相关测试核验原 cause 链、精确清理原因和资源退出，而不是错误要求库层
  不复制 Job 的异常。直接注入的 IO/清理故障仍核验实际 primary/suppressed。
- 当前集中回归尚有失败，新增真实根 UI 测试必须启用 Compose 编译 lowering；
  RPC 订阅需等待实际 checkpoint 发出，受阻 resume fixture 必须并发调用后
  显式释放服务门闩。这些修正不替代最终全矩阵重跑与独立审查。

## Independent re-review baseline

- 六线源码修复已交接。State、缓存与 Native 截断疑点通过真实门闩复现后修正；
  后续根 UI/Node/Linux 核心门槛已通过。完整最终矩阵和独立结论仍待汇总。
- 为避免共享树变化污染审查，固定 1290 个项目文件到本轮独占临时快照：
  `file:///tmp/kodex-closure-review-6b7129fa-20261007/`。
  `REVIEW_SOURCE_MANIFEST.json` 的 SHA-256：
  `3d661ec7abdf3e33a8de2b103ad7a19b8504cfd2b8a1168f00980fbd7765335e`。
  基线历史仍只读原仓库 `6b7129fa`；快照不包含外部子模块和 Git 历史。
  集中构建使用实际工作区；交付前核验它与此快照一致，变动需审查增量。
- 新审查者未参加对应实施，分别只写
  [Lease/State](../done/2026-10-07-recheck-lease-state-closure.md)、
  [Backend/协议](../done/2026-10-07-recheck-backend-protocol-closure.md)、
  [前端/消融](../done/2026-10-07-recheck-frontend-contract-closure.md)、
  [平台资源](../done/2026-10-07-recheck-platform-resource-closure.md)。
  未收到报告前不宣称复审通过；固定快照暂保留到本批验收，便于复核历史
  阻塞与准确字节，验收后删除，不提交为另一套源码。

## Post-snapshot validation deltas

### Central evidence and remaining review gates

- 本节至下一 handoff 的计数与等待状态是逐轮历史 checkpoint；
  当前交付口径以文末 `Final scoped closure evidence` 为准。固定报告不回写成
  审查了后来修改的字节，失败命令也不因随后绿色而改为通过。

- 当前实际 build 入口 203；settings 另注册 12 个无独立 build 文件的
  聚合根，不把它们当新增业务项目。直接主依赖图无缺失/环，fixture 没有
  主源码消费者；16 个冻结 migration/registry 路径与 `6b7129fa` 字节一致。
  外部子模块 pins、版本和 wire schema 没有本轮升级。
- 35 个 JVM test 目标的最终集中命令通过；当前 XML 共 1135 项、零失败/
  skipped，包含 Gradle `UP-TO-DATE` 结果，不称全部强制重跑。CLI 和
  Integration JVM 测试源码编译通过。此前失败命令与本次成功分开记录。
- 实际平台门槛通过：raw Process Node 14 / Linux x64 9，Shell Node 37 /
  Linux x64 33，Curl Linux x64 17。均为隔离子进程或本地临时 HTTP fixture；
  不推广为 macOS/Windows/PTY 故障矩阵通过。
- 原 4GB Daemon 链接 OOM 后，独占 8GB Daemon、单 worker 的最终
  Linux x64 release CLI 链接通过；没有修改版本、构建属性文件或操作 IDE。
  配置缓存报告的 888 个既有 serialization 问题不作为源码构建失败，
  也不宣称 resolved 依赖边界完全清净。
- 实际隔离 PTY 运行通过：初始草稿、Settings 当前页→OpenAI→Close、
  草稿输入保留及 Ctrl+C 正常退出（0）。启动、Settings 和仅编辑草稿均不
  创建 sessions 根；退出后无 Home owner 文件。空 `.locks/home` 目录不是
  遗留 lease。未做有账号/外网操作，也不由此覆盖所有交互。
- 第二快照后仅 JVM Pipe 测试的 `runCatching<Unit>` 类型显式化发生变更：
  `utils/shell-client/impl/src/jvmTest/kotlin/io/github/stream29/kodex/utils/shellclient/PipeProcessLifetimeTest.kt`，
  SHA-256 `5264d07c5c768a6ad6b46434ca73a8c16892558756d4c71a99e2aaba0f913129`。
  生产逻辑不变，JVM Shell 37 项通过；快照未改写。
- 第二轮独立报告均 READY：存储/前端未新增确认 B1，保留
  `ExternalWriteFinished` 与新导航的 B2 门闩；Native 确认 Curl 异常抑制链
  可重复/成环及 MCP 启动回滚覆盖 primary。前述测试绿色不覆盖这两个问题。
  两条末项实施线分别独占实际 Curl/MCP 和 History 原 owner，修完对确切
  增量再审；原报告结论保持历史范围。

### Final review remainder handoff

- 当前等待末项交接与独立增量结论，不宣布本批通过，不提交或推送。
- 协调者实际验证 History B2：用第二快照的原 VM（逐字节 `cmp` 相同）
  执行四个新 completion 门闩，四项均失败于结束后的 window/effect 身份；
  随后恢复作者唯一修复，VM SHA-256
  `5e6cd0f760c400a6c033e036cfbb8a77b3f6e29fea79d8d90159460a0b323367`。
  先前首次命令捕获的已是修复后的字节，结果绿色，不冒充 red；
  真正旧字节 red 与当前完整 History/renderer/RPC 回归分别记录。
- 本次完整 History 追加回归中，VM 的 50 项及 RPC 85 项通过，但旧 1000-item
  renderer 压力用例再次超时：Ready、窗口 509→502、可见 508→503，
  71 项 View 中 1 项失败。命令整体失败，不能写“追加回归全绿”；
  这是先前已记录的未闭合风险，当前 completion 修复不宣称解决其根因。
- 后续 [fixture 定位](2026-10-07-investigate-history-bounded-navigation-stall.md)
  确认旧 measured candidate 与随后另读新 model 窗口的地址/进度基线可能混用。
  只改两个测试：返回 exact 已测 window、校验 current window/Ready、用原
  logical edge 定位并正确关闭 renderer/parent。没有改生产 View 或 VM。
  消融 exact-window 校验后两条实际 renderer 门闩均失败，恢复两文件原哈希后
  完整 73 项 View 通过；不由此宣称所有历史 timeout 唯一根因已证明。
- 最终 Native/Curl 22、MCP stdio JVM 8 / Node 8、composition JVM 27、
  Shell Node 37 的集中命令通过。MCP 测试将 deprecated `launch(Job)` 改为
  显式父 scope 子 Job；仅测试作用域变化，生产 API 没有增量。
- 含这些生产末项的最终 Linux x64 CLI 再链接通过（5m12s）；
  本次配置缓存 discarded 892 问题如实保留。新二进制的隔离 PTY 实操再次
  通过、退出 0、无 owner 文件、仅编辑/Settings 不创建 sessions。
- [第三轮独立审查](../done/2026-10-07-recheck-final-remainder-closure.md) 已 READY：
  8 路径 manifest `fff68f99f942a9bf201e26cf3ce852718a4df29f26e33757d3352cbdd7ec0c1e`
  支持 History completion、Curl graph 和 MCP rollback 修复，未新增确认 B1；
  不覆盖新两文件 fixture 或保留的 B2/U。
  [fixture 独立审查](../done/2026-10-07-recheck-history-fixture-consistency.md)随后完成并归档，
  两文件 manifest `784e4eddacc6879a6c19a1729012cdb1e51fea934d5fd27f9625c8a4f4234be6`。

- 第二固定快照包含 1292 文件，相对第一快照 37 个增量路径；
  manifest SHA-256
  `13a0843b6d57557bf1c98ad2ad940b77015a91c5469b8a130a38324ca513894f`。
  独立增量复审分为
  [存储/前端](../done/2026-10-07-recheck-storage-frontend-deltas.md)和
  [Native terminal](../done/2026-10-07-recheck-native-terminal-deltas.md)。
  交付前核验实际源码与第二快照一致，后续作者交接不等同增量验收。

- 历史 checkpoint：Native CLI 链接因 4GB Daemon 堆耗尽失败，
  Daemon 自行退出；当时没有正在执行的本轮 Gradle。各增量线随后完成源码交接，
  再由协调者以独占、充足资源验证并通过。不把原资源失败写作代码或链接通过。
- 四份第一轮固定快照报告已 REVIEW READY。后续缓存/共享锁、Native 与前端
  两条具体时序实验分别继续；固定报告不改写，增量另行冻结和审查。
- 剩余 host-test-support spec 也只有第三方 mock 坐标，无代码/fixture；
  唯一 OpenAI 测试消费者改为直接依赖 mock，退役空坐标转发项目。
  两个 History/New Session Koin marker 无装配调用，直接删除，原工厂保持唯一。

- 最终矩阵暴露 Session Catalog 的失败断言类型不匹配和真实 frontend fixture
  正常关闭时等待未关闭 draft child。前者改为单次准确原因断言；fixture 在原
  connection 内显式拥有 frontend child，并在 RPC 正常收尾前 cancel/join。
  这不是产品 draft 脱离父 scope 的回滚，亦不改变实际 Application 关闭路径。
- Backend 独立审查指出 lazy repository 扫描失败后的空 supervisor child。
  新增真实 filesystem 连续失败/同 owner 重试用例，已失败于遗留 child；
  将 fallible 初始扫描提前到附加 repository child 之前，保持原工厂和存储格式。
- 上述源码/测试增量不在初始固定快照中；快照不改写。最终结论需单列这三个
  路径及 repository 回归增量，并对修复后代码复核，不能沿用旧快照通过口径。
- B1 cache admission delta — historical early handoff, source edits pending the then-current
  central Gradle completion. Owned sources: filesystem `CachedAgentStorage.kt`,
  `CachedAgentStorageTest.kt`, additions only to
  `FileSystemKodexSessionRepositoryTest.kt`, and State contract `latestIndex`
  KDoc. Preserve the coordinator's `failed initial scan never attaches an
  unreturned repository owner` test and imports; State impl/fault tests remain
  coordinator-owned.
  - Planned guards: retain `unresolvedFailure` on the original timeline;
    `requireAuthority()` under its existing `indexes` SafeRw lock gates every
    cached query, append and revert. Hold that lock through delegate access and
    reconciliation; clear uncertainty only after that actual scan succeeds.
    A failed scan leaves metadata flows at last-known values and admission
    closed until the original Session is closed/reopened; no backend poisoning.
  - Planned exact test names, for both `io` and `cancellation`:
    `failed rescan after published <kind> closes cached timeline admission`;
    `failed settings rescan after published <kind> rejects stale runtime CAS until reopen`.
    Real isolated filesystem gates publish index 5/1, fail temporary deletion
    with A, then fail enumeration with B; retain A identity/B suppression,
    raw record bytes and index, reject stale actual runtime CAS, preserve a
    healthy sibling Session, and admit latest-settings CAS after reopen.
  - Central validation and independent delta re-review: **PENDING**.

**B2 shared-read accounting delta — historical implementer handoff**

- Only this delta's
  [original read implementation](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/filesystem-lease/impl/src/commonMain/kotlin/io/github/stream29/kodex/utils/filesystemlease/FileSystemReadLeaseImpl.kt#L31)
  and new
  [dedicated accounting suite](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/filesystem-lease/impl/src/commonTest/kotlin/io/github/stream29/kodex/utils/filesystemlease/FileSystemSharedReadAccountingTest.kt#L38)
  were edited, plus this appended handoff. Spec/KDoc already states exact
  per-handle release and saved-result semantics; no spec change was needed.
- Mechanism: the original `SharedReadLeases` now atomically updates one immutable
  inventory of resolved-key acquisition pins, current raw-owner identity and
  exact per-owner references. CAS transforms do memory-only work; no IO, Job
  wait, cancellation or handle construction occurs inside them. Accounting is
  synchronous and outside cleanup deadlines, so timeout cannot discard a decrement.
  This uses the existing Kotlin 2.4 stdlib, with no new dependency/public layer.
- Lock order: pin via CAS → that key's acquisition Mutex → CAS select/reserve →
  existing filesystem guard/publication → CAS install → unlock → CAS unpin.
  Release does CAS decrement → bounded original raw `closeAndJoin` → CAS retire;
  it **never takes the acquisition Mutex**. A slow B publication cannot prevent
  A's accounting or physical release. Pins include cancelled/waiting callers and
  are dropped in the acquisition's synchronous `finally`, preventing split-key locks.
- Zero-reference/lost owners remain a retirement barrier until their actual raw
  lease Job completes. Same-key acquisition awaits that Job before publishing a
  replacement. Its memory-only completion callback removes an idle entry even
  after a close-wait timeout; it launches no Job/scope. Old-generation references
  remain independently counted, and identity checks prevent their final release
  from removing a replacement's entry. One acquisition reserves one reference
  before constructing its original handle; post-construction handoff failure
  closes/awaits that handle with the original primary/cleanup handling.
- Same active parent still gets duplicate handles over **one original raw owner**;
  another active parent still receives `FileSystemLeaseInUseException`.
  Guard failure and undelivered publication retain the prior cleanup path.
  The original raw lease and each handle remain structured children of the
  receiver owner; no manager, provider, global cleanup Job, retry or journal.
  Physical cleanup retains its 10-second cooperative limit and the 30-second
  close-wait limit. Failed physical cleanup may retain bytes and is still observable;
  no stronger compare-and-delete protocol or forced OS interruption is claimed.
- Exact new test names:
  - `unrelated gated B publication cannot block A last release or abandon its reference`
  - `same-key concurrent acquisition shares one original owner and competing parent rejects`
  - `cancelled same-key waiter consumes no reference and leaves no orphan or idle owner job`
  - `cancelled receiver waiting on a contended key cannot publish or inherit another owner's lease`
  - `cancelled publication with same-key contender cleans undelivered owner before replacement`
  - `physical cleanup keeps its real deadline and saved timeout without retaining a shared reference`
  - `one-to-zero concurrent reacquire waits for old physical release and repeated old close cannot delete new owner`
  - `owner-loss concurrent reacquire waits for retiring job and never deletes or reuses foreign read owner`
- Fixtures use isolated system filesystem directories and actual guard/owner
  bytes, syscall gates and parent Jobs. Cached, already-resolved directory headers
  let `UNDISPATCHED` callers enter the same-key Mutex wait deterministically; all
  guard/owner IO remains real. The A/B test requires A to finish within 5 real
  seconds **while B remains gated**, then repeats close and reacquire/1→0 checks.
  One test deliberately consumes the real 10-second physical deadline (15-second
  real test bound), preserving exact repeated timeout identity; no production
  deadline parameter was added. Virtual time is used only to trigger owner-loss
  renewal, not to substitute for real cleanup timeout.
- Static self-review and owned-source whitespace checks passed. The three original
  lease lifecycle/ordinary/process suites, spec and cleanup support match the
  fixed snapshot byte-for-byte; **16 frozen Home files match `6b7129fa`**.
  Independent reports and the fixed snapshot were not edited. No build/test run,
  IDE/device operation, commit, push or branch operation was performed; no
  temporary files/resources were created by this delta.
- **READY — CENTRAL VALIDATION PENDING**. Coordinator must run the new suite,
  unchanged lifecycle/process/Home consumers and affected platform compilation,
  then obtain independent review of these two source/test paths. No runtime
  red/green or independent acceptance is claimed; the original fixed snapshot's
  B2 finding remains historical evidence, not an updated acceptance report.
- B1 cache admission delta — **historical SOURCE READY handoff**.
  Source edits began after the observed central Gradle process ended. This
  handoff supersedes the early source-pending status above; it is not acceptance.
  - [Original cached timeline](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-session/impl/filesystem/src/commonMain/kotlin/io/github/stream29/kodex/agentsession/filesystem/CachedAgentStorage.kt#L145):
    `unresolvedFailure` is confined to the existing `indexes` SafeRw sessions.
    `set`/`revert` hold the write session through delegate access; value reads
    retain the read session through loading. `requireAuthority()` gates all
    cached queries (including empty ranges), append and revert. Failed append
    marks uncertainty before resync; only that successful numbered-record scan
    clears it. Failed scan retains A and suppresses B, without retry/rollback
    or cancelling the timeline, Session or borrowed repository owner.
  - [State latestIndex KDoc](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-state/spec/contract/src/commonMain/kotlin/io/github/stream29/kodex/agentstate/contract/KodexAgentState.kt#L145)
    now conditions durable-tail publication on successful reconciliation.
    Cached metadata is last-known observation, not authority; failed scan
    closes admission until the original Session is closed/reopened. Guard
    errors do not link A as their cause, avoiding a cycle when State suppresses
    a subsequent guard error on A. No new exception type/API/owner was added.
  - [Timeline regressions](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-session/impl/filesystem/src/commonTest/kotlin/io/github/stream29/kodex/agentsession/filesystem/CachedAgentStorageTest.kt#L43):
    `failed rescan after published io closes cached timeline admission`;
    `failed rescan after published cancellation closes cached timeline admission`.
    Real index 5 publication/temp-delete A/list B gate holds reconciliation;
    queued read/append/revert cannot complete before release and then reject.
    Tests check original A identity/B suppression, all query guards, last-known
    flows, no implicit scan, exact raw bytes, no index 6, and healthy rebuilt
    original timeline after owner shutdown.
  - [Actual runtime CAS regressions](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-session/impl/filesystem/src/commonTest/kotlin/io/github/stream29/kodex/agentsession/filesystem/FileSystemKodexSessionRepositoryTest.kt#L97):
    `failed settings rescan after published io rejects stale runtime CAS until reopen`;
    `failed settings rescan after published cancellation rejects stale runtime CAS until reopen`.
    Real repository/Session/runtime publish settings 1 then fault cleanup/scan.
    After one-shot faults disappear, stale, equal/no-op and durable-settings CAS
    still reject; raw settings 1 bytes stay unchanged, settings 2/timestamp 1
    remain absent. The same operation updates a healthy sibling Session under
    the live borrowed owner. Closing/reopening only the affected Session loads
    durable settings 1, rejects old expected settings, and appends 2 via real CAS.
  - Existing successful-resync, sparse-value, normal append/nonce and healthy
    revert tests are unchanged. The coordinator's initial-scan test/imports
    are preserved; only new imports/tests/fault gate were added to its source.
    State impl/existing State fault tests, review report, frozen snapshot,
    migration and Gradle files were not edited.
  - Scoped `git diff --check` passed. No build/test/IDE/device operation,
    commit/push/branch operation, temporary file or retained process was used
    for validation. Gate/owner teardown is included in test source.
    Coordinator-owned central build/regressions and independent post-snapshot
    B1 delta re-review remain **PENDING**; no runtime red/green result is claimed.

## B2-H1 History completion — staticREDTESTREADY early handoff

- Focused gate source is ready in
  [HistoryViewportAndScrollEffectTest](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/impl/viewmodel/src/commonTest/kotlin/io/github/stream29/kodex/cli/history/HistoryViewportAndScrollEffectTest.kt#L214).
  At this checkpoint the History implementation/spec have not yet been changed
  for B2-H1. Central execution, including a pre-fix red if practical, is owned by
  the coordinator; the author has run **no test/build**.
- Four exact nonce gate names:
  - `delayed revert completion retains new nonce Latest`
  - `delayed revert completion retains new nonce Latest beyond captured end`
  - `delayed rewrite completion retains new nonce Latest`
  - `delayed rewrite completion retains new nonce Latest beyond captured end`
- Actual History VM `getExact(7)` blocks while 256 mixed navigations fill its
  bounded wake channel. Controlled same-Agent state/latest/nonce flows enter
  ExternalWrite; actual `runtime.modify { revert(end); index[end] = replacement }`
  mutates InMemory storage, then nonce 41 and the actual state collector queue
  completion behind saturated wakes. A new Latest is retained before release.
  The append variants advance the actual cursor after completion captures its
  end, without another nonce. An unconfined test observer captures the exact
  new effect/window at publication; a duration FIFO barrier **after completion**
  requires that same window, child and effect to survive until exact ack.
- Positive fallback gate:
  `no nonce same cursor destructive rewrite releases old child and withdraws its exact effect`.
  Cursor stays 8 with no supplied nonce; actual revert/rewrite plus the state
  producer must advance local generation, withdraw the old effect and release
  the old lazy child. New payload reads must see rewritten data.
- All existing tests remain. Gates include old-child late access/timestamp
  rejection, exact stale/repeated ack, no mounted renderer, current payload read
  and owner teardown. This is controlled actual-VM source delivery over real
  InMemory runtime mutation, **not cached filesystem/RPC propagation evidence**.
  Expected pre-fix failure is the post-completion window/effect identity gate;
  that is static prediction, not an observed runtime red.
- Next scoped step is original-owner completion/nonce reconciliation, not
  deleting ExternalWriteFinished or introducing another VM/controller/journal.
  Independent reports and both snapshots remain unchanged.

## B2-H1 History completion — sourceREADY centralTestsPending

- This supersedes the early implementation-pending checkpoint, **not** the
  independent report's B2/U verdict. No runtime red/green or independent
  acceptance is claimed. Central regression and a new independent small-delta
  audit are both **PENDING**.
- Only three History paths changed for this delta, plus these appended
  handoffs:
  - [VM producer](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/cli/history/AgentHistoryViewModel.kt#L151)
    captures the supplied nonce on ExternalWriteFinished **before** its
    bounded-channel send can suspend.
  - [Existing loop consumer](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/cli/history/AgentHistoryViewModel.kt#L812)
    runs after the original `synchronizeGeneration()`. Same-current-nonce
    completion refreshes the **actual** latest cursor, never destructively
    replaces from captured end. Obsolete-nonce completion is ignored. There
    remains one authoritative nonce, one original History owner/window and
    one retained latest-only navigation intent; no operation journal, retry,
    new controller or second VM.
  - The **unchanged no-nonce algorithm**, nested in the fallback branch, still
    compares operation start/end with the existing `lastInvalidation` pair.
    Same-cursor rewrite invalidates locally; cursor regression already
    reconciled by Refresh is not invalidated twice. No completion responsibility
    was deleted. Typed Invalidate ordering, paging, exact effect CAS ack,
    generation-bound item contexts, visible-chunk eviction and close are intact.
  - [Normative spec](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/history/contract/AgentHistoryViewModel.kt#L125)
    states same-nonce completion/actual-cursor reconciliation and no-nonce
    replacement semantics. Demand KDoc on `requestOlder`/`requestNewer`
    precisely distinguishes Ready/edge admission and full-buffer rejection
    from retained navigation; structural failures remain Failed, with no
    synchronous throws or automatic retry.
  - [Focused gates](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/impl/viewmodel/src/commonTest/kotlin/io/github/stream29/kodex/cli/history/HistoryViewportAndScrollEffectTest.kt#L212)
    add the four nonce combinations and
    [no-nonce positive rewrite](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/impl/viewmodel/src/commonTest/kotlin/io/github/stream29/kodex/cli/history/HistoryViewportAndScrollEffectTest.kt#L349).
    Comparison to the second fixed snapshot shows only imports/test additions,
    **no deletion/change of any existing test**.
- Static before/after trace: before, synchronization loads nonce 41 and retained
  Latest publishes; delayed start≥end completion calls `replaceWindow(...,
  invalidate=true)` again, releasing that exact child and withdrawing its
  effect. After, tagged completion uses the already synchronized window and
  current cursor. Append variants check that stale captured end cannot regress
  a later same-nonce tail. This is static evidence, not executed reproduction.
- Central handoff: run all five new gates and the complete existing History
  VM suites, renderer late-mount/anchor/invalidation/effect regressions and
  affected RPC/frontend/CLI compilation. Preserve exact captured-target,
  repeated destination, stale ack, release and business paging assertions.
  Record actual test artifacts/counts against these bytes; do not inherit the
  earlier reported 45 History results as coverage of these new gates.
- Independent handoff: compare only these three paths against the unchanged
  second snapshot, trace actual producer → channel → original synchronization
  → completion consumer, verify FIFO barrier follows completion and the
  publication-time identity capture. Check source-port authoritative-nonce
  assumptions and no-nonce positive behavior. Cached FS/RPC nonce propagation,
  mounted UI and full-platform execution remain outside this author's evidence.
- Scoped `git diff --check` passed; static source/KDoc and ablation review
  completed. No build/Gradle/test, IDE, device/process/resource control,
  commit/push/branch operation or temporary file was used. Independent
  reports/snapshots, Gradle files and all other dirty owners remain untouched.
- Three-path SHA-256 inventory for central/independent correlation:
  - VM: `5e6cd0f760c400a6c033e036cfbb8a77b3f6e29fea79d8d90159460a0b323367`.
  - Spec: `47aa16d2663f87184e7b3e41ff9bca894c6b5153a06b09ea42030110889d4ee8`.
  - Focused test: `19aa965b01c38e652d5ef80221b329a33595d6f0e633495c3e7f885ebca8276e`.

## Final review remainder handoff

- B1-C1/P1 — **EARLY HANDOFF; SOURCE EDITS IN PROGRESS**. This delta owns only
  `KodexCurlMultiApiHandler.kt`, `KodexCurlTerminalCleanupTest.kt`,
  `McpStdioTransport.kt`, `McpStdioContractTest.kt`, and this appended section.
  No public/spec signatures are planned to change; any Curl disposal callback
  adjustment remains private to the original holder.
- Planned actual-handler test names in `KodexCurlTerminalCleanupTest.kt`:
  - `getInfo primary has one acyclic cleanup graph with shared faults and later exact primary`
  - `admitted cancellation has one acyclic cleanup graph with shared faults and later exact primary`
  - `successful transfer keeps first cleanup identity with reused release faults`
  - `API close exposes one cleanup graph across holders and multi release`
- Planned constructor rollback tests in `McpStdioContractTest.kt`:
  - `constructor <failure|caller cancellation> awaits exact raw rollback before rethrowing primary`
  - `constructor <failure|caller cancellation> keeps primary and one later cleanup failure`
  The existing recording raw session will expose distinct close/await counters
  and a real suspending gate; synchronous `close()` cannot satisfy that gate.
- Scope: remove Curl dual attachment, guard identity/cause/suppressed graph
  overlap, retain exact admitted primary and observable cleanup context; invoke
  fault hooks only after real native release. MCP uses the acquired session's
  `closeAndJoin()` in bounded NonCancellable constructor rollback only.
  Normal transport close/callback protocol is unchanged.
- Central execution and independent final delta review are **PENDING**.
  This early handoff is not test execution or acceptance; fixed snapshots and
  independent reports remain immutable.
- B2-H1 History — **SOURCE READY; centralTestsPending; independent delta audit
  PENDING**. The earlier staticREDTESTREADY checkpoint and the completed scoped
  [three-path handoff](#b2-h1-history-completion--sourceready-centraltestspending)
  above list the exact five new gates, nonce completion reconciliation,
  preserved no-nonce behavior and source hashes. No author-run red/green.
  Central full History/renderer/consumer regression and a new independent
  audit must cover these bytes; the old fixed review is not superseded.

- B1-C1/P1 — **SOURCE READY; CENTRAL VALIDATION AND INDEPENDENT DELTA REVIEW
  PENDING**. This supersedes only this lane's early in-progress handoff above.
  All eight planned test names are now present (four Curl, four MCP); none has
  been executed by this author.
  - [Original Curl release helper and API close](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/ktor-client-ext/impl/src/linuxMain/kotlin/io/github/stream29/kodex/utils/ktorclientext/kodexcurl/KodexCurlMultiApiHandler.kt#L43):
    one cleanup subtree is attached beneath the exact admitted primary.
    Later faults attach only to that subtree, not also to the primary. Holder
    disposal reports each independently attempted release directly to its
    original caller; scheduling rollback uses the same identity-safe policy.
    API close shares the cleanup root and original→safe-entry identity pairs
    across holders and multi cleanup, retaining the first actual cleanup
    identity when safely attachable and its observable thrown outcome.
  - A cleanup whose cause/suppression graph reaches an existing node cannot be
    attached unchanged without a cycle or duplicate identity path. That case
    gets one detached IOException containing its diagnostic text; the supplied
    throwable/cause is not rewritten. Repeated exact primary or already-reachable
    cleanup is not attached again. Reusing the same unsafe cleanup reuses its
    diagnostic entry, including across API-close holders and multi release.
  - [Actual-handler regressions](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/ktor-client-ext/impl/src/linuxX64Test/kotlin/io/github/stream29/kodex/utils/ktorclientext/kodexcurl/KodexCurlTerminalCleanupTest.kt#L131)
    inject shared faults and later exact primary only after real remove/easy/
    slist/StableRef releases. They check exact pending-promise/cancellation roots,
    successful-transfer promise and loop failure, and two real admitted holders
    plus multi release. Iterative traversal tracks visited identities and the
    active path to detect cycles; edge counts reject duplicate identity paths.
    Existing published-header/body, ordinary cancellation/close, wakeup and
    token tests remain. Promise settlement still precedes map retirement; a
    throwing completion callback cannot retain disposed references for retry.
  - [Original MCP suspend constructor catch](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/spec/stdio/src/commonMain/kotlin/io/github/stream29/kodex/mcp/stdio/McpStdioTransport.kt#L56)
    calls the acquired `process.closeAndJoin()` in NonCancellable cleanup with
    a 10-second cooperative outer deadline. Raw cleanup failure crosses this
    boundary as a value, preserving identity; deadline failure is also secondary.
    Constructor failure is rethrown unchanged, with a distinct cleanup attached
    once. Normal transport close/onClose and the original owner remain unchanged.
  - [MCP contract gates](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/spec/stdio/src/commonTest/kotlin/io/github/stream29/kodex/mcp/stdio/McpStdioContractTest.kt#L106)
    strengthen the existing RecordingRawSession with separate close/await
    counters, an entered gate and a cleanup-finished observation. Both ordinary
    rejection and actual caller-Job cancellation must remain pending while
    exact raw cleanup is gated. After release they retain the original primary
    through successful cleanup, late IO/CE, exact-primary reuse and an already
    suppressed secondary. The borrowed client stays active; teardown releases
    the gate and joins test Jobs. These are deterministic contract tests, not
    new real-Node OS fault evidence.
  - Public/spec signatures and dependencies: **unchanged**. Private changes:
    `RequestHolder.dispose(afterRelease, onFailure)` and `finishRequest`'s
    optional `recorded`/`cleanupRoot` parameters; file-private Curl graph/attach
    functions only. No generic Throwable framework, provider, transport factory
    wrapper, owner, process/Node implementation or close-protocol rewrite.
  - Scoped tracked-source `git diff --check` and untracked Curl test whitespace
    inspection passed; source/control-flow self-review completed. No Gradle/
    build/test, IDE, device, process/network fixture, commit/push or resource
    control was performed. No temporary files/resources were created.
    Fixed snapshots, independent reports and other dirty owners were untouched.
  - Coordinator should run `kodexCurlTerminalCleanupTest`, existing Curl
    lifetime/truncation consumers, `mcpStdioContractTest` on affected hosts and
    existing real Node raw/Pipe cleanup regressions; then obtain an independent
    review of these four current paths. B2-C2/W1 and other report D/U gates are
    not claimed closed by this B1 delta.
  - Four-path SHA-256 inventory:
    - Curl source: `5048c74f970bb35c2c7d5b70f90d05653d6f6b454303a0516f54773dcf84b8f8`.
    - Curl test: `76d147483555f3981bc85f4c6a143bb68d65c1bcf12148c671214a2bc2417447`.
    - MCP source: `af5763e657ec25fb46d32317f093107bb195d85b8cc0ab3ac482ddb328312b06`.
    - MCP test: `34933a629f7e2aa3ffd2949536169d5ad1a35cf798ef4551345eabd26c72059a`.

## Final scoped closure evidence

### Confirmed repairs and exact-source review

- 原终验及修复后独立复审确认的缺陷已在原 owner 中补修：Lease 交接/共享引用/
  可等待清理、State 压缩保留最新 settings、缓存不确定结果拒绝陈旧写入、父
  scope/错误出口、按需 repository、MCP admission/回滚、Process/Shell 清理屏障、
  Curl terminal outcome/抑制图。真实闲置兼容层和空转发项目已删除。
- `SafeRw` 有真实生产职责，保留；不为消融删除它。新增项目为零，
  没有新增 Aggregate、Controller、Provider 或第二套 ViewModel。
- 独立 reviewer 与对应实施者分离，读取固定源码快照；复审发现的新 B1
  先修复再审查准确增量，不将上一快照结论覆盖到后来源码。
  [八路径末项复审](../done/2026-10-07-recheck-final-remainder-closure.md)
  未新增确认 B1；[先前 fixture 复审](../done/2026-10-07-recheck-history-fixture-consistency.md)
  只覆盖它的两个历史字节。
- 最后仅两文件测试有增量：
  [continuation 独立复审](../done/2026-10-07-recheck-history-continuation-tests.md)；
  manifest `e6840e8e13625b268d7306caa561e298a3f6036769b44e3c83aa668d82cab1f1`。
  第三快照 1292 文件加这两个覆盖文件与实际工作区逐文件 SHA-256 一致，
  没有未审查的后续生产改动。最后独立报告已 READY，支持这两个测试字节，
  未发现新增确认 B1；fixture 准入条件、真实输入和所有预算/断言获静态核对。
  本批确认缺陷的限定范围修复与复审完成，现获封存提交授权，不整体关闭 B2/U。

### Final centralized validation

- 最终 35 个 JVM test 目标通过：XML **1152 项，零 failure/error/skipped**。
  此最后命令实际执行 History View，另 34 目标 up-to-date；此前完整 34 目标
  已集中验证 1073 项，不能把最后命令说成 1152 项全部强制重跑。
  CLI 主源码与 Integration JVM 测试源码编译通过。
- 实际平台门槛：raw Process Node 14 / Linux x64 9；Shell Node 37 /
  Linux x64 33；Curl Linux x64 22；MCP stdio JVM 8 / Node 8、
  composition JVM 27。故障用的是隔离子进程和本地 HTTP/SDK fixture，
  不操作用户真实服务、账户或设备。
- 包含最终生产修复的 Linux x64 release CLI 链接通过，5m12s；
  本次配置缓存 discarded 892 问题仍记录为构建债务。之前 4GB 堆 OOM
  是单列失败，不用随后成功将其改写。
- 最终二进制隔离 PTY 实操：初始草稿、Settings 当前页→OpenAI→Close、
  草稿编辑保留、Ctrl+C 退出 0；这些路径不创建 sessions 根，退出无 owner
  文件。没有做有账号/外网/全部工具能力的实际验收。
- 203 个实际项目的直接主图无缺失或环，fixture 无生产消费者；
  16 个冻结 Home migration/registry 路径和字节与 `6b7129fa` 一致。
  版本、wire schema、外部子模块 pins 无本轮变更。

### History controls and remaining limits

- 旧 1000-item logical-pressure 超时记录保留：首轮窗口 509→502，
  先前 fixture 修改后又出现 422→415。不宣布两次失败的唯一根因已证明。
- 确认的 fixture 混用窗口修复后，exact-window 条件消融 **2 项均红**；
  新 Ready/payload-height 条件消融 **6 项均红**。恢复当前准确字节后完整
  History View **79 项通过**，相同源的 continuous physical-input pressure
  三次 fresh 各 1 项通过；它们不是把失败测试重跑成成功的修复证据。
- “Loading message 零高”被真实 TextLayout 反证：空 Text 仍有 1 行。
  六条实际 renderer 门闩区分旧 logical intent 已消费、窗口/高度晚到、
  captured anchor 和新 pointer/keyboard 输入。压力测试改为持续真实滚轮，
  保留双向 1000、采样 peak≤48、5s/300s/360s 预算和输出断言；
  没有手调 VM 分页、扩大超时或修改生产滚动算法。
- 保留 B2/U：History 实际 VM admission 被拒后同 mount 的 one-shot 行为，
  kRPC 晚到错误报告与 shutdown phase、持久化 OS publication 后取消、
  Curl 真实分配器地址复用/罕见 callback 或已提交 EOF 后清理故障、
  MCP 任意异常图拓扑，以及 macOS/Windows/arm64/PTY 故障矩阵。
  待复现实验不能升级为确认故障，也不能由当前绿色宣布不存在。
- 直接 project 主图不是包含全局 kRPC substitution 的 resolved 图。
  本批只交付确认修复与限定范围复审，不宣布 203 项目全平台行为无缺陷，
  迁移总任务保持打开；本轮未提交、推送或发布。

### Artifact retention and resource release

- 运行后所有本轮隔离 Home 与 PTY helper 已删除，无活动测试/CLI worker。
  本轮独占 Daemon 在确认 idle 后按准确 PID 关闭，编译 worker 随之退出，
  没有停止用户 IDE、Kodex 或其他构建。
- 固定审查快照、准确 red/green 日志与最终计数暂保留在本轮 `/tmp`
  命名空间供本批验收，约 30MB 源码快照，不是另一套可执行项目；
  验收后删除。它们有报告 provenance 用途，不声称临时文件已全部清空。
- 收尾复核：22 份本轮记录/规范的 613 个本地链接、锚点与行号通过；
  内外层 `diff --check` 通过，两个 index 为空，内层仍固定 `6b7129fa`。
  实际生产/测试源码与最后独立审查对应的 1292 文件哈希全部匹配；
  其他用户改动未纳入本轮处理。

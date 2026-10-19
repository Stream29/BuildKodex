# Task Tree

- `Commit all existing workspace changes by concern`()
- `Trace the failed wait through renderer, RPC and runtime`()
- `Reproduce with bounded instrumentation in an isolated copy`()
- `Compare a controlled pending-preview fixture without changing production`()
- `Record cause, evidence and remaining scope`()
- `Release owned validation resources`()

# Details

## 授权与基线

- 用户要求先提交当前工作区，再研究既有 steer 超时原因；本任务不授权顺带修改生产 steer 行为或发布。
- 提交前 Kodex 源码与上一轮最终 manifest 一致，两个仓库索引为空。已按主题提交全部既存改动，未推送；提交后两个工作区干净。
- 产品检查点：`9568d1cf8`（单例 New Session）及 `aa8da0194`（既存 Gradle heap 4g→6g）。
- 外层检查点：`bbf959e`（New Session 验收）、`9d5c62f`（Ktor 文档）、`0690401`（usage 调查）、`ac14809`（消融审计）、`4bd330a`（Antigravity 规划）。
- 不改动其他任务、用户 Draft、运行中的 backend/Home 或共享窗口。后续研究记录与隔离实验不冒充这些基线提交的一部分。

## 根因与历史

- 失败用例是 [ComposerHostRepairTest.kt](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/agent/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/app/ComposerHostRepairTest.kt#L187)。
- 先前当前源码和冻结旧源码均在 `agent.pendingSteer.first { it.size == 2 }` 等待 40 秒；不是模型网络超时。证据见 [上一轮父级验收](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-18-singleton-new-session-navigation.md#L365)。
- 用例先进入 mock 模型响应、塞入 earlier，再通过真实 Enter 提交 next steer；已越过输入清空等待才停在 pending 数量等待。
- 已证实原因：用例把响应中的 `finish.await()` 当作 pending 队列不会被消费的屏障，但生产 runtime 会因 steer 取消该响应并继续消费。诊断已捕获两次正常抢占、前后端 pending=0，以及 start/earlier/next steer 各落盘一次。
- 已排除本次失败是输入事件未处理或消息丢失/重复：输入已清空，RPC 写入的两条 steer 均可从真实存储读出，内容/顺序正确。pending 是尚未消费的队列状态，不是累计发送数或入队事件日志。
- Git 历史：该测试主体来自 `a1adf2f2ac`（2026-10-05）；`cc9277360`（2026-10-12）引入模型请求中的 steer 抢占。当前 [requestResponseWithSteer](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-runtime/decorator/compact/src/commonMain/kotlin/io/github/stream29/kodex/agentruntime/decorator/compact/KodexAgentCompactionRuntimeImpl.kt#L109) 会取消模型请求并等待其收尾，再由 [StateFlowSteerProvider.take](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-runtime/steer/src/commonMain/kotlin/io/github/stream29/kodex/agentruntime/steer/StateFlowSteerProvider.kt#L20) 原子清空队列；原测试的响应等待点是可取消的。
- 抢占改动之前，旧 SteerRuntime 在 `delegate.resume()` 返回后才消费，测试依赖的正是这种时序。新行为生效后，这个旧测试前提未更新：本次 earlier 已先消费，next steer 又单独消费，等待 pending=2 没有可靠成立条件。历史说明来自源码对照，不冒充旧提交动态运行；40 秒只是 fixture 外层预算，不是一次模型网络请求耗时。

## 实验记录

- `red-original`：检查点源码原样复现，1 项失败；仍为第 214 行等待 pending=2 的 40 秒超时，整次构建 130 秒。
- `diagnostic`：只修改隔离测试，记录 mock 请求生命周期并把该等待局部限定为 2 秒；1 项预期失败，整次构建 18 秒。实际日志显示 request 1/2 均因 steer 取消，request 3 已启动；Composer 已清空，前后端 pending=0，存储用户消息恰为 start、earlier、next steer。
- 诊断失败后的 teardown 另有 storage closed 日志，发生在证据记录和超时之后；不把这个次生收尾日志当成本次超时起因。
- `gated-preview`：只在第一条 mock 响应的取消收尾处加入可释放 gate，等其进入后再执行原 UI 场景；不改生产、不删除原断言、不延长超时。1 项通过（整次构建 16 秒）；额外检查释放 gate 后两条 steer 各落盘一次、队列清空、模型请求数为 2。扩大复验见下节。
- 输入、两份变体、日志与 XML 在 [自有实验目录](file:///home/stream/ACodeSpace/demo/kodex-steer-timeout-948/)；计数以 XML 为准，不重复累计同一用例。

## 对照验证结论

- 扩大 gate 的 finally 覆盖范围，使初始化失败/取消也会释放；随后 `gated-related-suites` 退出 0，整次构建 20 秒，**74 项、0 失败、0 跳过**：
  - agent/view 23；composer/viewmodel 10；agent-runtime/steer 7；decorator/compact 34。
- 其中目标测试仍检查原提示、pending 预览只渲染一次、清空后提示消失和无错误通知；没有用删除断言换绿。最终真实存储恰为 start、earlier、next steer，模型请求 2 次，pending=0。
- 隔离副本除该测试文件外，1,305 个受跟踪文件均与产品检查点逐项 SHA-256 一致；主线 Kodex 工作区未修改。此处 74 项是隔离测试候选的结果，不宣称主线原测试已经修复。
- 最小修复方向已验证：为“队列积累到 2 时的 UI 预览”显式构造尚未消费的窗口，释放后再验证真实消费/落盘；不要关闭生产抢占、强制保留队列、延长超时或跳过测试。候选仅保存在 `results/gated-proposal.diff`，没有合入。

## 实验边界与验收

- 只在自有副本改诊断/测试 fixture，保持生产源与检查点一致；保留原红测、实验差异、命令及 XML。
- 使用真实 backend/JSON/RPC/Agent/renderer，模型外部调用 mock；观察取消与落盘，区分暂态 pending 和稳定历史。
- 对照实验若需阻挡模型取消收尾，只限测试 gate 并在 finally 释放；不以 NonCancellable 保活真实业务。
- 重型验证沿用 MacBook 空闲预检、共享锁、显式 daemon JVM、私有缓存、单 worker/4 GiB/内存守卫；不在本地启动 Gradle 或 IDE Sync。
- 收集证据后删除自有候选/缓存/临时 Home，确认进程与共享锁释放。实际产品/测试修复另待用户确认。

## 收尾

- 后续用户已授权更新测试并发布 v0.5.1；主线落地与复验见 [测试修正](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-19-fix-running-steer-preview-fixture.md#L1)。以下为研究结束时的历史边界。
- 研究完成，候选测试修正未合入主线；新增的本记录也未包含在研究前的检查点提交中。未推送、发布或替换运行中的程序。
- XML 中记录的 12 个自有临时 Home 均已不存在；收回报告后，在共享锁下删除整个远端 `kodex-steer-timeout-948` 私有副本与缓存。无自有 Gradle/Kotlin daemon 或锁残留，swap 为 0；见 `results/final-resource-cleanup.log`。
- 本地实验目录保留基线源码、诊断/修正候选、原红测和所有 XML/日志、历史源码对照及 hash；精简诊断见 `results/diagnostic-summary.txt`。
- 主会话已完成文档删减、意图对照和全文一致性复查，并验证 URI/完成态。agy 辅助审阅在 90 秒预算内未返回正文，退出 124/ERROR，不计作审阅通过；保留 `results/agy-document-review.*`，不重复占用资源。
- 研究期间另出现 Antigravity 文档的新修改和 `scratch_out.txt`；这些后续变动不是本研究产生，未覆盖、删除或再次纳入提交。

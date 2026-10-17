# Task Tree

- `Freeze the first-party source baseline and trace candidate consumers`()
- `Run isolated baseline, positive and negative ablations on the MacBook`()
- `Collect the user's accepted independent read-only reviews`()
- `Separate experimental findings from unsupported deletion claims`()
- `Verify evidence, restore inputs and release owned resources`()

# Details

## Scope and record ownership

- 用户要求全仓消融审查，并明确包括过度防御性编程；后续同意派发并行复核。
  本记录收束 Session878 的实验及 Session892/893 的只读复核，不授权产品清理。
- 本地原审查任务 `kanban/executable/2026-10-17-audit-redundant-abstractions-with-ablation.md`
  已有并行补充，本轮不覆盖其正文或阶段，也不将该未跟踪活动卡片纳入提交。
  不移动其他看板任务。
  其中新增候选不能自动获得本记录实验背书；有争议的删除理由以下表为准。
- 冻结产品 `adbe5215867d0af283feb5d69a6d2b0b42035273`。
  12 个第一方域、670 份生产 Kotlin 文件 / 80120 行已做全仓词法筛选，
  主要候选另查生产调用链、契约与测试。不是逐行审阅或完整 IDE resolve。
- 主工作区产品代码未修改。默认堆 6g 及其他会话记录保留；未提交、发布、
  重启真实后端或使用真实账号。重型 Gradle 全在 remote MacBook。

## Supported experiment findings

| 范围 | 已获得的有限结论 | 证据 |
| --- | --- | --- |
| A1：未使用的 Koin 接线 | 27 个 ViewModel 项目统一注入插件/依赖，但唯一 Koin 业务注解入口为空 module，未找到容器/生成模块调用；支持整体清掉这套无实际使用的接线 | 全仓 JVM 生产编译、Path Picker 测试及下述组合验证 |
| A2：九组固定工厂转发 | `create… → Default…Factory → constructor` 没有其他仓内 Factory 消费者；支持保留创建函数和依赖 ports，消去中间单例/接口 | 九项目 138 用例全通过；公开源码 API 兼容范围与 normative KDoc 仍须正式处理 |
| A3：旧入口 | `initializeCliLogging` 只被专用旧测试使用；`AgentRuntimeStatus`、`jsonToolFailure` 无仓内调用，支持清理 | 全仓 JVM 生产编译；31 剩余用例仅有基线同一超时。移除的四个旧 logging 测试不算通过 |
| A4：重复 Composer helper | Agent/Composer 重复布局限高、pending preview 及辅助链，可沿已有 composer-view 依赖复用 | 全仓 JVM 生产编译；29 用例仅有基线同一超时。正式清理需补共享 helper 的 API/KDoc |
| A5：五处重复依赖 | 同源集对同一 project 同时 `api` / `implementation`，可只删后者 | 全仓 JVM 生产编译及组合 metadata |
| D1：三组重复 catch | 相同处理的 SerializationException 分支已被 IllegalArgumentException 覆盖，可去掉较窄分支，保留非法输入恢复 | AgentState 66 用例全通过 |
| D2：Path Picker 过宽恢复 | 当前将 AssertionError 包装成普通文件失败；显式保留取消、限缩捕获后，普通 I/O 失败仍有原 typed failure | 合成故障对照：13 用例从 1 失败到 0 |
| D3：AGENTS.md 过宽恢复 | 当前会把调用方仍 active 时独立抛出的取消及 AssertionError 降级为 warning；显式传播取消、限缩捕获后，普通 IOException 仍是 warning | 合成故障对照：6 用例从 2 失败到 0 |

- A1–A5/D1 属于结构/分支简化候选；D2/D3 有意改变异常策略，
  不能冒充纯等价删除或批量替换全仓 catch。上述候选仍非已批准的实施补丁。
- A2 初次生成器用了错误的 SessionSettings 构造名；已纠正并独立重跑。
  首次编译失败和该轮残留报告不计为通过证据。
- **N1 实际反证：**删除
  [writer owner 校验](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/read-write-mutex/src/commonMain/kotlin/io/github/stream29/kodex/utils/ReadWriteMutex.kt#L81)
  会在非法解锁时先发布 Free、底层锁仍未释放，15 用例新增 1 失败。
  该校验必须保留；“底层也会抛错”不是可删理由。

## Independent review reconciliation

- 用户接受的两路只读 Session 均已返回；仅读源码/现有证据，不改文件、不跑测试、
  不操作 IDE/Mac 或真实后端。原回复保存于
  [核心复核](file:///home/stream/ACodeSpace/demo/kodex-ablation-878/review-892.md#L1)、
  [UI/工具/构建复核](file:///home/stream/ACodeSpace/demo/kodex-ablation-878/review-893.md#L1)。
- 下表是主审再次核对源码后的静态判定，不是新增动态测试结果。

| 原并行建议 | 复核判定 |
| --- | --- |
| D5：同步 requestReset 不会取消，可删取消重抛 | **反驳。**[端口明确允许同步投递抛取消](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/account-usage/viewmodel/src/specMain/kotlin/io/github/stream29/kodex/app/accountusage/AccountUsageViewModel.kt#L29)，且[现有测试明确断言传播](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/account-usage/viewmodel/src/commonTest/kotlin/io/github/stream29/kodex/app/accountusage/AccountUsageViewModelTest.kt#L168)。去掉分支会被后续宽 catch 当作业务失败 |
| D6：NonCancellable 内取消 catch 不可达 | **反驳。**[内部仍调用挂起的 injectHistory](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-runtime/decorator/compact/src/commonMain/kotlin/io/github/stream29/kodex/agentruntime/decorator/compact/KodexAgentCompactionRuntimeImpl.kt#L140)，依赖可自行取消/超时。单删分支还会将 INFO 取消日志误分为 ERROR |
| D7：timestamp 前后 generation 校验无并发窗口 | **反驳。**[两次检查之间是挂起式 getExact](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/cli/history/HistoryItemLoadContext.kt#L140)，可能走 RPC；后检拒绝读取期间被替换的 generation，契约也明确要求 |
| D4：CachedAgentStorage getter 全部只是虚假防御 | **缩小范围。**[timeline getter](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-session/filesystem/src/commonMain/kotlin/io/github/stream29/kodex/agentsession/filesystem/CachedAgentStorage.kt#L66) 仍拦截关闭后新获取元数据的入口；旧引用可逃逸不证明这层无效。未找到所称 close 回调重读 URI 的生产链，URI 可另做窄实验 |
| 八模块把测试桩依赖放在 commonMain | **逐项反驳。**实际都位于 commonTest/mosaicTest；例如 [filesystem](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-session/filesystem/build.gradle.kts#L21)、[history view](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/view/build.gradle.kts#L23)、[RPC server](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/server/build.gradle.kts#L39)。完整八项位置见 UI 复核 |
| A6：Patch Action/Status/Target 整体可等价压平 | **部分支持。**Action 文案可考虑从 Status 派生；但 Status 参与颜色，Target 有校验、类型及相等语义，不能凭 label 相同就宣称字符串化等价 |
| A7：SafeRw 唯一使用，可直接换 Mutex | **反驳等价性。**[读/写回调获得不同静态类型](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/read-write-mutex/src/commonMain/kotlin/io/github/stream29/kodex/utils/SafeRw.kt#L8)，且允许并发读；普通 Mutex 会串行化读。若只内联，仍须保留原临界区、读写并发及类型约束并做对照 |
| A8：旧 settings store 和专用 codec 全死 | **缩小范围。**旧具体 store 尚有迁移/集成测试消费者，KDoc 保留 legacy/testing adapter；[当前统一 store 仍调用 split-file codec](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/settings/filesystem/src/commonMain/kotlin/io/github/stream29/kodex/cli/settings/UnifiedSettingsStore.kt#L100)。可复审收拢旧具体 adapter，不能顺带删除 codec 或兼容测试 |

## Additional static candidates

- [tool/core 两个 specMain 块](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/core/build.gradle.kts#L10)
  重复声明 OpenAI models 和 serialization core 的 `api`；
  可各保留一次，不能漏掉第二块独有的 clean-model 依赖。尚未为该新增项跑消融。
- 无 budget 参数的 `retryOpenAiStreamingTransport` 只服务两个测试；
  可复审把测试直连生产使用的 WithBudget 入口。不支持删除真实 retry budget。
- MCP/RPC 的 SHA-256 算法、rotate helper、常量表仅空白不同；
  可评估共用纯 helper，不合并 OAuth 生命周期或删除其安全校验。
- Skill resolver 三处 catch-Throwable/ensureActive/warning 模式可按 D3 做逐路故障注入；
  本轮只取得静态同类证据。
- 其他 A9/A10 等并行条目未包含在主实验或上述两路完整动态复核中，
  保留为待审候选，不采纳“消融安全/测试通过”的无证据表述。

## Validation, evidence and limits

- 未修改基线：16 项目 / 270 用例，269 通过、1 个 Composer host 用例超时。
- A1–A5/D1–D3 组合后：140 个 metadata 产物、全仓 JVM 生产编译、
  Mac CLI Native 编译通过；16 项目 / 275 用例，274 通过，仅保留同一
  40000 ms 超时，用例名、异常类型及消息一致。新增故障探针通过。
- 后续 JVM Test action 强制执行并禁用其缓存；未以缓存耗时推导性能收益。
  浏览器/其他 OS、Native runtime、IDE、实际 TUI 和外部公开 API 兼容未重新验收。
- 15 次远端 Gradle 调用无资源守卫中止；最大 swap 0 MiB，
  最低可回收内存 8865.2 MiB。196 份原始证据下载后 SHA-256 全部一致。
- 隔离副本 1302 个输入已还原，无额外源码；本机产品唯一原有差异仍是默认堆 6g。
  自有三个 JVM 已停、无残留；设备锁已释放未删除。
  私有源码/Gradle/Native 缓存和传输归档已清理，仅保留小体积证据。
- [复现口径和完整证据索引](file:///home/stream/ACodeSpace/demo/kodex-ablation-878/README.md#L1)、
  [机器可读结果](file:///home/stream/ACodeSpace/demo/kodex-ablation-878/summary.json#L1)、
  [资源清理回执](file:///home/stream/ACodeSpace/demo/kodex-ablation-878/purged-resources.json#L1)。

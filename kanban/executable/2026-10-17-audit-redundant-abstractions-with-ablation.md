# Task Tree

- `Freeze the code baseline and screen all first-party production domains`()
- `Trace candidate consumers and identify semantic boundaries`()
- `Run isolated baseline and ablation controls on the remote MacBook`()
- **`Await a future opportunity to resume the partially completed audit`()**
- `Recheck supported removals rejected hypotheses and validation gaps against current code`()
- `Preserve evidence and release owned resources`()

# Details

## 当前状态 — 2026-10-19 / Session945

- 用户确认本线约完成一半，作为未来有时间可反复关注的持续审查保留；当前暂缓，不归档为全部完成。
- 下方候选与消融结果是已有阶段证据。再次推进先核对当前源码和已采纳项，不因旧卡残留而重复删除或将历史结论当作当前验证。
- 本卡保留 executable 中的暂停位置，不是正在运行的审查，也不是 Session 层级、subagent 或 Tool as CLI 的前置门槛；本轮不扩大删除或异常策略变更授权。

## Scope and authorization

- 用户要求对整体代码做消融检查，寻找确有可能冗余的抽象层和代码。
  本任务授权调查与隔离实验，不将候选直接删除到主工作区，不提交或发布。
- 用户补充将过度防御性编程纳入目标：筛选吞异常、重复校验、静默回退、
  默认空值、无依据重试和非法状态掩盖；用故障/边界反例检查删除的真实影响，
  不将必要的取消、断线、兼容和清理语义归为冗余。
- 覆盖 `Kodex/` 全部第一方业务域、工具及 build logic；嵌套外部 fork
  只检查产品使用边界，不把其源码量算成第一方全面审查。
- 从已提交产品 `adbe5215867d0af283feb5d69a6d2b0b42035273` 开始。
  并行默认堆 6g 及 Ktor 上游任务记录不属于本次改动，保留原样。
- 全仓词法筛选不等于完整调用解析或逐行审阅。单实现、短文件、
  测试专用及接口转发仅是候选信号，必须复核平台、生成代码与真实消费者。
- 消融判据：去掉抽象或路径后，必要行为、异常/取消与资源所有权、
  RPC/持久化形状、平台覆盖和依赖方向仍成立。编译成功只证明所跑编译范围，
  不能单独证明语义等价、外部兼容或性能提升。
- 历史 Home migration/codec/fixture、RPC 冻结契约、八个真实破环 contract、
  JS system 平台适配边界不得为消融方便随意裁剪。
- Gradle 仅在 remote MacBook 私有实验副本运行；复用明确的 JVM，
  使用既有设备锁与资源守卫。用户或其他会话接管资源时停止自有工作。

## Evidence

- 本地隔离根：
  [kodex-ablation-878](file:///home/stream/ACodeSpace/demo/kodex-ablation-878/)。
- 全仓筛选包含 1115 份已跟踪 Kotlin 文件，其中生产源集 670 份、
  80120 行，测试源集 443 份；覆盖 12 个第一方业务域。
  词法声明清单、长重复片段与防御语句索引保存在上述隔离根。
- Mac 私有副本基线全仓 JVM 生产编译及五个专项测试项目通过。
  A1 移除空 Koin module、ViewModel convention 的插件/依赖、
  build logic 插件依赖和 catalog 条目：全仓 JVM 生产编译及
  Path Picker JVM 测试通过。27 个 ViewModel 项目应用该 convention，
  产品中未找到 Koin 容器启动、注入、模块加载或被扫描的组件注解。
- 后续 JVM 测试使用独立 init 强制执行 Test action、禁用该 action 的
  up-to-date/build-cache，编译仍保持正常增量。基线 16 项目共 270 用例，
  269 通过，Agent View 的
  [ComposerHostRepairTest.kt:188](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/agent/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/app/ComposerHostRepairTest.kt#L188)
  固定在 40000 ms 超时；后续逐项比较用例名、异常类型和消息，
  不把它当作消融新回归，也不宣称整仓测试全绿。

## Supported structural candidates

| ID | 候选及代码证据 | 独立消融结果 / 边界 |
| --- | --- | --- |
| A1 | [ViewModel convention](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/build-logic/src/main/kotlin/kodex.kmp-viewmodel.gradle.kts#L3) 为 27 个项目注入 Koin；唯一业务注解是空的 [PathPickerViewModelModule](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/path-picker/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/app/pathpicker/PathPickerViewModelKoin.kt#L7)，未找到容器或生成模块消费者 | 移除整个无实际使用的接线，而非只删一个注解。全仓 JVM 生产编译和 Path Picker JVM 测试通过；未据此宣称 Sync/启动提速 |
| A2 | 九组 `create… → Default…Factory.create → constructor`；例如 [AccountUsage](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/account-usage/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/app/accountusage/AccountUsageViewModel.kt#L13)。每个 Factory 名称只在自己的 contract 和默认实现文件出现，无其他仓内消费者 | 移除九个 Factory 接口/单例，保留顶层创建入口、依赖 ports、原构造和 owner。全仓 JVM 生产编译、九项目 138 用例全部通过。涉及公开源码 API 与 normative KDoc，正式清理仍须确认兼容边界并迁移 KDoc；实验补丁不是文档完备的交付补丁 |
| A3 | 旧 [initializeCliLogging](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/cli/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/LoggingStartup.kt#L13) 只被其自身测试调用；当前 Main 已改用 `initializeProcessLogging`。另有零仓内调用的 [AgentRuntimeStatus](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/agent/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/agent/AgentRuntimeView.kt#L8) 和 `jsonToolFailure` | 三处入口移除，旧 helper 的 111 行专用测试随之移除；不删除现用日志、工具失败代数或失败清理。全仓 JVM 生产编译通过，Agent/Tool 31 用例仅保留基线同一超时；被删除的旧测试不计入通过用例 |
| A4 | [AgentRuntimeScreen:228](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/agent/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/AgentRuntimeScreen.kt#L228) 和 [ComposerView:150](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/composer/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/agent/ComposerView.kt#L150) 重复 `boundedComposerRows`、pending preview 及其辅助链 | 复用已存在的 composer-view 依赖，移除 Agent 侧副本，只更新测试 import，不改断言。全仓 JVM 生产编译通过，29 用例仅保留基线同一超时。需正式定义两个复用 helper 的可见性/KDoc，不增加新模块或再包一层 |
| A5 | 五处同源集对同一 project 同时声明 `api` 和 `implementation` | 只删重复 `implementation`，保留 `api`，全仓 JVM 生产编译通过；不是将跨源集/测试依赖一并去重 |
| A6 | [PatchPresentationAction 与 PatchPresentationTarget](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/view/src/commonMain/kotlin/io/github/stream29/kodex/cli/patch/PatchPresentation.kt#L22-L52)：Action 枚举（Editing, Edit, FailedToEdit）与 Status（Running, Completed, Failed）严格双射同构且仅用于拼装 header；Target sealed interface 仅包装单文件/计数但无 `when` 分支消费，唯一被读取属性为 `label` | 消融掉 Action（状态标签移入 Status）并将 Target 简化为 `targetLabel: String`；全仓渲染、测试及 Diff 显示保持语义等价 |
| A7 | [SafeRw](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/read-write-mutex/src/commonMain/kotlin/io/github/stream29/kodex/utils/SafeRw.kt#L8)：全仓仅 [CachedAgentStorage:148](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-session/filesystem/src/commonMain/kotlin/io/github/stream29/kodex/agentsession/filesystem/CachedAgentStorage.kt#L148) 一处生产使用；仅是对 ReadWriteMutex 的 reader/writer withLock 薄封装，且在其读锁内执行了磁盘 I/O | 消融封装后在 Session 内部使用标准读写锁或 Mutex，消融该工具类抽象对并发模型无语义退化 |
| A8 | [FileSystemBackendSettingsStore](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/settings/filesystem/src/commonMain/kotlin/io/github/stream29/kodex/cli/settings/FileSystemBackendSettingsStore.kt#L20) 与 [FileSystemCliFrontendSettingsStore](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/settings/filesystem/src/commonMain/kotlin/io/github/stream29/kodex/cli/settings/FileSystemCliFrontendSettingsStore.kt#L15)：守护进程架构切至单文件 UnifiedSettingsStore 后，两类及其工厂 `openBackendSettings` / `openCliFrontendSettings` 在全仓生产（包括 CLI/RPC/Migration）零调用 | 属于 0.4.7 阶段遗留拆分 store；生产已被 UnifiedSettingsStore 接管，仅被自身单测调用，消融对主线运行零影响 |
| A9 | 零调用记录操作与未挂接条目：[fileSystemSessionEntry](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-session/filesystem/src/commonMain/kotlin/io/github/stream29/kodex/agentsession/filesystem/FileSystemKodexSessionRepository.kt#L366) 及 [moveRecord/deleteRecord](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-storage/filesystem-layout/src/commonMain/kotlin/io/github/stream29/kodex/agentstorage/filesystemlayout/FileSystemLayout.kt#L84-L103) 在生产代码中零引用 | FileSystemAgentStorage 统一使用 FileSystemIndexVersioned 事务操作，上述顶层函数已无生产消费者，消融安全 |
| A10 | [ToolSearchExecution](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/tool-search/src/specMain/kotlin/io/github/stream29/kodex/tool/toolsearch/ToolSearchModels.kt#L57) 孤立协议枚举与 [completeToolCall](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-runtime/composition/src/commonMain/kotlin/io/github/stream29/kodex/agentruntime/impl/KodexAgentRuntimeComposition.kt#L186) 仅为打一条 info 日志而调用 `toResponseHistoryItems()` 反向重投影 | 序列化使用 OpenAiJson.kt 私有常量，枚举无外部引用；completeToolCall 直接依靠 `by delegate` 委托接管，消除完成工具时冗余历史解码开销 |

- A2 实际共 22 组 ViewModelFactory 接口排查：经全仓交叉调用检索，仅有 3 组（`NewSessionViewModelFactory`、`SessionCatalogViewModelFactory`、`PersistedSessionViewModelFactory`）被 `ApplicationViewModel` 作为构造参数接收用于动态懒加载。其余 **19 组**（包括 `SuggestSubagentTask`、`Composer`、`RequestUserInput`、`UsageReset`、`WorkingDirectory`、`SessionSettings`、`McpSettings`、`AuthenticationSettings`、`HookSettings`、`AccountUsage`、`ApplicationPreferences`、`ContextSourceSettings`、`DirectoryPicker`、`NewSessionDefaults`、`OpenAiLogin`、`RuntimeConfiguration`、`SessionTitleSettings`、`Settings` 等）为 100% 的虚拟单实现工厂，没有任何依赖注入、参数传递或测试 Double，所有生产/测试调用方 100% 仅调用顶层 `create*ViewModel`，工厂接口与其顶层属性 `val *Factory = *Factory(::create*)` 属于纯粹冗余抽象。
- A8 遗留架构背景：v0.4.7 将单一配置拆分为 backend/cli 两份文件，随后 v0.4.15 将其统一回单文件 `UnifiedSettingsStore`（`settings.yml`）。`MigrateToV0_4_15.kt` 包含完整自包含的 YAML 解析与规范化逻辑，完全不依赖这两个 Store。目前全仓除历史遗留单测外，生产代码调用为 **0**。`SplitSettingsFiles.kt` 中的专用编解码函数亦属于死代码。
- A10 开销细节：[KodexAgentRuntimeComposition.kt:186-266](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-runtime/composition/src/commonMain/kotlin/io/github/stream29/kodex/agentruntime/impl/KodexAgentRuntimeComposition.kt#L186-L266) 中，`completeToolCall` 仅为了打印一条 `toolLogger.info { "Tool call completed by host." }` 日志，由于 `CompletedTool` 接口未直接暴露 `callId` 属性，调用了 `completed.toResponseHistoryItems()` 将已完成事件反向解码为 OpenAI ResponseItem 列表以反查 `callId`。每次工具执行完成均触发无意义的协议对象构造与过滤开销。
- A5 清单：`agent-context/prefix → agent-context-skill`、
  `agent-state/core → tool-tool-search`、
  `app/component/openai-login/view → utils-external-url`、
  `app/rpc → app-component-openai-login-viewmodel`、
  `rpc/server → app-settings-filesystem`。
- A2 初次实验生成器误用了 `DefaultSessionSettingsViewModel` 构造名；
  实际名为 `SessionSettingsViewModelImpl`。只修正实验补丁后独立重跑通过。
  首次失败及残留测试报告不作为候选结论依据。

## Excessive defensive recovery

| ID | 当前行为与位置 | 实验对照与结论 |
| --- | --- | --- |
| D1 | [PendingToolEventProjection:197/229/240](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-state/core/src/commonMain/kotlin/io/github/stream29/kodex/agentstate/tool/PendingToolEventProjection.kt#L197) 分别捕获 `SerializationException` 和其父类 `IllegalArgumentException`，执行相同分支 | 去掉三个更窄的重复 catch，保留非法输入投影；AgentState 66 用例全通过。可简化异常分支，不是去掉输入容错 |
| D2 | [DirectoryPickerBrowser:35/65](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/path-picker/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/app/pathpicker/DirectoryPickerBrowser.kt#L35) 把 `Throwable` 包成普通文件系统失败 | 注入 IOException、取消和 AssertionError：基线 13 用例中“断言应传播”失败；收窄为 Exception 并保留取消分支后 13 用例全通过。证明恢复范围包含不应轻率降级的实现错误；正式修改需确定各平台可恢复异常集合，不能泛化为删除整个失败状态 |
| D3 | [FileSystemAgentsMd:139](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-context/agents-md/src/commonMain/kotlin/io/github/stream29/kodex/agentcontext/prefix/agentsmd/filesystem/FileSystemAgentsMd.kt#L139) 捕获 Throwable 后仅检查调用方 Job，继而返回 ReadFailed warning | 调用方仍 active 时，独立抛出的 CancellationException 被吞成警告；AssertionError 同样被降级。基线六用例的这两项失败；显式传播取消、限缩为 Exception 后六项全通过，普通 IOException 仍为警告。是已复现的异常策略问题，不是等价删行 |
| D4 | [CachedAgentStorage:66-105](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-session/filesystem/src/commonMain/kotlin/io/github/stream29/kodex/agentsession/filesystem/CachedAgentStorage.kt#L66-L105) 属性 Getter 包含虚幻存活检查 `requireActive()`：对不可变 `uri` 字符串与已实例化的 timeline 引用在 getter 读操作中做 `check(ownerScope.isActive)` | 调用方持有的 timeline 引用在 scope 取消后逃逸，getter 校验无法防范后续并发 I/O；反而导致 session 关闭回调中访问 `storage.uri` 记日志抛出意外的 IllegalStateException。改为不可变 val 字段消除关闭期异常隐患 |
| D5 | [AccountUsageViewModel:65](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/account-usage/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/app/accountusage/AccountUsageViewModel.kt#L65) 在同步非挂起方法 `requestReset()` 中机械 `catch (cancelled: CancellationException) { throw cancelled }` | 同步方法内不可能产生协程挂起，机械照搬协程规范套板代码无实际防御意义；收窄为标准业务异常捕获即可 |
| D6 | [KodexAgentCompactionRuntimeImpl:146](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-runtime/decorator/compact/src/commonMain/kotlin/io/github/stream29/kodex/agentruntime/decorator/compact/KodexAgentCompactionRuntimeImpl.kt#L146) 在 `withContext(NonCancellable)` 内部单独捕获 CancellationException 仅为打 info 日志 | NonCancellable 已隔绝外部取消，此处 catch/rethrow 属于过度防御套板 |
| D7 | [HistoryItemLoadContext:142-145](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/cli/history/HistoryItemLoadContext.kt#L142-L145) 在 3 行内的同步 `withContext(Dispatchers.Default)` 块中前后两次重复 `check(isCurrent())` | 紧邻调用前后无并发线程切换，后一个检查纯属机械重复防御，消融后对完整性无影响 |
| D8 | [PatchHistoryItemViewModel:23-35](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/cli/history/PatchHistoryItemViewModel.kt#L23-L35) 与 [WorkGroupHistoryItemViewModel:55-72](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/cli/history/WorkGroupHistoryItemViewModel.kt#L55-L72) 广泛使用 `catch (_: Throwable)` 吞掉不变量/断言检查 | `error(...)` 或 `check(...)` 抛出的 `IllegalStateException`（如 work group 预期数量不符）被直接降级为静默的 UI `Failed` 状态，抛弃了原本详尽的错误消息，排查异常时丢失关键堆栈与上下文 |
| D9 | [UsageResetViewModel:89-113](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/usage-reset/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/app/usagereset/UsageResetViewModel.kt#L89-L113) 20 行内连续 4 次 `currentCoroutineContext().ensureActive()` 与 3 次 `if (!owner.isActive)` | `local` 作用域直接以 `owner` Job 为父级，若 owner 取消，`ensureActive()` 已在上一行抛出 CancellationException；紧随其后的 `if (!owner.isActive)` 属于 100% 不可达的机械重复防御 |
| D10 | [DefaultSessionCatalogViewModel:47-48](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-catalog/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/app/sessioncatalog/DefaultSessionCatalogViewModel.kt#L47-L48) 背靠背调用 `coroutineContext.ensureActive()` 与 `owner.ensureActive()` | 注释明确指出导航关闭弹窗会取消 `owner`，故弹窗关闭被移至操作外；若内部 `captured.openSession(entry.sessionIndex)` 触发了会话切换和清理，后续对已取消 `owner` 的 `ensureActive()` 将意外中断正常的会话跳转流程 |

- D2/D3 为合成故障注入，未宣称已复现真实用户故障。它们支持收窄恢复策略，
  但有意改变 Error/独立取消的可观察结果，不能冒充纯重构或直接批量替换所有 catch。
- Skill resolver 的 `runCatchingIo`、`readResource`、`read` 也有
  `catch Throwable → ensureActive → warning/failure` 模式，值得同类复审；
  本轮未对这些路径逐项做动态消融，不将它们列为已验证修复。

## Rejected removal and protected boundaries

- **N1：writer owner 前置校验不能删。** 在
  [ReadWriteMutex:81](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/read-write-mutex/src/commonMain/kotlin/io/github/stream29/kodex/utils/ReadWriteMutex.kt#L81)
  去掉三行看似与底层 Mutex 重复的检查后，15 项同步测试新增一项失败：
  非法 owner unlock 抛错前已把 State.Write 发布为 Free，而底层锁仍未释放。
  必须在发布状态前验证 owner；不能仅因底层也会抛异常就删除。
- 历史 Home migration/codec/fixture（如 [MigrateToV0_3_3.kt:67](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/migration/src/cliHostMain/kotlin/io/github/stream29/kodex/app/migration/v0_3_3/MigrateToV0_3_3.kt#L67) 的 `removeLegacyPatchDelta`）虽无现代生产调用，但作为历史版本升级链严格保留。
- 图像编解码多平台实现（`SharpPromptImageTransformer`、`JvmImageIo`、`GdiPlus`、`Skiko`）虽在单平台词法引用低，但属于 KMP 多平台 target 必要适配，予以保留。
- `SuggestSubagentTaskDecision` 虽仅有内部 Accepted/Rejected 常量引用，但属于对外 Tool 输出 JSON Schema 属性，予以保留。
- `OAuthCallbackListener` 虽仅有单生产实现与单测试 Fake，但作为网络回调测试 Seam 予以保留。
- 时间字段读取失败隐藏、Settings YAML 隐去凭据诊断、关闭后的命令 no-op、
  generation/nonce/准确 handle 校验、有限 NonCancellable 清理及 suppressed
  failure 保留都有现行契约或并发/安全理由，本轮未支持删除。
- Test-support / in-memory fixture 的生产源集没有生产入口，不等于死代码；
  它们供跨模块测试复用。冻结历史 codec/fixture、RPC service 具体重声明、
  八个破环 contract 和 JS system 适配也不在无依据的裁剪范围内。

## Static follow-up candidates

- **生产源集泄漏测试桩依赖**：8 个生产模块（包括 `agent-session:filesystem`、`app:application`、`app:component:history:view`、`app:component:history:viewmodel`、`app:component:new-session:viewmodel`、`app:session`、`app:view`、`rpc:server`）在 `commonMain` 中使用了 `implementation(project(":agent-session-in-memory"))` 或 `implementation(project(":rpc-in-memory"))`。[InMemoryKodexSessionRepository.kt:15](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-session/in-memory/src/commonMain/kotlin/io/github/stream29/kodex/agentsession/inmemory/InMemoryKodexSessionRepository.kt#L15) 与 [InMemoryKodexAgentStorage.kt:26](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-storage/core/src/commonMain/kotlin/io/github/stream29/kodex/agentstorage/inmemory/InMemoryKodexAgentStorage.kt#L26) 虽被放置在 `commonMain`，但全仓 0 生产调用，100% 仅由测试源集消费。生产源集无任何类引用测试桩，将其迁移至 `test` 源集可解耦生产构建图并避免测试桩混入生产类路径。
- RPC `Pkce.kt` 和 MCP `McpOAuth.kt` 的 SHA-256 函数、rotate-right helper、
  常量表仅空白不同：三段分别词法归一后完全一致，详见
  [PKCE 对照](file:///home/stream/ACodeSpace/demo/kodex-ablation-878/pkce-duplicate.json#L1)。
  支持复审共用纯 helper，不支持合并两套 OAuth 生命周期或删除 state、
  issuer/resource/PKCE 校验；共享 owner、依赖方向及双路径运行验证尚未设计。
- OpenAI 的 `retryOpenAiStreamingTransport` 只由两个测试调用，
  生产路径使用 `retryOpenAiStreamingTransportWithBudget`；
  可复审把测试直接连到实际入口，避免为测试保留一层转发。
  实际 retry budget、取消传播和协议失败分类仍有生产用途，未支持删掉重试机制。

## Coverage and limitations

| 第一方域 | 生产文件 | 本轮判定重点 |
| --- | ---: | --- |
| agent-context | 15 | D3；Skill 同类恢复待复审 |
| agent-runtime | 13 | 单步状态、自动压缩及工具装饰器职责不同，未支持压成一层 |
| agent-session | 7 | cache、lease、Session ownership 非纯转发 |
| agent-state | 10 | D1；原子状态转换与 runtime 编排仍有边界 |
| agent-storage | 58 | 持久化/只读与可写视图及历史 codec 保留 |
| app | 259 | A1–A4、D2；不合并 View/ViewModel 或 frontend/backend |
| hook | 1 | 通知执行和退出清理未发现足够删除证据 |
| mcp | 15 | SHA-256 重复候选；连接/认证 owner 保留 |
| openai | 41 | 测试入口转发候选；跨线/供应商模型兼容不按少引用删除 |
| rpc | 66 | A5、PKCE 候选；wire、异常还原及重连不裁剪 |
| tool | 52 | A3 中未使用 helper；不删除失败代数/typed tool 适配 |
| utils | 133 | N1 反证；平台 IO、锁和清理差异不按形似合并 |

- build logic/catalog 和业务构建脚本另做注入/同 scope 依赖检查。
- 没有完整 IDE resolve、公开库外部消费者证明，也没有对每个声明逐行验收；
  词法引用、生成代码和反射盲点在手工候选检查中分别处理。
- 全仓浏览器、Linux/Mingw、iOS 设备、Native runtime、真实账号/网络业务及
  交互 TUI 本轮未重新执行；不以 JVM 消融或 metadata 编译替代这些验收。
- 后续采纳时应分开处理“无实际使用的结构清理”和“异常策略修正”；
  公开 API/KDoc、平台覆盖及现有失败测试各自保留验收边界。

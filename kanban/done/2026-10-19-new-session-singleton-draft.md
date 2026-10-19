# Task Tree

- `Await first-wave Session approval`()
- `Load frozen interfaces and inspect draft ownership`()
- `Implement reusable draft and shared-default settings`()
- `Migrate RPC fixtures and owned regression tests`()
- `Review scoped diff and report evidence`()
- `Receive parent acceptance`()

# Details

## 父级最终验收

- 已通过静态审查、RPC/VM/Application/集成回归及 macOS Arm64 Native/PTY 的单例复用检查，任务移入 done；实际证据见父任务 §6.7–6.10。
- 后文“未执行/待验收”为 worker 交付时点，不替代父级最终结果。

## 上下文与授权

- 父任务：[singleton New Session](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-18-singleton-new-session-navigation.md#L1)，重点第 1、2.1、6 节；用户授权实现，主会话验收。
- 工作根 `/home/stream/ACodeSpace/push/BuildKodex`，产品根 `Kodex/`。加载根指导、相关技能、Draft、父任务以及 RPC 状态/Session、frontend、资源生命周期 checklist。
- 第一波 B，与 A/C 独立改文件；不运行 Gradle/Native/IDE Sync、联网模型或远端操作，不提交/推送、不另派 worker。保留既存改动。

## 唯一写范围

- `Kodex/app/component/new-session/viewmodel/**`（spec、实现、build、测试）。
- `Kodex/app/rpc/src/commonMain/kotlin/io/github/stream29/kodex/cli/rpc/RpcSessionDraft.kt`。
- `Kodex/app/rpc/src/commonTest/**`：共享测试统一由 B 迁移，包含 C 删除 RpcDraftSettingsSource 后的旧 draft-settings 测试。保留与本任务无关的全局/持久化/取消语义覆盖。
- `Kodex/app/test-support/rpc/**`：共享 fixture 与 factory 调整由 B 独占。
- 不改 `RpcSessionSettings.kt`（C）、Application（D）、New Session View（E）、integration-test（F）、checklist 或父兄任务。

## 冻结接口与职责

- `NewSessionViewModel : AgentSettingsViewModel, AutoCloseable`，删除草稿 name/rename/clearExplicitThreadName。
- `submit(expectedRevision: Long): PersistedSessionViewModel?` 替换 materialize；`NewSessionViewModelArguments(initialWorkingDirectory: Path)`；factory port 保留 `create(arguments)`。
- 默认实现 factory 构造固定 `(views, sessions, global: RpcGlobalSettings, scope)`；借用 global，其 models 直接使用，不能由 draft close。
- 本地只持有 composer/cwd；settings 从后端四字段投影与 cwd 派生。编辑经现有完整 CAS/冻结目标字段语义，model/effort/tier 原子更新，questions 独立，不写坏其他 defaults。
- 创建调用 A 的 `createSession(cwd)`，普通创建不传前端 defaults；C 的 persisted handle 不再继承 SessionViewModel。
- 只对长度 0 无操作；纯空格/换行和首尾空白均按原文追加。保持普通 persisted Composer 既有行为，不做全局 trim 迁移。
- 接受前验证 revision，拒绝进行中重复提交；不能将重复命令排队再执行。正常路径创建、打开、追加、resume admission 后准确清空/重置，单例继续可用；清空不覆盖后续编辑，cwd 重置不覆盖后续目录编辑。
- 移除 createdIndex/submitted/草稿命名和永久不可编辑状态；失败抛出，不自动重试、续交、转移草稿、删除实体或强制导航。
- 借用 views/sessions/global；显式 owner 管理 Composer/配置观察，renderer 卸载不关闭。与 Application 的 accepted-work 交接遵循父任务 §6.2；不可用 NonCancellable 包住正常提交。

## 测试迁移与检查

- 更新 shared fixture：先设后端 defaults 为 test-model 等准确配置，再 cwd-only create；初始化 RpcGlobalSettings 并按新 factory 构造；关闭其自有 observations，保留临时目录清理。
- fixture 的 `draft()` 不再接受草稿名字；需要 persisted 名字的测试在创建后显式 rename。不要恢复草稿命名来兼容旧测试。
- 旧草稿 Settings tests 改为“不存在草稿设置来源”和新共享-default 语义，保留已有 persisted Settings CAS/取消测试；C 提供 unavailable source，不能为让测试编译恢复 RpcDraftSettingsSource。
- 覆盖双向 defaults、原文、空输入、准确 revision、重复提交、第二次有效提交、成功后清空/重置、生命周期/取消及异常不自动重放。
- 本次仅写测试和做静态检查；统一构建归主会话。上游暂未完成不是添加旧 API fallback 的理由。

## 交付报告（2026-10-19，第一波 B）

- Session：`954`，[storage](file:///home/stream/.kodex/sessions/954/)。
- 已按本任务范围实施；未提交/推送、未另派 worker、未启动 Gradle/Native/IDE Sync、远端、GUI 或线上 API。共享工作树中 A/C 的变化和既存 `gradle.properties` 均未由 B 修改。
- 父会话尚未验收；下文是源码与静态检查证据，不是测试通过声明。

### 实际文件

- [NewSession 契约](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/new-session/viewmodel/src/specMain/kotlin/io/github/stream29/kodex/app/session/contract/NewSessionViewModel.kt#L8)：独立 `AgentSettingsViewModel/AutoCloseable`、cwd-only arguments、nullable revision-bound submit；删除命名和 materialize 契约。
- [NewSession 实现](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/new-session/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/cli/newsession/NewSessionViewModel.kt#L13)：冻结的 factory 构造、借用 global/models、字段更新与 persisted registry 交接、自有 child 清理。
- [RpcSessionDraft](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/rpc/src/commonMain/kotlin/io/github/stream29/kodex/cli/rpc/RpcSessionDraft.kt#L44)：共享 defaults 投影、本地 cwd/revision、非排队提交、原文追加、准确清空与重置；删除 allocation/submitted/草稿命名及永久禁编辑状态。
- [共享 RpcFrontendFixture](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/test-support/rpc/src/commonMain/kotlin/io/github/stream29/kodex/app/test/RpcFrontendFixture.kt#L29)：暴露自有 global，后端 defaults 设为 test-model；`create()` 只传 cwd、创建后 rename；`draft()` 不接名字；关闭 global 观察与 frontend owner，包括初始化失败路径。
- [NewSessionViewModelTest](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/new-session/viewmodel/src/commonTest/kotlin/io/github/stream29/kodex/cli/newsession/NewSessionViewModelTest.kt#L15)。
- [DraftAndSettingsBoundaryTest](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/rpc/src/commonTest/kotlin/io/github/stream29/kodex/cli/rpc/DraftAndSettingsBoundaryTest.kt#L23)。
- [RpcSessionViewsTest](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/rpc/src/commonTest/kotlin/io/github/stream29/kodex/cli/rpc/RpcSessionViewsTest.kt#L50)：全部普通 create 改 cwd；本文件 frontend fixture 接入/释放 global；旧一次性草稿测试改为可重复创建，未知 append 回执仍保留实体且不自动重放。
- [RpcSettingsTest](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/rpc/src/commonTest/kotlin/io/github/stream29/kodex/cli/rpc/RpcSettingsTest.kt#L227)：普通 create 迁移；草稿 Settings source 测试改为共享 defaults、后端编号标题和创建后 persisted rename。
- [UltrafastRpcIntegrationTest](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/rpc/src/commonTest/kotlin/io/github/stream29/kodex/cli/rpc/UltrafastRpcIntegrationTest.kt#L17)：先设置真实后端 defaults 再 cwd-only 创建；保留 tier 的 CAS/reopen/fork/建议子任务和诊断断言。
- 以上九个产品文件及本任务文件为 B 的实际写入；未改 build 文件、父任务或 checklist。

### Admission、owner 与设置设计

- `RpcSessionDraft.submit(expectedRevision, open)` 用 `Mutex.tryLock()` 做 admission，不将进行中重复命令排队；拿 gate 后再次捕获并校验 Composer lifecycle/revision/length，才创建。没有通过全局 settings 等待 barrier 限制发送。
- 创建只读本地目录，调用 `createSession(cwd.path)`。`open` 是实际 host 打开端口：VM 传 `sessions::open`，确保稳定 persisted handle 在 append 前已打开；RPC 层仍通过借用的 views 取得同一个真实 binding，不向 app/rpc 引入 NewSession VM 依赖。
- 正常路径为 create → host open → 捕获 binding/Agent → 原文 append → 校验仍为同一活跃 binding/Agent → `agent.resume()` 的同步 admission → `composer.clear(captured.revision)` → cwd CAS reset → 返回。resume 后异步运行仍走 Agent 既有错误出口，不等待模型输出。
- 清空只匹配准确输入 revision；cwd 保存 `(path, revision)`，每次编辑推进 revision，以 compareAndSet 保护成功重置，包含目录 A→B→A 的后续编辑。成功后不关闭单例，下一份有效草稿创建另一 Session。
- settings 仅由 `global.settings.newSession` 的只读投影与本地 cwd combine/stateIn 得到；模型目录直接借用 `global.models`。四字段编辑使用既有完整 `editField` CAS；单次意图冻结目标 baseline，model/effort/tier 为一个 tuple，questions 独立；无关字段合并，目标冲突抛出，不用回执回写本地 settings。
- 自有 Job 管理 Composer、settings projection 与 runtimeConfiguration；parent cancellation/close 释放这些 child。global/views/sessions 从不由单例关闭。
- **Application 交接要求**：child submit 是直接挂起编排，不另建 detached/NonCancellable 正常业务、不新设失败通知。D 必须在进入串行等待前拒绝重复入口，并在存活 command owner 内调用整个 child submit 与导航发布；renderer 仅等待，取消该等待不能取消 accepted command，也不能丢掉无人等待时的异常。child 的 gate/revision 校验不能替代 Application 的队列前 admission。真实错误直接向该 owner 传播。
- 失败不缓存 createdIndex，不做重试/续交/补偿/删除/转移或强制导航；不承诺手动再次调用会复用已有实体。普通 persisted Composer 未改。

### 测试新增/迁移（已写，未执行）

- VM 测试覆盖 factory parent/fixture 关闭、借用资源仍活跃、稳定 runtime child、四字段与 Defaults 页双向同步、persisted 设置不被 defaults 改写、注册表准确 handle、cwd 成功重置和第二次创建。
- RPC 边界测试覆盖零长度/过期 revision、空格/换行/首尾空白原文、create/append 阻塞期间的重复提交、后续文本和 ABA cwd 编辑保护、前端 defaults 订阅暂停时仍使用后端当前四字段、成功不等待模型结束。
- 覆盖存活 frontend command owner 与 renderer waiter 取消的分离、command owner 取消、父级/显式 close 生命周期、异步 resume 失败使用既有错误出口且不恢复已消费输入、create/open/append 失败不自动重放或删除。
- RpcDraftSettingsSource 的旧草稿锁/一次性不可写测试已迁移为 shared-default tuple 冲突与无目标 Unavailable source 断言；未恢复删除接口。保留原有 global/default queue 命令取消测试，以及 RpcSettingsTest/RpcSessionViewsTest 的 persisted CAS、paced CAS 关闭取消、普通 Composer 取消/未知回执等覆盖。RpcOAuthTest 与 SettingsUpdateQueueTest 无改动。
- 测试 fixtures 均用 mock 外部模型；本会话未执行这些 fixture，也未发起 loopback OAuth 或任何模型请求。

### 静态检查与未跑验证

- 已执行 scoped `git -C Kodex diff --check -- <B 的四个范围>`，退出码 0、无 whitespace error；逐文件查看 scoped diff/numstat。
- `rg` 检查 B 的实现、测试及 shared fixture：无旧 materialize、RpcDraftSettingsSource、createdIndex/persistedIndex、草稿命名接口、initialSettings 创建入参或 trim/isBlank/isNotBlank 残留；普通 create 全部只传 cwd/home/root。
- B 的生产 NewSession/RpcSessionDraft 中无 `NonCancellable`；fixture 的 NonCancellable 仅用于资源收尾。未创建临时文件或占用 GUI/远端/构建资源。
- **未跑** Kotlin 编译、`:app-rpc:jvmTest`、`:app-component-new-session-viewmodel:jvmTest`、fixture 消费者单测、跨模块/renderer/Native/PTY 验证。统一调度与实际 task 名核实归父会话；当前不能声称编译或测试通过。

### 范围外待迁移/父级检查

- D：[Application.kt](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/application/src/commonMain/kotlin/io/github/stream29/kodex/cli/app/Application.kt#L107) 仍消费旧 models factory、RpcDraftSettingsSource、默认草稿名/arguments；[ApplicationViewModel.kt](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/application/src/commonMain/kotlin/io/github/stream29/kodex/cli/app/ApplicationViewModel.kt#L194) 仍调用 materialize；[ExactTargetCommandTest.kt](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/application/src/commonTest/kotlin/io/github/stream29/kodex/cli/app/ExactTargetCommandTest.kt#L108) 仍有有名 draft、旧 materialize wrapper/factory/arguments。这些位置未改，待波次 2。
- E：[NewSessionScreen.kt](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/new-session/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/NewSessionScreen.kt#L32) 与 [SessionTreeCliScreen.kt](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/SessionTreeCliScreen.kt#L567) 要切 revision-bound owner 提交入口；[SessionViewModelTestFixture.kt](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/app/SessionViewModelTestFixture.kt#L11) 仍传 `rpc.draft(name)`。组件/根 view 的空白 Ready/Enter 规则及旧命名 tab/recording 测试也须按 E 范围迁移；不能走普通 Composer 的 trim/isBlank 提交路径。
- F：[ApplicationRootBoundaryTest.kt](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/integration-test/src/jvmTest/kotlin/io/github/stream29/kodex/cli/app/ApplicationRootBoundaryTest.kt#L290) 仍为旧 factory/arguments，另在第 182/186 行传完整 settings 创建；本轮仅写了边界测试，未替代 F 的真实 Application/renderer 集成验收。
- 额外普通创建消费者：[BackendHostTest.kt](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/backend-host/src/commonTest/kotlin/io/github/stream29/kodex/backend/host/BackendHostTest.kt#L143)（另第 196 行）、[BackendReconnectTest.kt](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/backend-host/src/jvmTest/kotlin/io/github/stream29/kodex/backend/host/BackendReconnectTest.kt#L93)、[ComposerHostRepairTest.kt](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/agent/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/app/ComposerHostRepairTest.kt#L463) 仍传完整 settings；另见 D 的 [ExactTargetCommandTest.kt](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/application/src/commonTest/kotlin/io/github/stream29/kodex/cli/app/ExactTargetCommandTest.kt#L297)。除 D 的文件外，这些额外路径未在 B 范围，需父级指派迁移测试后端 defaults/cwd，不得用旧 RPC overload 兼容。
- 额外文档遗漏：[AgentViewModel.kt](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/agent/contract/src/specMain/kotlin/io/github/stream29/kodex/app/agent/contract/AgentViewModel.kt#L27) 的 AgentSettingsViewModel KDoc 仍把 New Session 描述为编辑本地 settings draft，需父级分配改为共享 defaults。该路径不在 B 写范围，已停止在此处修改。
- 上述均为已知分波依赖或范围外遗漏，未添加旧 API 转发来掩盖；冻结接口没有自行变更。父级须检查 D 的入口 gate/owner/唯一异常出口及 shutdown 顺序，再统一编译和运行测试。

### 父级检查点

- 主会话已核查本波实现/测试 diff，与冻结接口一致，允许下游 Application 集成；构建与运行门禁尚未通过。
- 原范围外四处创建调用和 AgentSettingsViewModel KDoc 已由主会话迁移；D 的 accepted-work/导航/唯一错误出口仍是下阶段必须验收的职责。
- 主会话已实际执行 `:app-rpc:jvmTest`，107 项，0 失败、0 跳过；含本轮草稿/RPC 边界测试。`new-session-viewmodel` 组件及 shared fixture 的下游 Application 消费者测试仍待 D 集成，不将 RPC 边界用例视为整个 B 的最终验收。证据见父任务 §6.7。

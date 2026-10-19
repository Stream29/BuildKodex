# Task Tree

- `Await foundation review and worker dispatch`()
- `Load accepted foundation interfaces and production assembly`()
- `Implement singleton navigation and accepted submission ownership`()
- `Migrate Application popup, lifecycle and regression tests`()
- `Review scoped diff and report evidence`()
- `Receive parent acceptance`()

# Details

## 父级最终验收

- 已完成导航、提交 owner/唯一异常出口与资源关闭审查；Application 67 项、下游真实集成及 Native/PTY 通过，任务移入 done。返工与证据见父任务 §6.8–6.10。
- 后文“未执行/待验收”为 worker 交付时点，不替代父级最终结果。

## 依赖与范围

- 父任务：[singleton New Session](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-18-singleton-new-session-navigation.md#L1)，第 1、2.3、6 节是冻结规格。
- 工作根 `/home/stream/ACodeSpace/push/BuildKodex`；仅在主会话验收 1A/1B/1C 并派发后开始。
- 独占 `Kodex/app/application/**`（spec、实现、生产 composition、build 和全部单元测试）及本任务文件。
- 不改 B 的 shared fixture、C 的 Settings、E 的根 View 或 F 的 integration；报告跨范围消费者。加载根指导/技能/Draft/父任务和 frontend、VM 状态、RPC Session、生命周期 checklist。
- 不运行 Gradle/Native/IDE Sync、联网模型/远端操作，不提交/推送、不另派代理，保留既存修改。

## 精确目标

- navigation 只包含 `List<PersistedSessionViewModel>` 与 `ApplicationTabSelection.NewSession | SelectedTab(index)`；允许空 tabs，校验重复引用和有效 selected index，派生 `selectedSession`。
- 一个稳定 `val newSession` 由 Application 拥有，启动不创建后端实体，关闭最后一个 tab 只切回该实例；删除 ordinal 和草稿 tab registry 分支。
- `selectNewSession()` 替换 createNewSessionTab；`submitNewSession(expectedRevision)` 返回 persisted 或无操作 null。createApplicationViewModel 的 newSessionArguments 改为单个 `NewSessionViewModelArguments(initialWorkingDirectory)`，保留真实 factory ports。
- 生产 factory 构造使用 B 的 `(views, sessions, global, scope)`，Settings source 只认 RpcPersistedSessionViewModel 或 null→C 的 unavailable source；不要将未知实现 catch 为 null。
- 提交在进入应用 mutex 排队前拒绝进行中重复事件，mutation 边界再次验证 revision。child 成功后只追加/选择返回的 persisted handle，不再关闭/替换单例。
- 已接受编排由存活 frontend owner 完成，caller/renderer 取消只结束等待；普通失败仍走唯一现有错误出口。审查 orphan async failure、不小心用 caller Job、重复通知及被 shutdown 遗漏的子工作，不用 NonCancellable 保活正常业务。
- `selectTab/closeTab/rename` 接受 persisted target。close/delete/archive 后保持存活的选中实例；选中项被移除才夹取后继/末项，空列表回 NewSession；处于 NewSession 时后台变化不切页。
- openCreatedSessions 只追加不选择，openSession 复用同一稳定 handle 并选择，保持正确 popup 所有权。
- nullable Settings target 与 Login returnTo 不隐式重绑定；cwd picker 验证 singleton 或所属真实 rootAgent；关闭无关 tab 不关闭无目标 Settings。
- shutdown/close 正确释放唯一 NewSession、popup 和本地 registry，不 Stop backend，不覆盖断线草稿保留规则。

## 断言与交付

- 迁移多草稿/原位替换测试为单例与联合导航，保留真实 Session 的 exact target、关闭、删除、fork、archive、建议子任务、OAuth 和 cwd 既有覆盖。
- 增加空启动/无 Session 创建、重复提交/旧 revision、第二次有效提交、切换草稿保留、成功重置和部分失败无自动恢复测试。
- 覆盖 renderer caller 取消后完成导航/清空的 accepted-work 语义，以及 shutdown/owner 终结资源；使用真实 RPC fixture、mock 外部模型，不能只写 fake 成功。
- 验证 `newSession.settings` 不随全局默认值变化改写已有 Session；Settings null/关闭目标/Login round trip 不串目标。
- 编写并静态审阅测试，运行留给主会话；不可把依赖迁移产生的暂时编译缺口当成失败兜底理由。

## 交付报告（worker 填写）

- Session ID / storage URI：959，[storage](file:///home/stream/.kodex/sessions/959)。
- foundation 验收来源：父任务 §6.6 已记录主会话对 A/B/C 生产/测试 diff 的静态通过；构建与执行尚未验收。只代表可以按冻结接口集成，不代表要替基础任务声称测试通过。
- 父会话验收：未开始；实现与静态复查已交付，停在 `Await parent acceptance`，未移动 done。

### 实际文件

- [ApplicationNavigationState.kt](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/application/src/specMain/kotlin/io/github/stream29/kodex/app/application/contract/ApplicationNavigationState.kt#L5)：联合 selection、persisted-only tabs、空列表、引用去重与有效 index 校验、派生 `selectedSession`。
- [ApplicationViewModel.kt 契约](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/application/src/specMain/kotlin/io/github/stream29/kodex/app/application/contract/ApplicationViewModel.kt#L36) 与 [ApplicationPopupState.kt](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/application/src/specMain/kotlin/io/github/stream29/kodex/app/application/contract/ApplicationPopupState.kt#L37)：稳定 singleton、revision-bound submit、persisted exact target、nullable Settings/Login 绑定与取消/异常契约。
- [ApplicationViewModel.kt 实现](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/application/src/commonMain/kotlin/io/github/stream29/kodex/cli/app/ApplicationViewModel.kt#L82)：唯一 child、导航、accepted-work owner、错误交接、移除选择规则、cwd/popup 所有权和 shutdown。
- [Application.kt 生产装配](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/application/src/commonMain/kotlin/io/github/stream29/kodex/cli/app/Application.kt#L102)：factory 借用 `RpcGlobalSettings`；单个 cwd arguments；null→Unavailable、RpcPersisted→真实 source、未知实现明确抛错；command handler 接回既有 `reportUnhandledError`。
- [ApplicationFixture.kt](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/application/src/commonTest/kotlin/io/github/stream29/kodex/cli/app/ApplicationFixture.kt#L30)：仅 Application 自己的 fixture，后端 defaults 选 mock 模型；`submitTestSession` 走真实新提交，再等 mock answer 与 idle，避免将 resume 前初始 false 当成完成。
- [ExactTargetCommandTest.kt](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/application/src/commonTest/kotlin/io/github/stream29/kodex/cli/app/ExactTargetCommandTest.kt#L26)：重写多 draft/原位替换断言；gate 只观察真实 child/registry，创建、打开、追加和 resume 仍走 B 的 JSON/RPC fixture。
- 新增 [ApplicationNewSessionTest.kt](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/application/src/commonTest/kotlin/io/github/stream29/kodex/cli/app/ApplicationNewSessionTest.kt#L18)：实际生产 Application factory 的 singleton/defaults/Settings/Login/关闭测试。
- [SessionTabRegistryStateTest.kt](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/application/src/commonTest/kotlin/io/github/stream29/kodex/cli/app/SessionTabRegistryStateTest.kt#L9)、[SessionTreeCliViewModelTest.kt](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/application/src/commonTest/kotlin/io/github/stream29/kodex/cli/app/SessionTreeCliViewModelTest.kt#L15)、[SessionTreeCliSessionDeletionTest.kt](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/application/src/commonTest/kotlin/io/github/stream29/kodex/cli/app/SessionTreeCliSessionDeletionTest.kt#L9)：导航、稳定 handle、catalog、fork/archive/delete、后台追加和 popup 生命周期迁移。
- [SessionActionComponentOwnershipTest.kt](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/application/src/commonTest/kotlin/io/github/stream29/kodex/cli/app/SessionActionComponentOwnershipTest.kt#L8)、[WorkingDirectoryComponentOwnershipTest.kt](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/application/src/commonTest/kotlin/io/github/stream29/kodex/cli/app/WorkingDirectoryComponentOwnershipTest.kt#L11)、[KodexApplicationContextSettingsTest.kt](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/application/src/commonTest/kotlin/io/github/stream29/kodex/cli/app/KodexApplicationContextSettingsTest.kt#L59)、[UnhandledErrorReportingTest.kt](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/application/src/commonTest/kotlin/io/github/stream29/kodex/cli/app/UnhandledErrorReportingTest.kt#L33)：全部旧创建/选择调用迁移，保留 exact child、cwd、迁移与 Hooks/OAuth 所有权覆盖。Hooks 在首次真实提交前配置，避免 bootstrap turn 的延迟通知污染计数。
- 总计 15 个产品源码/测试文件（含 1 个新增）及本任务；build 无需新增依赖。未改 shared fixture、Settings、根 View、integration、checklist、父兄任务或其他产品文件。

### Admission、owner 与唯一异常出口

- [提交入口](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/application/src/commonMain/kotlin/io/github/stream29/kodex/cli/app/ApplicationViewModel.kt#L186) 在 `commandMutex` 前 `submissionGate.tryLock()`，进行中重复直接 null；owner Deferred 完成时解锁，包含尚未开始就被取消的情况。
- mutation 内再次检查 revision/零长度，再调用整个 `newSession.submit(expectedRevision)`；child 返回成功才 append/select。child 自己还会校验 revision，正常完成时清空/cwd reset 仍只由 B 执行。父级不复制设置、不重置第二次、不关闭单例、不做配置 barrier。
- `SupervisorJob(frontend Job)` 是 Application 可取消/可 join 的 command owner；`commands.async` 同时拥有 child submit 和成功导航。caller 只 await，renderer 取消不取消 accepted work，包含尚在应用 mutex 排队的 accepted work。
- **有 waiter**：非取消异常原样从 await 传播，走 renderer 既有 `onOperationFailure`；owner 不另外报告。
- **waiter 被取消**：仅在等待捕获取消且 caller 已不活跃时，安装同一 command owner 下的 `launch { accepted.await() }`；无人等待的 async 失败由继承的 `CoroutineExceptionHandler` 接回生产既有 `reportUnhandledError`。不缓存失败、不新增公开错误状态/事件流/通知消费者。非取消异常路径不在 finally 按 caller 活性二次报告，避免交接竞态下重复通知。
- [shutdown](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/application/src/commonMain/kotlin/io/github/stream29/kodex/cli/app/ApplicationViewModel.kt#L361) 先关闭 admission/取消 command owner、关闭 singleton/popup/tabs，再 join 自有命令/失败观察者，最后在 mutex 内 shutdown registry；不会持 mutex 等自己取消，包含已打开但尚未发布 tab 的 registry handle。frontend parent 取消也触发 close。普通 tab release 不 Stop backend。
- 移除 tab 按仍存活的选中实例重定位；NewSession 态后台移除不切页；选中项被移除才夹取后继/末项。cwd popup 在准入时捕获真实 Session owner，避免 rootAgent 后续重绑定导致关闭 tab 时漏释放；singleton popup owner 为 null，不被无关 tab 关闭。

### 已编写测试（未执行）

- `ExactTargetCommandTest`：
  - `empty startup and repeated plus selection never allocate or reconstruct the singleton`
  - `duplicate admission rejects before the application mutex and revalidates a queued revision`
  - `in-flight duplicates create once then a second valid draft creates another Session`
  - 三个 `Application submits nonzero ... text unchanged through real RPC`，直接检查真实 index 的首条 `StableUserMessage`。
  - `late success publishes the captured Session but preserves newer input and ABA cwd edits`
  - `renderer wait cancellation during accepted work still resets and publishes navigation`
  - `accepted submission survives renderer cancellation even while queued for the mutex`
  - `opening failure after allocation propagates unchanged without retry reset or navigation`
  - `orphan async opening failure uses the unique owner error outlet exactly once`
  - `shutdown cancels and joins accepted work closes singleton and popup without Stop or notification`
  - `shutdown releases a created registry handle even when submission never published its tab`
  - `frontend owner termination cancels orphan work and closes popup and singleton`
  - 另保留/迁移 queued moved target、removed/stale target、同名同 index equality-equal foreign handle、普通 caller 排队取消、未准入取消与 stale Agent/cwd 拒绝。
- `ApplicationNewSessionTest` 五项生产装配断言：空启动无实体、切换文本/cwd/child 保留及成功重置、null Settings/Login 不重绑定、四字段 defaults 双向共享且不改既有 Session、shutdown 释放全部 child 且 mock backend turn 未被 Stop。
- `SessionTabRegistryStateTest` 新增空列表/无效 index/引用重复与 equality-equal 不同实例；close/delete/archive 覆盖选中实例保留、后继/末项和 NewSession 态后台移除。
- `SessionTreeCliViewModelTest` 保留稳定 registry handle、unarchive/catalog/exact popup/OAuth、child 更新不重发 navigation、fork/openCreatedSessions 去重；补 NewSession 态后台追加不选中且保留输入。
- 原 `SuggestedWorkingDirectoryAdapterTest` 及其他无旧 API 的 Application 单测保持原样；未删除建议调用替换/提交中拒绝、反馈/配置保留、MCP、sidebar、settings、资源清理等现有覆盖。

### 静态检查与未执行验证

- 已多次执行 `git -C Kodex diff --check -- app/application`，退出 0；本任务文件同样检查无 whitespace error。新增测试额外检查尾空白、末尾换行（`0a`）。
- `rg` 扫描整个 `app/application/src`：无旧 `SessionViewModel` 整词、createNewSessionTab/materialize(NewSession)、selectedIndex/.selected、ordinal/defaultName、RpcDraftSettingsSource、旧命名 arguments 或旧 factory models 参数；生产 NewSession factory 仅创建一次。
- 逐段静态复查生产 diff、全部受影响测试调用、错误分流、gate 释放及 shutdown 先取消再 join 的顺序；没有用 `NonCancellable` 包正常业务。生产已有 NonCancellable 仅位于 host finally cleanup，测试仅用于 fixture 收尾。
- `gradle.properties` SHA-256 为 `a73fd71eaa67ae5c40edf81a8738bdaadbab7b8549566772638101f980228f8c`，与父级 [冻结记录](file:///home/stream/ACodeSpace/demo/kodex-new-session-948/baseline/gradle-properties-sha256.txt#L1) 一致。A/B/C 代码未由本 Session 编辑。
- 只读识别到 IDEA 打开 PersonalSkills；未操作窗口或 Sync。没有临时文件、持有的 GUI/远端/构建资源需要释放。
- 未提交/推送、未另派 worker、未操作远端/GUI 或线上模型；所有写入使用 apply_patch。
- **未运行** Gradle、Kotlin 编译、`:app-application:jvmTest`、任何单测/集成/Native/PTY/模型调用。静态检查不是编译或测试通过结论；实际验证与失败归因归父会话。

### 范围外交接

- E：[SessionTreeCliScreen.kt:576](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/SessionTreeCliScreen.kt#L576) 迁移 revision-bound submit 时，**live waiter 必须使用既有 `reportOperationFailure(onOperationFailure)` 一次**；该 helper 继续传播 cancellation，不能再给 accepted command 额外无条件上报。无人等待的失败已由 D 的 owner handler 接管。
- E：[SessionTreeCliScreen.kt:140](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/SessionTreeCliScreen.kt#L140)、[SessionTabRenderState.kt:9](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/SessionTabRenderState.kt#L9)、[HistoryRenderState.kt:9](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/HistoryRenderState.kt#L9) 仍为旧导航/共同父接口。`app/view/src/mosaicTest` 的 `SessionTabBarTest.kt`、`SessionTabBarTestAdapter.kt`、`SettingsLoginLifetimeTest.kt` 及 `app/view/src/jvmTest` 的 `Docs{Completion,SessionManagement,Workspace,HistoryInvalidation}RecordingTest.kt` 尚有旧消费者；仅需迁移源码，不是执行录屏授权。
- E：[NewSessionScreen.kt:37](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/new-session/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/NewSessionScreen.kt#L37) 的零参数 onSubmit 与旧替换 tab KDoc、[SessionTabBar.kt:75](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-tab-bar/view/src/specMain/kotlin/io/github/stream29/kodex/app/sessiontabbar/contract/SessionTabBar.kt#L75) 的 `createNewSession` 以及其 mosaic 实现第 137 行仍待冻结回调迁移。
- F：[ApplicationRootBoundaryTest.kt:290](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/integration-test/src/jvmTest/kotlin/io/github/stream29/kodex/cli/app/ApplicationRootBoundaryTest.kt#L290) 仍传 models 构造 factory、第 298 行仍是命名 arguments lambda，需改 global/单个 cwd arguments；也需按本报告的 live/orphan 失败交接提供 owner handler 并覆盖真实 renderer。
- 没有恢复旧接口兼容 overload，也没有接手父级已处理的 backend-host/agent-view/KDoc。上述是未派发 E/F 的预期迁移缺口；本轮不声称全仓可编译或验收通过。

## 父级验收检查点

- 主会话已完成生产及测试静态审查；首次实际构建发现本文件新增测试四处误用 Login.isActive 布尔属性，Application 测试未能编译。该 API 实际按 attemptId 查询，不能当组件生命周期标志。
- 主会话将断言修正为观察公开 effects Flow 的关闭：无关 tab 关闭不终止、dismiss/shutdown 终止，无须启动 OAuth；未修改生产代码或删除生命周期覆盖。另修复父级自己的 backend-host 模型断言，详情见父任务 §6.8。
- 六模块复跑已通过，Application 为 67 项，0 失败、0 跳过；包含真实提交、renderer waiter 取消、排队 admission、孤立失败唯一上报及 shutdown 等用例。当前放行 E/F 按现有接口实施，Native/renderer/端到端仍待父级最终验收，不移 done。

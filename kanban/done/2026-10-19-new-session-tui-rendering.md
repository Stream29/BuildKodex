# Task Tree

- `Await Application review and worker dispatch`()
- `Inspect frozen navigation and renderer consumers`()
- `Implement selected plus button and singleton screen routing`()
- `Migrate renderer state, focus and scoped tests`()
- `Review scoped diff and report evidence`()
- `Receive parent acceptance`()

# Details

## 父级最终验收

- 已通过范围审查、四个相关组件 47 项、根界面 55 项、macOS Arm64 编译与独立 Native PTY 交互，任务移入 done。焦点/测试返工、准确计数及清理证据见父任务 §6.9–6.10。
- 下文 worker 的“未运行/待验收”说明保留交付时点；相邻 Agent 的既有 steer 超时仍单独记录，不混作本任务通过。

## 依赖与范围

- 父任务：[singleton New Session](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-18-singleton-new-session-navigation.md#L1)，重点第 2.4、4、6 节。
- 根 `/home/stream/ACodeSpace/push/BuildKodex`；波次 2 已通过父级静态审查与六模块复跑，本 Session 获用户授权实施第三波 E。
- 独占 `Kodex/app/view/**`、`Kodex/app/component/new-session/view/**`、`Kodex/app/component/session-tab-bar/view/**`，含 build、相关测试与已有 recording 测试的 API 迁移；不执行录屏场景。
- 不修改 Application/ViewModel/Settings/shared fixture/integration-test，主会话拥有 checklist。加载根指导、技能、Draft、父任务和 TUI/frontend/VM 状态规范。
- 不运行 Gradle/Native/IDE Sync、远端或真实账号/模型，不提交/推送、不另派 worker；保留既存修改。

## 冻结接口与行为

- tabs 全是 persisted，selection 为 NewSession 或 SelectedTab(index)，selectedSession 派生可空；newSession 是 Application 稳定属性，不在 tabs。
- `SessionTabBarState.selected == null` 表示 NewSession：右端 `+` 用与普通 selected tab 相同的 Invert/highlight，其他 Session tab 不选中。callback `selectNewSession()` 替换 createNewSession。
- New Session 不能被重命名、关闭、归档或作为普通 tab 上下文菜单目标；persisted tab 原操作保持。
- 根 Screen 按 selection 分支；NewSession Settings 传 null/NewSession page，cwd picker 传 singleton；persisted Settings 传准确 handle/CurrentSession。
- `NewSessionScreen`/`NewSessionContent` 的 `onSubmit: (expectedRevision: Long) -> Unit`；事件携带输入组件确认后的准确 revision，传 Application.submitNewSession。不要让异步回调执行时再读取后来的 revision。
- 输入状态引用 singleton Composer；切换重挂载保留原文/光标/cwd；长度 0 不创建，非零长度空白有效，不在 View 层偷偷 trim 或 disable。
- 更新 SessionTabRenderState、HistoryRenderState、tab identity registry、终端标题计数等全部 typed 消费者；计数只算真实 Session，不能把 `+` 算成一个 Session。
- 不重做 persisted Agent Composer 语义、History 虚拟化、sidebar layout 或焦点协议；renderer state 不变成 VM 的第二份数据真源。

## 检查项

- 空 tabs 下顶栏/Sessions/+ 可达；NewSession 和普通标签的互斥高亮、点击/键盘切换、菜单、overflow/横向滚动、窄终端不越界。
- NewSession 切走再回来保留输入/光标，成功后返回 NewSession 显示已清空/重置；旧回调不吞掉新草稿。
- 测试真实 settings 入口页/null target，关闭后台 tab 不改变 NewSession 选中。
- 迁移 app/view 的 recording 测试旧调用但不执行真实录制/线上模型；保留对应场景断言。实际交互由主会话统一验证。
- 只做静态检查，测试运行由主会话串行组织；不得因 API 删除而简单移除整个测试文件。

## 交付报告

- Session ID / storage URI：961，[storage](file:///home/stream/.kodex/sessions/961)。
- Application 接口验收来源：父任务 §6.8；D 已静态审查、六模块 134 项复跑通过（Application 67 项），仅修复测试用法，生产冻结接口未变。其他四模块先前 417 项通过。
- 本轮修改 23 个产品/测试/build 文件及本任务文件，全部在授权写范围内；未修改 shared fixture 模块、父兄任务或 checklist。
- 父会话已完成静态审查、JVM、Native 与 PTY 验证，并修复挂载焦点和测试时钟/帧编排；记录见父任务 §6.9。下方 worker 的“未运行”说明不是最新父级状态。

### 实际文件与交互

- [NewSessionScreen.kt](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/new-session/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/NewSessionScreen.kt#L31)：Screen/Content 均为 `(expectedRevision: Long) -> Unit`。发送事件同步以 widget 的 text/cursor 调用 Composer.update，取得返回的确认 revision 后立即传回 host；不读滞后 composition revision，不在异步 Application 调用时重新读取 revision。借用同一个 Composer，重挂载从其当前文本/光标初始化；增加局部裁剪保护极窄/零高度内容区域。
- [Tab Bar 契约](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-tab-bar/view/src/specMain/kotlin/io/github/stream29/kodex/app/sessiontabbar/contract/SessionTabBar.kt#L34) 与 [renderer](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-tab-bar/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/sessiontabbar/SessionTabBar.kt#L60)：null selection 明确选择 singleton；`+` 使用同一 TuiButton selected/Invert 规则，callback 为 selectNewSession。保留普通 tab 的 exact identity、菜单、滚动与 selected bounds；不赋予 `+` 普通 tab identity 或 secondary action。
- 顶栏自然最小宽度为 16 列（15 列 controls 加至少 1 列 tab viewport）。更窄时由外层 horizontalScroll 裁剪和分页，只有这种窄布局增加一个 Mosaic focusable 滚动表面，避免焦点控件被裁掉后无法继续分页；常规宽度不增加焦点目标、不更改原 tab 横向滚动规则。1/8/14/15/16/32 列均有待执行测试。
- [SessionTreeCliScreen.kt 导航投影](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/SessionTreeCliScreen.kt#L140) 与 [主体路由](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/SessionTreeCliScreen.kt#L564)：按 ApplicationTabSelection 分支；NewSession 使用 viewModel.newSession、null/SettingsPage.NewSession 和 singleton cwd target；persisted 分支使用同一导航快照派生的 selectedSession、准确 rootAgent 与 CurrentSession。
- [提交错误出口](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/SessionTreeCliScreen.kt#L574)：live waiter 只调用既有 reportOperationFailure 一次；helper 未改，CancellationException 继续传播。没有为 Application 的 orphan command 增加第二次通知、无条件 catch/finally 上报或 renderer-owned 业务 owner。
- [SessionTabRenderState.kt](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/SessionTabRenderState.kt#L18)、[HistoryRenderState.kt](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/HistoryRenderState.kt#L17)、[TerminalTitle.kt](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/TerminalTitle.kt#L12)：全部 tabs/identity/menu target 收窄到 persisted，删除草稿 filter/cast 分支；空 tabs 计数为 0，`+` 不计数。History 仅保留准确 child 的 renderer scroll state，不复制业务窗口。
- [app/view/build.gradle.kts](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/view/build.gradle.kts#L60)：仅新增 mosaicTest 的 app-settings-filesystem 依赖；本地 root 测试与迁移的 recording 源码在私有 Home 明确设置 mock 模型、禁用自动标题，避免首次真实提交影响原场景。没有生产依赖或既存 gradle.properties 修改。
- [本地 SessionViewModelTestFixture.kt](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/app/SessionViewModelTestFixture.kt#L9)：仅 app/view 内的测试 owner，持有一个 rpc.draft() 单例，移除 name 参数；普通 tab 测试改用真实 rpc.create。未改 app/test-support/rpc，未用兼容 wrapper 伪造多个 NewSession。
- recording 源码迁移保留原场景与文件：[DocsCompletionRecordingTest.kt](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/view/src/jvmTest/kotlin/io/github/stream29/kodex/cli/app/DocsCompletionRecordingTest.kt#L86)、[DocsSessionManagementRecordingTest.kt](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/view/src/jvmTest/kotlin/io/github/stream29/kodex/cli/app/DocsSessionManagementRecordingTest.kt#L63)、[DocsHistoryInvalidationRecordingTest.kt](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/view/src/jvmTest/kotlin/io/github/stream29/kodex/cli/app/DocsHistoryInvalidationRecordingTest.kt#L99)、[DocsWorkspaceRecordingTest.kt](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/view/src/jvmTest/kotlin/io/github/stream29/kodex/cli/app/DocsWorkspaceRecordingTest.kt#L95)、[DocsHistoryRecordingTest.kt](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/view/src/jvmTest/kotlin/io/github/stream29/kodex/cli/app/DocsHistoryRecordingTest.kt#L196)。创建改为 composer revision 提交后才 rename persisted；建议任务 prompt 只发送一次；多标签溢出场景使用真实 persisted Agent screen，历史失效切到 singleton。没有删除整个测试文件，也没有执行录屏。

### 已编写或迁移的测试（未编译、未执行）

- [Tab Bar contract tests](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-tab-bar/view/src/commonTest/kotlin/io/github/stream29/kodex/app/sessiontabbar/contract/SessionTabBarContractTest.kt#L7)：`empty tabs and null selection represent the singleton New Session`，保留重复 identity/无效 selected 校验。
- [Tab Bar renderer tests](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-tab-bar/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/sessiontabbar/SessionTabBarTest.kt#L29)：
  - `plus and ordinary selected tab use mutually exclusive reverse video`。
  - `empty tabs retain Sessions and plus pointer and keyboard actions`。
  - 六个 `empty top bar clips to <width> columns and can page to both controls`。
  - 保留排序、running 截断、准确 pointer/secondary identity 和 stale callback 拒绝。
- [NewSessionScreenTest.kt](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/app/NewSessionScreenTest.kt#L26)：
  - `unmounting and returning retains the singleton Composer cursor and cwd`，重挂载后插入字符验证准确光标。
  - `submitCallbackRetainsTheExactComposerRevision`，直接断言回调参数。
  - `back to back input and send confirm revision despite a stale collected projection`，强制 projection 滞后，输入/发送不等重组，验证捕获值不会变成后续编辑的 revision。
  - 四个 raw-text 事件测试覆盖空串、纯空格、纯换行及首尾空白；空串由 owner 判零长度，不在 View 禁发。
  - `successful submit remounts the same cleared singleton and rejects a late revision`，真实 RPC child 成功后检查 composer identity、空文本/光标和 cwd reset；新草稿不被旧 revision 消费。
  - 保留空页、bounded rows、cursor echo、undo 与外部更新断言。
- [SettingsLoginLifetimeTest.kt](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/app/SettingsLoginLifetimeTest.kt#L40)：
  - `New Session Settings has no target and background close keeps its default page and owner`：真实点击 Settings、默认 NewSession、Unavailable、后台关闭不切页，以及 Current session 的 No selected session。
  - `root routes singleton cwd picker and exact persisted Settings without rebinding`。
  - `root passes the synchronous revision through a delayed submit and remounts its cleared singleton`：真实 root callback 延迟后验证过期拒绝、有效下一次提交、切回与迟到 callback 保留新草稿。
  - `root live New Session submit reports an ordinary failure once` 和 `root New Session cancellation propagates without an operation failure notification`。
  - 原 identity churn 改成真实 persisted Session；保留 MCP/Login owner、gated opener 与 root 卸载生命周期场景。没有把 Login.isActive(attemptId) 当布尔属性。
- [SessionTabBarTest.kt](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/app/SessionTabBarTest.kt#L74) 与 [adapter](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/app/SessionTabBarTestAdapter.kt#L17)：重命名场景改为 persisted；保留 overflow Page keys/native wheel、选中自动入 viewport、primary/secondary target 断言，callback 统一 selectNewSession。
- [SessionTabContextMenuTest.kt](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/app/SessionTabContextMenuTest.kt#L86)：旧草稿菜单改为 `singletonPlusHasNoOrdinaryTabContextMenu`，验证 secondary/Menu/Shift+F10 不出现菜单、不会 close，普通 persisted 菜单断言保留。
- [SessionViewportTest.kt](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/app/SessionViewportTest.kt#L21) 只迁移单例 fixture 调用；[TerminalTitleTest.kt](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/app/TerminalTitleTest.kt#L12) 去除伪草稿 tab，新增空列表计数，保留 OSC/count update/cleanup。

### 静态检查、范围与父级待验收

- `git -C Kodex diff --check -- app/view app/component/new-session/view app/component/session-tab-bar/view` 及本任务文件 diff check 均退出 0。
- 对授权目录全部 Kotlin 源码扫描：无整词 SessionViewModel、createNewSessionTab/materializeNewSession/createNewSession、onCreateNewSession、旧 navigation.selected/selectedIndex、NewSessionDraft/RpcDraftSettingsSource 或带 name 的 rpc.draft 调用。只读扩展扫描 app/integration-test 同一旧 API 集合也无命中；这不是全仓编译结论。
- 逐段审阅生产 diff、回调的同步/异步边界、persisted 精确类型、空列表与窄布局，以及迁移测试/recording 调用；NewSession View 路径没有 trim/isBlank 判空或调用普通 Composer.submit，未改普通 persisted Composer。
- 既存 gradle.properties SHA-256 仍为 `a73fd71eaa67ae5c40edf81a8738bdaadbab7b8549566772638101f980228f8c`，与父级冻结记录相同。
- 所有写入使用 apply_patch；未提交/推送、未另派 worker、未操作 GUI/远端或真实账号/模型。只读识别本地 IDEA 进程，未操作窗口或触发 Sync；没有创建临时文件或持有构建/GUI/远端资源。
- **未运行** Gradle、Kotlin 编译、单测/集成测试、Native、独立 PTY 或任何 recording 场景。新增测试只表示覆盖意图，不能计作通过；既有 551 项仅引用父级先前门禁，不包括本轮 UI。
- 未发现需要变更冻结接口或越界修改的阻塞；没有接手 F 的 integration-test。父级仍需集中执行 renderer/assembly 测试、跨模块集成、Native 与独立 PTY 窄终端/焦点验收，并验证 recording 源码编译但不启用录制。

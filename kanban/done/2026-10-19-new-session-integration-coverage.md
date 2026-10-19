# Task Tree

- `Load accepted Application contracts and task boundaries`()
- `Map end-to-end checks to real backend and renderer fixtures`()
- `Migrate existing integration consumers and add regressions`()
- `Review test validity and report runnable evidence`()
- `Receive parent acceptance`()

# Details

## 父级最终验收

- 两个安全 suite 共 31 项实际通过；父级另完成 Native frontend/本地 mock backend 的独立 PTY 22 项检查，补齐本任务交付时声明的真实交互缺口，任务移入 done。证据见父任务 §6.9–6.10。
- 下文“未执行/待验收”为 worker 交付时点，不代表最终结果，也不宣称 integration-test 全仓/全平台通过。

## 依赖与范围

- 父任务：[singleton New Session](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-18-singleton-new-session-navigation.md#L1)，第 4、6 节为验收清单。
- 仓库根 `/home/stream/ACodeSpace/push/BuildKodex`；波次 2 静态验收后，与 E 独立派发。
- 唯一写范围 `Kodex/integration-test/**` 与本任务文件；产品修复、shared fixture、app/view tests、checklist 不属本任务。发现缺口报告给主会话，不能跨范围补丁或更改产品期待。
- 加载根指导、相关技能、Draft、父任务及 frontend/RPC/TUI/资源生命周期 checklist。
- 不能运行 Gradle、Native、IDE Sync、真实账号/模型、远端/GUI 或录屏，不提交/推送、不派 worker。

## 必须覆盖的链路

- 实际 backend＋JSON/kRPC＋frontend/Application，新建不通过手工插入假的 persisted handle。外部模型/API 使用现有 mock，隔离临时 Home 并完成清理。
- 空启动不创建实体；`+` 与真实标签切换保留 singleton composer/cwd，真实标签计数不包含 NewSession。
- 默认值由后端四字段与前端请求 cwd 合成；双向编辑显示，但创建不依赖前端 settings 已追平。不为配置竞态构造新同步需求。
- 首条消息零长度不创建；纯空格、换行、首尾空白逐字保留；首次成功后选择新 Session、准确清空/重置；第二份有效输入创建另一个 Session。
- 同 revision 重复/迟到回调不多建、不发送到后来选中的 Session；renderer caller 取消不撤销 frontend owner 已接受编排。
- null Settings 默认 NewSession page，CurrentSession unavailable；切普通标签再开 Settings 绑定准确真实目标，Login returnTo 不串目标。
- 关闭/删除/归档最后或后台标签的选择规则、单例身份与草稿保留；关闭本地 tab 不 Stop backend 工作。
- 创建/追加异常不自动重试/补偿；异步运行失败不恢复已消费草稿。不要添加“失败必须入列跳转”“重试保证不重复创建”的断言。

## 现有测试与缺口

- 迁移 `ApplicationRootBoundaryTest.kt` 的旧 create/materialize/navigation 与 nullable Settings 消费；fixture 或 RPC 设置 API 的既有漂移需分清与本任务相关性，记录原始行和最小修正，不掩盖原断言。
- 需要 E 未完成的 renderer API 时按父任务冻结签名写测试，不修改 E 的文件；缺少可安全执行入口则报告，而非引入生产测试开关。
- 对整个第 4 节给出“用例名/文件/测试层级”映射，区分单元已覆盖、端到端新增和仍待真实 PTY 的项目。
- 编写测试但不执行；主会话统一选取本任务 suite，不跑真实账号、real IO、performance 或录屏测试。

## 交付报告

- Session 962，[storage](file:///home/stream/.kodex/sessions/962)。
- 已按本轮明确派发实施；只写下列 3 个 integration 文件及本文件，全部使用 apply_patch。
- **只完成编写和静态审查；未编译、未运行任何测试。** 父级既有 551 项不是本任务结果。本任务停在 `Await parent acceptance`，不移 done。

### 文件与真实链路

- [ApplicationRootBoundaryTest.kt](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/integration-test/src/jvmTest/kotlin/io/github/stream29/kodex/cli/app/ApplicationRootBoundaryTest.kt#L58)：
  - 迁移全部旧 `.selected`、普通创建参数、NewSession factory/arguments 消费；History 切到旧草稿 tab 的断言改为选择稳定 singleton，保留 Agent 文本/revision/实例、失败次数、RPC 次数与后端实体数量断言。
  - 原 8 项 History、catalog、equality-equal/exact-target 删除覆盖全部保留；未删用例或添加旧 API 兼容。
  - [本地 fixture](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/integration-test/src/jvmTest/kotlin/io/github/stream29/kodex/cli/app/ApplicationRootBoundaryTest.kt#L252) 新增可选空 Home、外部模型响应、defaults 订阅 gate；Settings/Login/directory 使用真实组件工厂，nullable target 只走 unavailable source，非空目标解包原有 BoundarySession 后绑定真实 RpcSessionSettingsSource。
  - BoundarySession 仍只包真实 registry.open 的结果，保留原相等实例/History 校验故障注入；不制造 persisted 成功结果。
- [NewSessionIntegrationTest.kt](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/integration-test/src/jvmTest/kotlin/io/github/stream29/kodex/cli/app/NewSessionIntegrationTest.kt#L27)：新增 suite，参数展开后预期 23 项，见下表。
- [integration-test/build.gradle.kts](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/integration-test/build.gradle.kts#L50)：显式声明 fixture 使用的 migration、path-picker、session-settings ViewModel 依赖；未改任务开关/默认禁用配置。
- 链路是 `prepareKodexHome → withBackendServices → withInMemoryRpc(JSON/kRPC) → RestoringRpcClient → RpcSessionViews/真实 registry/真实 NewSession → createApplicationViewModel → SessionTreeCliScreen`。
- 仅外部 OpenAI/Login API 是 mock；RPC decorator 只计数、延迟或注入故障，正常返回值均委托真实 RPC。Settings 订阅 gate 只延迟真实 Flow，不伪造配置；它不进入生产，也不增加配置 barrier。

### 新 suite 用例索引（均未执行）

| ID | 展开数 | 精确用例名 / 文件位置 | 层级 |
| --- | ---: | --- | --- |
| N1 | 1 | [empty root plus and zero-length Enter never allocate a Session](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/integration-test/src/jvmTest/kotlin/io/github/stream29/kodex/cli/app/NewSessionIntegrationTest.kt#L29) | 根 renderer＋Application＋RPC/backend |
| N2 | 1 | [root tab and plus preserve singleton text cursor cwd and persisted-only count](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/integration-test/src/jvmTest/kotlin/io/github/stream29/kodex/cli/app/NewSessionIntegrationTest.kt#L48) | 根 renderer，含实际 ANSI reverse-video 互斥检查 |
| N3 | 3 | [root Enter submits {spaces,newlines,padded} verbatim selects resets and permits a second Session](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/integration-test/src/jvmTest/kotlin/io/github/stream29/kodex/cli/app/NewSessionIntegrationTest.kt#L78) | 根 Enter→revision callback→真实首条 StableUserMessage |
| N4 | 1 | [paused defaults projection does not delay creation with backend four fields and requested cwd](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/integration-test/src/jvmTest/kotlin/io/github/stream29/kodex/cli/app/NewSessionIntegrationTest.kt#L122) | Application＋真实 backend/JSON，前端订阅有意落后 |
| N5 | 1 | [Settings defaults and singleton edit one backend source without changing an existing Session](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/integration-test/src/jvmTest/kotlin/io/github/stream29/kodex/cli/app/NewSessionIntegrationTest.kt#L163) | Application＋真实 Settings children＋RPC/backend |
| N6 | 1 | [in-flight duplicate and late root revision never create twice or send to a later selected Session](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/integration-test/src/jvmTest/kotlin/io/github/stream29/kodex/cli/app/NewSessionIntegrationTest.kt#L198) | 根 renderer＋create 回执 gate；检查创建后/追加前不清空或重置 |
| N7 | 2 | [root renderer cancellation preserves accepted owner work and {exact reset,newer edits}](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/integration-test/src/jvmTest/kotlin/io/github/stream29/kodex/cli/app/NewSessionIntegrationTest.kt#L258) | 真正卸载整个根 renderer，Application owner 继续；新版文本/cursor/ABA cwd 保护 |
| N8 | 5 | [root {createSession,create-reply,keepSessionAlive,appendUserMessage,append-reply} failure reports original once without retry transfer compensation or navigation](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/integration-test/src/jvmTest/kotlin/io/github/stream29/kodex/cli/app/NewSessionIntegrationTest.kt#L313) | 根 renderer＋真实 RPC 故障/回执丢失边界 |
| N9 | 1 | [orphaned root submit failure reaches the Application owner handler exactly once](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/integration-test/src/jvmTest/kotlin/io/github/stream29/kodex/cli/app/NewSessionIntegrationTest.kt#L376) | 卸载根 renderer 后的 owner 错误交接 |
| N10 | 1 | [asynchronous model failure does not restore consumed draft or remove the successful tab](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/integration-test/src/jvmTest/kotlin/io/github/stream29/kodex/cli/app/NewSessionIntegrationTest.kt#L423) | Application＋真实 backend run，外部 mock provider 抛错 |
| N11 | 1 | [root no-target Settings and Login returnTo remain unavailable through tab selection and close](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/integration-test/src/jvmTest/kotlin/io/github/stream29/kodex/cli/app/NewSessionIntegrationTest.kt#L467) | 根 Settings 入口/Unavailable 呈现＋真实 Login effects 生命周期 |
| N12 | 1 | [root persisted Settings and Login retain the exact target rather than the latest selected tab](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/integration-test/src/jvmTest/kotlin/io/github/stream29/kodex/cli/app/NewSessionIntegrationTest.kt#L508) | 根 Settings 入口＋真实目标编辑/关闭/Login returnTo |
| N13 | 3 | [{close,delete,archive} background selected and last real tabs preserve navigation singleton and draft](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/integration-test/src/jvmTest/kotlin/io/github/stream29/kodex/cli/app/NewSessionIntegrationTest.kt#L545) | Application＋真实 catalog/registry/backend |
| N14 | 1 | [closing the last local tab and frontend shutdown never Stop a running backend turn](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/integration-test/src/jvmTest/kotlin/io/github/stream29/kodex/cli/app/NewSessionIntegrationTest.kt#L589) | Application＋真实运行/关闭/清理；统计实际 cancelRunningTurn RPC |

### 父任务 §4 全项映射

- 下表“已有”表示源码覆盖及父级 §6.7/6.8 的既有验收范围，不把组件单测称为根集成，也不将 E 正在编写的 renderer 测试称为已通过。
- 既有文件简称：
  - A = [ApplicationNewSessionTest.kt](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/application/src/commonTest/kotlin/io/github/stream29/kodex/cli/app/ApplicationNewSessionTest.kt#L18)（Application 模块测试，真实生产装配）。
  - X = [ExactTargetCommandTest.kt](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/application/src/commonTest/kotlin/io/github/stream29/kodex/cli/app/ExactTargetCommandTest.kt#L26)（Application owner/admission 测试，真实 RPC child）。
  - R = [DraftAndSettingsBoundaryTest.kt](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/rpc/src/commonTest/kotlin/io/github/stream29/kodex/cli/rpc/DraftAndSettingsBoundaryTest.kt#L167)（RPC/Draft 层）。
  - T = [SessionTabBarTest.kt](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-tab-bar/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/sessiontabbar/SessionTabBarTest.kt#L28)（E 的组件 renderer，尚待父验收）。
  - V = [NewSessionScreenTest.kt](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/view/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/app/NewSessionScreenTest.kt#L26)（E 的 renderer/Composer 边界，尚待父验收）。

| §4 顺序 / 检查项 | 已有用例名 / 层级 | 本任务集成新增 / 剩余门禁 |
| --- | --- | --- |
| 1 空启动不创建 | A: `production startup and selecting plus keep a backend-free stable singleton` | N1 |
| 2 四字段共享双向编辑 | A: `production defaults and singleton edit the same four fields without mutating persisted Session` | N5；实际状态栏编辑呈现另由 E 的 NewSessionStatusBarTest 验收 |
| 3 切真实 tab / `+` 保留 cwd、草稿 | A: `switching away and back retains text cwd and children then success resets the same singleton`；V: `unmounting and returning retains the singleton Composer cursor and cwd` | N2，直接检查 Composer 全 state（含 cursor/revision）；真实终端物理光标仍待 PTY |
| 4 `+` / 普通 tab 互斥高亮 | T: `plus and ordinary selected tab use mutually exclusive reverse video` | N2 检查真实根 ANSI spans，PTY 仍待 |
| 5 长度 0 / 三种空白原文 | R: `zero length and stale revisions never create or open a Session`、`nonzero $label text is appended byte-for-byte` | N1、N3（三项），直接读取真实 StableUserMessage.content |
| 6 重复、迟到 revision 与第二份有效草稿 | X: `in-flight duplicates create once then a second valid draft creates another Session`；V: `submitCallbackRetainsTheExactComposerRevision` | N3、N6；不向后来选中的旧真实 Session 追加 |
| 7 成功选择/准确清空/cwd reset，不提前重置 | R: `normal success resets only after resume admission and does not wait for model completion` | N3、N6、N7；N6 在 backend 已创建、尚未 append 时检查原文/cwd |
| 8 stale 前端默认值 / 既有 Session / 建议与 fork | R: `cwd-only creation uses backend defaults even while the frontend defaults subscription is paused`；[BackendSessionManagementTest.kt](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/server/src/commonTest/kotlin/io/github/stream29/kodex/rpc/server/BackendSessionManagementTest.kt#L44): `creation captures current backend defaults once and preserves prior settings title and identity semantics`、`suggested batch returns ordered metadata and launches children without waiting for model completion` | N4、N5；建议显式配置/fork 保留已有后端模块覆盖，未另造前端同步需求 |
| 9 创建/打开/追加失败，不重放/转移/补偿 | X: `opening failure after allocation propagates unchanged without retry reset or navigation`；R: `$phase failure propagates without reset replay deletion or automatic resume` | N8 五项、N9；create/append 回执丢失时保留已真实发生的数据 |
| 10 resume 异步失败不恢复已消费输入 | R: `admitted asynchronous resume failure uses the existing error outlet and cannot restore consumed input` | N10：实际 provider failure 穿过 backend，得到既有 AgentRunFailureReported；不凭空等待从未发布的 Agent notification |
| 11 renderer 取消/迟到完成保护 | X: `renderer wait cancellation during accepted work still resets and publishes navigation`、`late success publishes the captured Session but preserves newer input and ABA cwd edits` | N7 两项、N9，真正取消 Mosaic root 的 waiter；owner handler 只接 orphan failure |
| 12 最后 tab 关闭回 NewSession | [SessionTreeCliSessionDeletionTest.kt](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/application/src/commonTest/kotlin/io/github/stream29/kodex/cli/app/SessionTreeCliSessionDeletionTest.kt#L9): `delete preserves the selected surviving instance then chooses successor and last item` | N13（选中态最后 tab 与 NewSession 态后台最后 tab 均覆盖）、N14 |
| 13 后台移除保留实例 / NewSession / 批量打开不选中 | 同上：`delete preserves the selected surviving instance then chooses successor and last item`、`background delete and archive preserve New Session selection and unsent text` | N13 三项；原 root equality-equal survivor 测试仍保留 |
| 14 Settings NewSession 默认页 / unavailable | A: `no-target Settings stays unavailable through later selection close and Login returnTo` | N11 从真实根按钮进入并检查 `No selected session`；N12 检查真实 tab 的默认页和准确目标 |
| 15 null Settings/Login 不重绑定、shutdown 释放 | A: 上项及 `shutdown releases singleton and no-target Login and Settings without stopping backend turn` | N11、N12、N14；以 effects Flow 关闭验证，不调用 OAuth、不误用 isActive(attemptId) |
| 16 模块测试 / 空顶栏 / 窄终端 | 父级先前 551 项只覆盖其已列模块；T: `empty tabs retain Sessions and plus pointer and keyboard actions`、`empty top bar clips to $width columns and can page to both controls` | 本任务两个 suite 的 task/filter 见下；E renderer、Native、真实 PTY 均待父级执行 |

### 既有漂移与本轮变更的区别

- `git show HEAD:integration-test/.../ApplicationRootBoundaryTest.kt` 的原第 288 行已经使用四参数 `RpcGlobalSettings.open(services.global, openCliFrontendSettings(root), local, 120)`；实际 API 是 `open(rpc, scope, applicationWidth)`，frontend settings 已经通过 RPC 读取。此处是本任务接手时既存装配漂移，改成当前三参数，没有更改 Settings 期待。
- 原第 290/298 行 factory models 参数、带名称/完整配置的 arguments lambda，以及 `.selected`/默认草稿 tab，是本轮删除接口所需迁移；改为借用 global、单个 cwd arguments、联合 navigation。
- 原 fixture 没有 Settings/Login/directory 的安全真实工厂，只有 `error(...)`；本轮在 integration 范围内补真实组件装配。未去生产添加测试开关，未动 shared fixture。
- 原 `RpcSessionViews(reportUnhandledError = { throw it })` 不适合 orphan failure 交接。本轮注入一个既有语义的 `report` 捕获出口，同时给 views 与 Application owner CoroutineExceptionHandler；N8 的 live 原异常只由 renderer callback 报一次，N9 的 orphan 只由 owner handler 报一次，没有另加失败 state/consumer authority。

### 父级可执行入口（本 Session 未执行）

- 在父级既有隔离副本/资源锁与 JVM 参数门禁内，从 `Kodex/` 执行以下 **task/filter**；JVM 与 `org.gradle.java.home` 仍由父级按实际 daemon 校验后显式传入：

```bash
./gradlew :integration-test:jvmTest \
  --tests '*applicationRootBoundaryTest*' \
  --tests '*newSessionIntegrationTest*' \
  --no-parallel --max-workers=1
```

- 两个 suite 静态展开预期 **8＋23＝31 项**，不是已发现或已执行数量。单独验收新增项时只保留 `*newSessionIntegrationTest*` filter。
- 已只读核对本地缓存 TestBalloon 1.0.1-K2.4.0 Gradle/plugin sources：支持 `--tests` wildcard，并映射到 framework include patterns。父级须确认无继承的 `TESTBALLOON_INCLUDE_PATTERNS` / `TESTBALLOON_EXCLUDE_PATTERNS` 覆盖选择，并检查实际 XML 中 suite 名/数量，不能只看 task exit。
- 报告预期在 `integration-test/build/test-results/jvmTest/` 及 `integration-test/build/reports/tests/jvmTest/`；结果由父级记录。
- 不使用整仓 test task，不启用 realIO、perf、录屏或线上模型环境开关；此过滤只选择上述两个安全 fixture suite。

### 静态检查、清理设计与边界缺口

- 已读根指导/技能/Draft、父级冻结接口与 D 验收报告，以及 frontend、RPC session/settings、VM state、TUI、资源生命周期 checklist；只读识别本地 IDEA 进程，未控制窗口/Sync。
- 已运行 `git -C Kodex diff --check -- integration-test`，无 whitespace diagnostics；新文件另查无行尾空白且末尾字节 `0a`。扫描 integration 无旧 createNewSessionTab/materialize、共同 SessionViewModel、selected/selectedIndex、RpcDraftSettingsSource、旧 factory/arguments 消费。
- 逐段核对 API、真实数据来源、fault count、revision/cwd 成功边界、live/orphan 唯一上报和 cleanup；没有编译器或运行时验证结论。
- 任务文档已自行完成删减、变更对照与全文一致性审查；遵守不派 worker 的限制。
- `gradle.properties` SHA-256 仍为 `a73fd71eaa67ae5c40edf81a8738bdaadbab7b8549566772638101f980228f8c`，未编辑该文件或 E/生产/shared fixture/checklist/父兄任务。
- 每次 fixture 在系统临时目录创建唯一 `kodex-root-boundary-*` Home，先 prepare/持有 Home lease，最后关闭 lease 并递归删除。root Mosaic 在 finally cancel；frontend teardown 尝试 app close/shutdown、registry shutdown、views/global close、owner cancelAndJoin、global join，随后由 scoped RPC/backend 释放端点及后端资源。
- fixture 的每个 frontend cleanup action 使用限时 10 秒的 NonCancellable 收尾；即使较早 cleanup 失败仍尝试后续动作，并把异常附到原始错误。NonCancellable 仅用于收尾，不用于保活提交。gate 外层 finally 释放，阻塞的 external mock turn 由测试 finish 或 backend scoped teardown 终结。
- 当前没有创建临时实验文件、运行测试 Home、持有构建/远端/GUI 资源；没有 commit/push、worker、Gradle/Native/IDE Sync/录屏/真实账号或模型操作。
- **未证实产品缺陷**；静态审查不是产品无缺陷的证明。编译、31 项实际发现/断言、Mosaic focus/ANSI 帧时序、跨平台行为均待父级验证；若失败应先保留证据，不改期待来绿测试。
- **真实 PTY 仍待**：本任务提供的是进程内 TestMosaic＋实际业务图，该 fixture 不提供 standalone mock CLI/PTY 启动器，本任务未建立独立安全 PTY 入口。父级须另核实离线/mock 装配后验收物理光标、真实输入协议、顶栏和窄终端；不能用这里的快照/ANSI 断言代替。
- E 的组件/根 renderer tests、Native 与整仓编译均不在本 Session 执行范围；旧禁用 realIO/perf 测试保持原样。

## 本波补充交接

- 父级已实际执行两个安全 suite，共 31 项、0 失败、0 跳过（`g2-final`）；修正三个 renderer gate 的帧等待，断言未删除。另完成 Native 编译与独立 PTY，最新证据见父任务 §6.9；上文“未执行”均指 worker 交付时。
- 父任务 §6.8 已记录 D 审查及六模块 134 项通过；加基础四模块共 551 项，但不包括本任务，不能据此宣称端到端验证通过。
- Application factory 已用其 owner CoroutineExceptionHandler 接入现有错误出口：live waiter 收原始异常，取消 waiter 后由 Application-owned observer 报错一次。手工装配 Application 的 fixture 必须为 orphan failure 提供准确 handler，不另建通知真源。
- E 同时只改根 View/两个 renderer 组件；冻结 `NewSessionScreen/NewSessionContent.onSubmit: (Long) -> Unit`、`SessionTabBarCallbacks.selectNewSession()`，在本任务按该签名写测试，不越界改 E 文件。
- Login 的 isActive(attemptId) 不是组件布尔生命周期属性；优先通过实际 effects Flow 关闭与 popup exact handle 验证无 OAuth 的关闭/保留。

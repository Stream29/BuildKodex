# Task Tree

- `Inventory production behavior and prior decisions`()
- `Replace directory-based scope with behavior ownership`()
- `Review Hook redesign and confirm remaining ownership boundaries`()
  - `Review notification Hook design`()
  - `Confirm Settings authentication boundary`()
  - `Confirm backend automatic naming ownership`()
  - `Identify current settings and obsolete compatibility paths`()
- `Specify hard-cutover project mapping and deletion list`()
- `Define real-host and original-behavior acceptance matrix`()
- `Receive explicit implementation start`()
- `coroutineScope {`
  - `async { Cut over settings values and persistence() }`
  - `async { Retire control Hooks and assemble notification Hooks() }`
  - `async { Cut over backend authentication and naming() }`
- `}`
- `Integrate host consumers and dependency graph`()
- `Run original behavior, real RPC, and platform validation`()
- `Deliver scoped acceptance evidence`()
- `Receive user acceptance`()

# Details

## Authorization and status

- 用户明确验收通过并要求提交；本任务与三份 scoped 子任务归档 done。
  内层代码本地提交 `206266c8`，本轮未获推送授权。
- 用户完成工程方案审阅后明确要求“好，那就启动执行”，授权本任务及三条工作线的
  硬迁移。保持原 RPC 产品语义，不包含提交、推送、发布或真实 Home/凭据操作授权。
- 当前内层分支 refactor/spec，基线 `e89c5094b3ccd4f69ce4dbab8e8f30d1d8cb5df5`；
  已接受修复分四批本地提交，尚未推送。
- 原四领域、六个 Gradle 项目仅是待清点的代码范围，不是需要保留的领域结构。
  之前的 settings models/store、RPC DTO 位置也须按实际所有权复核。
- [总迁移任务](../executable/2026-09-28-plan-domain-spec-impl-boundaries.md)；
  [重验及未闭合风险](../executable/2026-10-05-reaccept-spec-impl-refactor.md#not-closed-by-this-repair)。

## Confirmed direction

- spec/impl 组织服从真实业务边界；MVVM 是交互结构，不把所有后台行为包装成 ViewModel。
- notification 当前是通知 Hook 的执行链，不规划独立通知中心、NotificationViewModel
  或新的共享管理层。
- 认证是 Settings 的完整功能切片。Settings/Authentication/Login ViewModel 负责交互，
  后端继续拥有凭据、续期和提交。不能让 Popup 关闭终止后台维护。
- 自动命名由后端拥有，前端只编辑配置、观察最终名称，不参与生成或写回。
  创建时确定 owner；实际生成时点须保留当前首条有效文本触发，除非另行批准变化。
- 采用一次性硬迁移：原真实类型直接接管，更新全部消费者，删除替代路径。
  不新增平行公开模型、转发项目或只为编译的 adapter。
- 功能相同不意味着运行生命周期相同：Settings 短期编辑、应用级已接受写入、
  前端全局 Hook 消费和后端 Agent 异步任务分别说明所有者。

## Work units and production evidence

### Hook — feature configuration and execution

- 独立子计划：[重新审视通知 Hook](2026-10-06-reconsider-notification-hook-design.md)。
- 当前事件由 BackendNotifications 生产，经 GlobalRpc 流交付；Application 建立一个
  前端消费者，Hook Settings 编辑本地配置。
- 不把旧根 hook/ 的控制协议和新的 notification-only 语义合并。
- 产品语义与旧代码退役已确定；项目与删除闭包见子计划的 Engineering cutover。

### Settings authentication — complete vertical capability

- 前端已有 Authentication Settings 与 OpenAI Login 组件；核对其状态、命令、
  登录 effect、Settings owner 与后端 OAuth/凭据实现的完整链。
- 真实后端装配使用 BackendFileSystemAuthStore；旧 FileSystemKodexAuthStore、
  KodexAuthStore 和 in-memory 实现经用途核对后退役，有效辅助逻辑和测试移入真实后端。
- **语义纠正**：现行 RPC 设计中 Codex 与 Kodex 两种来源都支持后端加载、
  续期回存、login/remove。旧 Codex 只读模型不是当前行为；
  本轮不因整理目录恢复该限制。
- 凭据与 OAuth 私有状态留后端；前端只取得摘要及当前交互所需临时数据。
  后端正常维护不依赖 Popup 存在或前端定时 reload。
- 既有跨层接口足够时直接复用，不再增加 AuthManager、provider 框架或第二份认证状态。
- 执行方案：[认证与后端命名接管](2026-10-06-cutover-settings-auth-and-backend-naming.md)。
  归属一条功能线不代表全部代码进入前端组件。
- 依据：[认证与 OAuth](../../checklist/rpc-authentication.md)、
  [BackendFileSystemAuthStore](../../Kodex/rpc/impl/server/src/commonMain/kotlin/io/github/stream29/kodex/cli/auth/BackendFileSystemAuthStore.kt)、
  [BackendServices](../../Kodex/rpc/impl/server/src/commonMain/kotlin/io/github/stream29/kodex/rpc/server/BackendServices.kt)。

### Backend automatic naming

- 当前 SessionTitleGenerator 是实际生成能力，OpenAiSessionTitleGenerator 是其实现；
  AgentTitleGeneration 负责一次性触发、显式改名/历史替换与结果写入的竞争。
  两种职责不能被一个“创建后 launch”替代而丢失行为。
- 生产 appendUserMessage 成功后调用 title.start，实际任务归 runtime scope，
  不是创建 RPC 的调用 scope，也不是前端页面 scope。
- 后端新建时保留初始名称；获得首条有效用户文本才生成。辅助任务失败不得使已成功
  写入的用户消息看起来失败；取消/重命名/历史替换必须阻止失效结果覆盖新值。
- 名称实际属于哪个 Agent/thread、如何反映到 Session 目录与标签，按现有模型核对；
  不新增 session-title 与 agent-title 平行状态或为讨论提前重命名公共 API。
- 前端订阅现有设置/名称投影即可；是否存在订阅缺口用真实 RPC 测试证明，
  不预建 title-generation-state 及专门通知。
- 执行方案与认证共用一个后端工作单元，避免同时修改 BackendServices/BackendAgentRuntimeRpc。
- 依据：[AgentTitleGeneration](../../Kodex/rpc/impl/server/src/commonMain/kotlin/io/github/stream29/kodex/cli/sessiontitle/AgentTitleGeneration.kt)、
  [后端消息与命名触发](../../Kodex/rpc/impl/server/src/commonMain/kotlin/io/github/stream29/kodex/rpc/server/BackendAgentRuntimeRpc.kt)。

### Settings — actual ownership and persistence

- 生产后端使用 BackendSettingsStore，前端使用 CliFrontendSettingsStore；
  Settings 页面展示两侧能力不是可写的统一配置真源。
- KodexGlobalSettings/KodexSettingsStore 旧合并模型及 in-memory 实现已退役；
  有效断言转到真实两侧 store 与纯值，不为测试保留平行模型。
- 配置值归其功能，不强制全部进入一个 settings/models 大项目；文件拆分、
  原子写入和后端 CAS 不必随逻辑领域拆分而再次改变。
- 现有 RPC 与文件序列化形状保留。若调整类型所有权，移动原声明，
  保持 FQCN/descriptor 或另行审阅必要变化，不新增转换 DTO 维持双轨。
- Shell 真实契约已修复；平台默认选择留在实际装配/实现。不得让纯值模型执行
  主机发现，也不得通过另造 Shell resolver service 掩盖依赖。
- 冻结 Home migration 的旧格式声明独立于当前模型，不因删旧运行路径修改已发布迁移。
- 依据：[设置与文件归属](../../checklist/rpc-settings.md#文件与字段归属)、
  [拆分迁移任务](../done/2026-09-27-rpc-settings-split-migration.md)。

## Planning gates

- 对每个保留的类型记录真实 owner、生产创建点、消费者、状态/资源生命周期和必要行为。
- 对每个删除项记录反向依赖闭包，区分原型、旧运行路径、冻结 migration 与有效测试。
- Hook 产品语义沿用原 RPC notification-only 决策，不重新选择；本轮确认删除旧控制
  代码后，细化 Runtime/Session 旧 Hook 签名的消融范围。
- 采用下述有实际消费者的项目映射，不为符合配对规则生成空 spec。
- 实施切片能完成真实宿主接管，不以新旧并存或只跑孤立组件测试验收。

## Acceptance coverage

- Hook：真实事件生产/RPC/前端执行链、Settings 同帧编辑、固定配置快照、
  超时/取消/进程退出、无订阅/重订阅、原 UI 对比。
- 认证：真实 Settings→Login→return、exact target/attempt、迟到结果、
  两来源持久化与续期失败、关闭页面不停止后台、原摘要 wire。
- 命名：首条有效文本、关闭前端仍完成、显式改名/历史替换竞争、后端 owner 结束、
  错误不改变消息成功事实、名称订阅与目录/标签更新。
- 配置：真实前后端入口、CAS/队列关闭语义、缺省 Shell 选择、文件权限及冻结迁移不变。
- 构建平台与实际运行分别记录；不拿 JVM 通过替代其他平台资源生命周期验证。

## Engineering map — one authority per real type

| 原代码 | 唯一目标 | 处理方式 |
| --- | --- | --- |
| NotificationHook、NotificationHookType | `hook/spec/notification` | 移动原模型，保留 FQCN、serializer 与验证；不复制 DTO |
| collectNotificationHooks 与真实进程执行 | `hook/impl/notification` | 原函数唯一入口，不新增 Executor/Dispatcher/Manager 接口链 |
| KodexAuthSource | `app/component/authentication-settings/spec` | 类型仍共享于 RPC/文件/组件，后端不依赖 view 或 ViewModel 实现 |
| KodexAuthLoginAttempt | `app/component/openai-login/spec` | 原真实前端 attempt 契约，RpcOAuth 与 UI 消费，删除旧 auth contract 文件但不删除该类型 |
| KodexNewSessionSettings | `app/component/new-session-defaults/spec` | 原值模型/默认值直接归位 |
| SessionTitleSettings、DefaultSessionTitleModel | `app/component/session-title-settings/spec` | 纯配置与默认值；不包含生成请求或后台 Job |
| SidebarSettings、SidebarContent、宽度常量、NewLineKey/SubmitKey | `app/component/application-preferences/spec` | 保留真实侧栏投影和键配对，不因旧 store 退役误删 |
| BackendSettings、CliFrontendSettings、CliSidebarSettings、Notification | 原 `rpc/spec/models` | 保留真实 published model/wire；更新到上述 pure specs 的直接依赖 |
| BackendSettingsStore、CliFrontendSettingsStore 的真实操作 | `app/settings/spec/persistence` | 以原名称直接接管接口，真实工厂/消费者返回该类型，无第二套 Store API |
| 两侧 store 的文件实现、SplitSettingsFiles/Persistence | `app/settings/impl/filesystem` | 保留两文件、两个 mutex/state 和共同原子写入机制，不建立合并设置真源 |
| BackendFileSystemAuthStore、token/JWT/PKCE/私有文件格式、PreparedOpenAiLogin | 原 `rpc/impl/server` | 成为现有后端实现的内部功能文件，不新增认证管理项目/状态 |
| SessionTitleGenerator、SessionTitleGenerationResult | 原 `rpc/spec/contract` | 原有真实后端注入契约，供 withBackendServices/生成实现/测试使用；不是新 RPC service |
| AgentTitleGeneration、OpenAiSessionTitleGenerator、生成 schema/helpers | 原 `rpc/impl/server` | 原算法与测试直接迁移，前端解除生成实现依赖 |
| 旧控制 Hook、TurnHook decorator 与旧合并 store/login | 删除 | 删除运行 API 与消费者，不通过 alias/adapter 维持旧模型 |

- 新增四个有真实职责的项目：
  `:hook-spec-notification`、`:hook-impl-notification`、
  `:app-settings-spec-persistence`、`:app-settings-impl-filesystem`。
  原六个 `:app-shared-*` 项目全部退役，旧 Hook 两项目和 TurnHook 两项目删除。
- 保持现有命名有意义的 FQCN；本次不同时改包名、序列化名或公共业务语义。
- 子方案：[配置与持久化](2026-10-06-cutover-settings-models-and-persistence.md)；
  [后端认证与命名](2026-10-06-cutover-settings-auth-and-backend-naming.md)；
  [Hook](2026-10-06-reconsider-notification-hook-design.md)。

## Dependency graph and cycle check

```text
pure component specs -> OpenAI models / coroutines / serialization
hook/spec/notification -> serialization
rpc/spec/models -> pure component specs + hook/spec/notification + clean models
app/settings/spec/persistence -> rpc/spec/models + coroutines
app/settings/impl/filesystem -> persistence spec + real filesystem + Kaml
hook/impl/notification -> hook spec + rpc/spec/models + Shell implementation
rpc/spec/contract -> rpc/spec/models + existing domain specs
rpc/impl/server -> rpc spec + settings filesystem + existing backend dependencies
frontend VM/application -> component specs/impls + client + settings filesystem + Hook execution
```

- Hook model spec 不引用 Notification 类型/API，否则会产生
  `rpc/models -> hook/spec -> rpc/models` 环。通知匹配/序列化运行代码留 impl。
- 四个承接纯配置值的组件 spec 不依赖 rpc/models；移除其 app-shared-settings-contract
  边，RPC 只依赖 spec，不依赖 UI/viewmodel/CLI。
- `SessionTitleGenerator` 是后端本地注入 port，不注册新服务、不发送 generator/result；
  DefaultSessionTitleModel 移到纯配置 spec，前端不再依赖 server 或生成实现。
- 共同持久化项目是文件机制，spec 暴露两种真实 store 的操作；
  不引用 Settings ViewModel、auth 后台或 Hook 执行器。
- 旧工程中已有的 app/contract 与混合 agent-state 项目暂不全树迁移，
  但不能被当作保留旧 Hook/合并设置的理由。

## Execution ordering and ownership

1. 协调者核验基线、index、共享资源及原 UI 测试输出，登记当前项目/符号清单。
2. 并行准备三个独立工作单元，所有 Gradle 构建归协调者：
   - **配置线**：pure 值归位、两 store 接口/真实文件实现、旧合并模型退役与持久化测试。
   - **Hook 线**：新通知模型/执行器、旧根控制链与 Runtime/Session 签名消融、相关测试。
   - **后端线**：server 内认证与命名接管、旧登录/loader 清理、真实 RPC 测试。
3. 共用文件只由指定 owner 修改：
   - 后端线独占 rpc/impl/server（含 BackendServices 删除 NoOp 参数）。
   - 配置线独占四个 pure component spec 中的值声明、app/settings、新旧设置文件。
   - Hook 线独占 hook/、Runtime Hook 分支、Agent Session dependency 签名。
   - 协调者独占外层文档、所有现有宿主 Gradle/RPC models Gradle、settings registry、
     Application 入口接线、RpcSettings/RpcConfigurationSettingsDependencies、
     integration-test probe 和主 UI 原测试。
   - 每条线只修改自己的新项目 build 文件；共享构建坐标由协调者集中切换。
4. 集中接线、删除六个旧 shared 项目与四个旧控制 Hook 项目和所有旧坐标，
   在唯一生产图上跑以下矩阵。
   中途不把新旧并存判为完成，不提交不可构建的中间切片。
5. 必要修正后同步 SOP 与迁移记录，交用户验收；提交/推送另需授权。

## Central validation matrix

| 单元 | 必跑的 JVM 目标 | 证据 |
| --- | --- | --- |
| 配置值与持久化 | 四个 pure component spec、RPC models、`app-settings-impl-filesystem` | 原 descriptor/file golden、两文件隔离、CAS、权限、失败主异常、store 返回真实 spec |
| Hook 功能 | `hook-impl-notification`、Hook Settings VM/View、RPC server | 四分支、同类 pending 聚合、配置快照、串行/超时/取消、原 UI 与真实订阅链 |
| 控制链消融 | Runtime Tool/Compact/Steer/Composition、Session 两 impl 与 RPC server | 不留 Hook 参数/NoOp/控制分支，保留正常工具调用、pending、compaction、运行关闭 |
| 认证与命名 | RPC server、OpenAI client、Authentication Settings/OpenAI Login/Title Settings VM/View | 两来源维护、exact attempt、前端关闭不停止维护/命名、首条文本与改名竞争 |
| 宿主 | app ViewModel RPC/Settings/Application/Session 与 View Settings/Application | 原端到端接线、失败与 queue 生命周期、Settings→Login→return、目录/标签命名订阅 |
| 兼容性 | migration impl、Integration JVM 测试源码、CLI JVM 主源码 | 已发布 migration 不改，legacy probe 不再引用旧 loader，主源码不依赖 app/shared |

- 执行时使用真实 Gradle 坐标 `:<path-with-dashes>:jvmTest`；无测试源码目标只记编译，
  不按成功测试计数。新 store/spec targets 首次验证实际实现与接口关系。
- Linux x64：对四个新项目、RPC server 及直接受影响的 Runtime/Session 主源码
  运行 `compileKotlinLinuxX64`。实际 `kodex.kmp-cli` convention 没有 JS target，
  不执行不存在的 `compileKotlinJs`，不把不存在的目标记为验证通过。
- JVM 隔离真实进程 fixture 必跑；Native runtime 仅在平台/资源可用时运行，未运行明确留项。
  不运行用户 Hook、真实 OAuth 或 live Responses probe，不读写真实 Home。
- 每个构建日志按目标/平台/执行或 up-to-date 分开记录，不预先承诺用例数量。
- 静态门槛：单一定义、旧坐标/旧类型清零、spec 主依赖无 impl、无无用接口、
  冻结 migration 生产 codec/算法与输入 fixture diff 为零；测试 consumer 可改用独立目标值
  断言，不保留旧合并 loader。旧 project build 目录在验证后安全清理。

## Readiness and stop conditions

- 工程方案已补齐；没有待选择的产品语义或已知外部前置阻塞，可准备执行。
- 当前已获明确开始授权并移入 executable；按下述交接与集中验证执行。
- 开始时若基线变化或用户占用构建资源，暂停受影响操作；不切分支或重置用户改动。
- 若发现真实生产依赖遗漏，先修正工程清单；若涉及 wire/文件格式、来源只读、
  新控制能力、触发时点或生命周期保证变化，停止该项并请求设计确认。
- 重验遗留实验独立跟踪；本轮静态闭包不存在阻塞不等于这些运行风险已经通过。

## Current checkpoint

- 三条工作线已交接并完成统一宿主接线。四个新项目真实承接契约/实现，
  十个旧项目及其生产 API 已删除；原类型、serializer 与实际装配保持唯一。
- Settings 配置值归对应组件 spec；两个真实 store 接口及共同文件机制归 app/settings；
  凭据维护与异步命名归原 RPC server，前端仅编辑/观察；通知配置与执行归 Hook。
- 执行接线发现原 auth contract 中的 KodexAuthLoginAttempt 已被现行 RpcOAuth/UI 使用，
  协调者直接迁入 OpenAI Login spec，不能按旧 store API 一起删除；不新增替代类型。
- migration 验收 harness 仍调用旧合并 loader；已改为独立的显式 BackendSettings/MCP 凭据
  期望值，保留全部输入 fixture、生产迁移代码及原故障/恢复断言，不保留旧模型凑编译。
- 当前生产只有一个 Application 通知 consumer；无旧控制 Hook 参数、NoOp、
  TurnHook/ToolHooks/CompactionHooks 或旧 shared 坐标，不新增 resolver/forwarder。
- `SplitSettingsFiles/Persistence`、BackendFileSystemAuthStore、SubscriptionJwt 与基线逐字节一致；
  Notification 执行体只增加 KDoc。命名移除无人使用的非 CAS 分支，唯一生产 CAS 保留。

## Central acceptance evidence

- 同一 Gradle 9.5.1 Daemon/JDK 25、`--max-workers=1` 集中构建，未操作共享 IDE。
- 最终 JVM 矩阵成功：29 个有测试目标，846 项，0 failure/error/skipped；
  包含本轮较早成功执行后最终 up-to-date 的结果，不声称全部重新执行。
  Runtime composition 无独立测试源码，仅记录编译；其真实装配由 Session/RPC 测试覆盖。
- 分组结果：四个纯值 spec 10、RPC models 31、持久化 61、通知 14、server 118、
  Runtime Tool/Compact/Steer 33、Session 两实现 126、migration 59、
  Hook/Auth/Login/Title VM/View 78、RPC/Settings/Application/Session VM 123、
  Settings/Application renderer 119、OpenAI client 74。
- 真实通知链：RPC Stop → 隔离本机命令，包括非零退出继续后续命令、实际十秒超时清理、
  旧 continue 输出不控制 Agent、混合 pending 事件的完整参数/顺序不丢。
- 真实命名链：接受首条消息后关闭借用前端 view 与 Settings，backend Job 存活；
  存活 tab settings、原 SettingsTimelineRpc 与 catalog 观察新名称。
  显式改名、历史替换、owner close 和迟到结果由原 port/RPC 用例覆盖。
- 原 Settings→Login→return、应用唯一通知订阅、原 UI 文本/键盘/宽度回归通过。
  没有改生产 renderer；不将录制测试称为人工交互式终端验收。
- CLI JVM 主源码、Integration JVM 测试源码编译通过；不运行 live probe。
- 十一个 Linux x64 主源码目标成功：四个新项目、server、Tool/Compact/Steer/Composition、
  Session 两实现及其依赖；这是 klib 编译，不是二进制 link 或 Native 进程运行。
  当前 convention 无 JS target；macOS/Windows/Linux arm64、Native runtime 和真实 OAuth 未运行。
- 首轮验证修正了 backend Shell.default 的显式 impl 边、两处测试 Int→Long、
  Settings 测试的直接 filesystem 边和一处旧 Hook draft 测试调用；
  不恢复旧 API/别名，不降低测试断言。
- 静态自有项目依赖图无悬空坐标/主依赖环/spec→impl 边；唯一类型和旧符号清理通过。
  原重验的 History 压力超时与其他未闭合实验仍独立保留，不借本批宣布全仓重验通过。
- 四份任务、总任务及本批 SOP 的 85 个当前本地链接/锚点核验通过；
  内外层 diff 与新增文件文本检查通过。十个旧生成目录和空 shared 树已安全清理。
- 临时构建日志清理；集中构建结束后仅停止本会话新建且已空闲的 Gradle/Kotlin Daemon，
  释放构建内存，不停止其他进程或操作用户 IDE。
- 用户已验收，内层代码提交 `206266c8`；外层记录与子模块指针分别提交，
  未推送。其他用户改动与共享设备未触碰，原重验未闭合风险继续独立跟踪。

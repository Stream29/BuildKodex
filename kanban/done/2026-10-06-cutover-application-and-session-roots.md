# Task Tree

- `Trace root navigation, materialization, registry, recovery, and host cleanup`()
- `Specify preserved identity and exact-target command changes`()
- `Receive review of root API, race experiments, and shutdown gates`()
- `Receive implementation authorization`()
- `Move original Session and Application contracts and real implementations`()
- `Cut over exact-target commands, renderer, CLI, and shared dependencies`()
- `Validate startup, cancellation, recovery, cleanup, and original UI`()
- `Return evidence for whole-batch acceptance`()
- `Receive scoped acceptance with the parent batch`()

# Details

## Actual owners and target projects

- [Parent plan](2026-10-06-hard-migrate-frontend-ownership-roots.md)；用户已授权执行。
- 原 `app/contract/session` → `app/spec/session`；
  Session/PersistedSession/Lifecycle/Registry/Factory 的原 FQCN 保留。
- 原 `app/contract/application` → `app/spec/application`，
  SidebarSettingsViewModel 移到独立 sidebar spec，不重新定义其值模型。
- [SessionViewModels](../../Kodex/app/impl/session/src/commonMain/kotlin/io/github/stream29/kodex/cli/session/SessionViewModels.kt)
  → `app/impl/session`；原
  [ApplicationViewModelImpl](../../Kodex/app/impl/application/src/commonMain/kotlin/io/github/stream29/kodex/cli/app/ApplicationViewModel.kt)、
  SuggestedWorkingDirectoryAdapter 与 Sidebar 实现 → `app/impl/application`。
- [withKodexApplication](../../Kodex/app/impl/application/src/commonMain/kotlin/io/github/stream29/kodex/cli/app/Application.kt)
  同样在 Application impl，保留真实 scoped host seam与启动 fault injection；
  非 frontend spec 的 factory参数不变成“通用应用服务”。
- 原 Root View → `app/impl/view`，保留
  [SessionTreeCliScreen](../../Kodex/app/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/SessionTreeCliScreen.kt)
  名称与真实根作用，不新增 RootScreenAdapter；CLI 只换真实 import/坐标。
- 无源码的 `app/view/session` 删除。Session Koin marker核对真实生成装配后退役，
  不重写全部 convention/DI 框架。
- 消融自查：保留原 root/registry/binding 的具体分工，不加 ApplicationCoordinator、
  SessionResolver 或 generic Target wrapper。Sidebar 的真实实现留在 Application impl，
  其 Shell 呈现与侧栏 View 同项目，不需要单独 Shell 工厂或 View 跨层转发。

## Session identity, observations, and errors

- 持久 Session handle 是本地 tab owner，可跨 binding replacement；
  当前 rootAgent 则绑定实际 Agent lifetime。保留 Loading/Open/Failed/Closed，
  不因本地生命周期给远端发 Stop/closeSession。
- 结合 Agent 合并，Session 从 RpcSessionView 的原唯一 Agent 发布源取得 child；
  删除原 presentation→new wrapper 的重复创建/收集路径。
  Session own close 与共享 view release 的顺序必须证明，不直接把可关闭借用对象
  变成两个 owner 都独立 close 的生命周期。
- RpcSessionView 使用单一 `agent` 发布源，Session rootAgent 直接借用；current
  命令验证 Ready 并捕获该 child，child 再按固定 binding 校验准入。
  不通过重新包装、URI 相等或 latest selectedAgent 保证目标，也不保留 presentation alias。
- RpcSessionDraft 已有实际 create/append/resume 算法保留，只将最后 resume 目标
  从 presentation 切到该准确 Agent。缺失/closed 不能伪造成功，不能借这次消融
  改成第二套 submit/receipt 或远端自动补偿。
- settings/name 投影按实际绑定与元数据进展；不能简单删所有投影：
  恢复间隙可能仍需最后可见值，必须逐字段列明它是缓存而非持久化第二真源。
- registry 与 RpcSessionViews 分别复用 tab handle、执行 RPC heartbeat/recovery。
  保留首次 Ready 之前失败不污染后续 open、一个 caller cancel 不驱逐共享初始化、
  closed 不重新激活、已接受后端 work 不因前端等待取消。
- 一个打开 index 一条 20 秒 heartbeat，后端 TTL 60 秒沿原规则；
  缺失是 Missing/SessionNotFound，不自动 create，不增加 index nonce/lease DTO。
- Session settings/fork/rename 按 captured owner；timestamp 每次实际菜单读取，
  cacheNonce 不是完整业务身份，更不是跨多次查询的事务锁。

## Approved exact-target API cutover

- 当前 UI 有准确 target，却先把它变成 index，再在另一 coroutine 中等 root Mutex；
  root 方法随后从最新导航读该 index，可能错选或抛 `Tab ... is not a New Session`。
- 提议直接替换，而非新增兼容 overload：
  - `selectTab(target: SessionViewModel): Boolean`：在同一 commandMutex 中验证
    target 实例仍在 tabs，成功更新其当前 index；已移除返回 false。
  - `materializeNewSession(target: NewSessionViewModel): PersistedSessionViewModel?`：
    取得锁后按实例定位；不存在/已替换返回 null，无第二次 materialize/append/resume。
    成功替换准确槽位，保留 list size/selected index，关闭其 popup 与 draft。
- 真正 materialize/create/open 错误仍抛出；null 仅是准确过期目标的结果，
  不用于失败、取消、后端结果未知。关闭后如何拒绝沿 root 原规范单独写明。
- 不加永久 mapping/idempotency journal。原 RpcSessionDraft 的已创建 index、
  renameAttempted/submitted 继续承担真实局部准入；成功创建后 append 失败仍保留
  Session/草稿，不自动删除或重放创建。
- 该变更已随 Planning 获用户评审与执行授权；相关独立
  [New Session crash 调查](2026-10-03-investigate-new-session-tab-crash.md)
  保持原文，不把更早 baseline 的行号当作现行源码定位。
- registry release/open 的旧 B2 风险必须 gated 复现；
  如确认，则提交现有 exact-view release/互斥范围的最小修正供评审。
  保持原两个职责，不新增第三个 registry；未证实不以“实验通过”宣告已修复。

## Root popup and lifecycle

- Application 保持 navigation/popup 的唯一 authority；child 自己持有 drafts、
  confirmation 与 commands，root 不镜像 child 状态。
- CloseTab/Archive/Delete/目录/Fork/建议批量开 tab 仍各有原业务含义，
  不把关闭 tab 变成关闭后端、把 fork变成自动选择，或把归档当 Stop。
- Login returnTo 保留准确 Settings open；旧 popup dismiss 不影响新 popup，
  删除/关闭 target 同时清理所属子组件与菜单。
- Settings、Rename、WorkingDirectory、Suggest cwd 对 target/CallId/revision 的
  捕获规则不同，保留原 spec，不能改成单一 popup resolver。
- 所有新 root API KDoc注明依赖、准确目标、状态/overlay渲染、返回、throw、
  caller cancel、owner close 和 accepted remote work区别。
- Host 启动/退出保留 Home→Backend→RPC→Frontend→Renderer 的结构化装配，
  失败清理反序、NonCancellable 收尾和原 primary/suppressed。
  Notification collector仍前端真实 Hook；backend title/auth 生命周期不改。

## Coordinator ownership and verification

- root 线可实现新 Session/Application spec及不共享的实现测试。
  协调者独占 Rpc*、Application.kt、SessionTreeCliScreen、CLI/Gradle、
  原文件删除和 recording fixtures；不能两线并改同一个 raw binding/owner。
- SessionRepositoryViewModel/FileSystemSessionViewModelOwnership/AgentHistoryAction、
  Application 十套 common suites、RpcSessionViews/Settings/OAuth 及现有 root suites
  全部保留并按实际所有者迁移；不把现有 mock storage测试改成外部真实 Home。
- Gated 测试：旧 target 等锁期间关闭前一 tab、重复 submit、同名不同实例、
  create 成功 append 失败、late callback、popup target关闭与 root shutdown。
- 恢复测试：失活旧 Agent不能再改写，新 child发布；borrowed child close
  不 Stop backend；终态首次 open失败后可重试；取消一个 opener不影响另一个。
- root release/open 与 shutdown竞态独立记录实验结果；不得把同一编号换成永久
  唯一协议来规避准入。测试不增加新的自动重放恢复路径。
- UI 原 tabs/title、菜单日期、NewSessionScreen、RevertAndEdit、
  SettingsLoginLifetime 与三套 Docs recording保留；实际回调/输入驱动。
- Root/CLI/Integration JVM 编译和真实 in-memory RPC矩阵由协调者运行；
  其他 Native、真实 CLI/live providers未验则列出，源码移动完成不等于生命周期已验收。

## Coordinator validation checkpoint

- 新 Session JVM suites 在原 release 准入算法上实际执行：13 项中 12 项通过，
  `gated registry release cannot close a fresh view registered by an opener waiting
  on its mutex` 因新 opener 被取消而失败。原 B2 风险已复现，不再仅为推测。
- 协调者随后用原 Mutex 覆盖旧 raw view 的准确释放，移除 handle 后只释放其 view；
  没有 handle 时仍在该准入锁中清理 index 的未完成初始化。最后等待 handle 完成。
  没有新增 registry、lease、重放或错误转换；改后 Session 的十三项测试全部通过。
- 集中编译发现的 Root Application 迁移逗号语法问题已由 Root 线修正；
  该修正不改变 Rename 行为。

## Coordinator final handoff and verification

- 原 Application/Session 契约直接迁入 `app/spec/{application,session}`；
  原 Application/SuggestedWorkingDirectoryAdapter/Sidebar 实现和原 scoped
  Application.kt 在 `app/impl/application`，原 Registry/PersistedSession 实现
  在 `app/impl/session`。旧四项目的 source/test/build 和无用 marker 删除，
  原有效测试保留并直接修改消费方，不保留兼容 overload 或另一套工厂。
- 唯一公开切换为 `selectTab(target: SessionViewModel): Boolean`、
  `materializeNewSession(target: NewSessionViewModel): PersistedSessionViewModel?`。
  原工厂及其参数维持真实职责；原 Registry 三参数构造、Raw RpcSessionViews
  与准确 view release 分工保留。Session.rootAgent 直接借用 view.agent，
  无 Presentation/类型 cast/第二份 mutable Agent 发布。
- Application 43、Session 13、RPC 73、Root View 52 项通过；
  新 exact-target 八个 gated 用例、真实恢复/取消/释放五个用例保留，
  原 startup/Home primary/suppressed 清理与 UI recording 断言仍在。
- 原 host 取消用例最终复跑暴露迟到订阅错误替代取消；
  经受控实验确认后，在原 Global 观察终结边界恢复取消优先，
  活跃 ordinary failure 仍传播。八次真实独立 Home 关闭循环通过；
  这是已证实的最小生命周期修正，不以反复绿色推断根因已解决。
- 两根 spec/impl、真实 RPC 与 Root View 的 Linux x64 主源码编译通过，
  CLI JVM/Integration 测试源码编译通过。完整
  [最终矩阵及限制](2026-10-06-hard-migrate-frontend-ownership-roots.md#final-validation-and-remaining-limits)
  由中央统一归档证据；用户已随整批验收，按
  [验收范围](2026-10-06-hard-migrate-frontend-ownership-roots.md#scoped-acceptance) 归档。
- 追加真实 Linux x64 CLI [局部验收](2026-10-06-hard-migrate-frontend-ownership-roots.md#actual-cli-behavior-acceptance)：
  实际新建/切换/关闭草稿、无账号物化、持久 tab 关闭后重开、重启后恢复历史，
  两次 Ctrl+C 退出码为 0，Home lease 清除。

# Task Tree

- `Pin the current committed source and complete project inventory`()
- `Start independent read-only reviewers`()
- `coroutineScope {`
  - `async { Review platform and process utilities() }`
  - `async { Review other utilities() }`
  - `async { Review tools and MCP() }`
  - `async { Review Runtime and OpenAI() }`
  - `async { Review backend ownership and persistence() }`
  - `async { Review Settings components and persistence() }`
  - `async { Review frontend roots and Agent components() }`
  - `async { Review UI foundations and Home bootstrap() }`
- `}`
- `Independently verify project coverage and important findings`()
- `Publish a qualified repository verdict and remaining work`()

# Details

## Authorization and baseline

- 用户要求赶紧全仓审查，允许多个 subagent 并行。本轮只读审查源码，各线仅写自己报告；
  不修复、构建、操作 IDE/设备、运行真实服务、提交或推送。
- 固定 `Kodex/refactor/spec`：`6b7129fa0e719dde8f88a0034b9d572679984a03`，
  内层工作区干净。HEAD 改变时立即报告，不把不同基线结果混合。
- [收尾任务](2026-10-07-plan-spec-impl-closure-and-consistency.md)
  的静态图判断只是本轮待审输入；未授权的 Home/State 修复继续停在 Planning。
- 原审查、修复和 done 测试记录作为历史证据，不直接继承其“通过”结论。
  不推定全仓已完成，也不为满足形式而判纯函数/DTO/spec-only 项目有错。

## Complete project allocation

| 审查线 | 当前项目数 | 实际范围 |
| --- | ---: | --- |
| [平台/进程](2026-10-07-final-audit-platform-utils.md) | 9 | utils 的 shell-client、process-client、os-environment、kodex-home、images-codec |
| [其他 utils](2026-10-07-final-audit-other-utils.md) | 23 | 其余 utils，包括实际 lease |
| [Tool/MCP](2026-10-07-final-audit-tool-mcp.md) | 29 | 两根完整项目与生产注册 |
| [Runtime/OpenAI](2026-10-07-final-audit-runtime-openai.md) | 20 | 两根完整项目 |
| [后端](2026-10-07-final-audit-backend-roots.md) | 33 | agent-state/context/session/storage、rpc、hook，含两个 fixture 项目 |
| [Settings](2026-10-07-final-audit-settings-components.md) | 38 | 12 个 Settings/账号组件与两侧 persistence |
| [前端](2026-10-07-final-audit-frontend-ownership.md) | 50 | 其余组件、Application/Session/RPC/root View、CLI、RPC fixtures |
| [UI/Home](2026-10-07-final-audit-ui-home.md) | 3 | components 与 migration；另专审 History View 内迁入的 cli.patch |
| 协调者 | 1 | integration-test；全仓 Gradle/规范/遗漏核对 |
| **合计** | **206** | 实际 build 文件，不含虚拟根项目和外部 included builds |

- 每线从当前 build 文件清单逐项记录审查覆盖，不能以整个根目录一句“通过”代替。
- History View 项目计入前端线；其 `cli.patch` 专属源码由 UI/Home 线复核，
  不作为新增项目重复计数。跨线可以读同一消费者，根因由协调者去重。
- 根 Gradle、buildSrc/conventional plugins、CLI 配置、资源/serializer/生成源码与
  test source-set 图也要检查。外部 Mosaic/LuceneKmp/KotlinMcpSdk 只核对使用
  与版本/装配边界；不宣称已重审其所有内部代码，不改其他子模块。

## Common evidence standard

- 逐域追踪原声明/迁移历史 → 当前实际 spec → 真实实现/工厂 → 生产装配/调用
  → 真实测试。记录准确 FQCN、文件 URI/行号及可复核的链路。
- 区分纯模型、serializer/协议、纯函数、组件 MVVM、平台 actual、基础设施与
  test-support；接口 `implements` 不是所有领域的必要条件，文件夹名称也不是证据。
- 排查未接入 spec、仍在 impl 的规范声明、旧并行 owner/状态/API、只有转发的包装、
  重复算法/DTO、spec 主依赖绕路、UI framework 泄漏及仅用于过编译的层。
- 正当 storage/RPC binding、真实资源/child 生命周期、derived state 与 widget
  buffer 不因名字包含 Adapter/Handle/State 就认定无效。消融建议必须说明删除后
  如何保留职责；不建议为了“配对”新增空接口。
- 对照当前规范检查 KDoc/@throws、close/release/cancel、owner/borrow、准确 target、
  generation/revision、持久化/wire 与原 UI 兼容。契约缺文档与确认行为错误分别分级。
- 测试必须实际验证相应链路；静态阅读、历史测试绿色、孤立 fake VM 和真实 renderer
  输入分开记录。不得把直接模型调用包装成 UI 验收。
- 基线 Home lease 取消/清理和 State compact 覆盖已复现但未修复；应记录真实影响，
  不当作“新增迁移回归”，也不因测试绿色当已解决。其他旧风险重新核验当前位置。
- 并行线不构建、不写源码/其他任务；发现需要运行的疑点写最小复现实验。
  协调者后续集中决定验证，不让八线争抢共享资源。

## Report and verdict rules

- 每报告第一行给出 `IN PROGRESS` 或 `REVIEW READY`，覆盖表列每个项目
  `R / D / B1 / B2 / U`，不得默认没有记录的项目已通过。
- `B1`：有实际链路证据的未接管/重复权威、契约或行为错误。
  `B2`：具体疑点及尚缺的证据/最小实验。
  `D`：已明确的合理例外或边界债务，注明来源和剩余门槛。
  `R`：已审关键路径，无上述发现；不是全平台运行认证。
  `U`：未完成/无法判断，说明剩余范围。
- 每发现写证据、影响、原有/引入历史、测试局限及最小消融/修复建议。
  不机械累加不同审查者的发现数；同一根因跨线去重。
- 协调者逐个验证重要发现，最后分别回答：物理布局是否收口、契约是否真实接管、
  是否存在行为阻塞、运行/平台验收还有哪些缺口。“全仓通过”必须附清晰范围。

## Review completion

- 八线清单已建立，205 个项目分配给审查者，Integration 与全仓闭包归协调者。
- 用户已确认八线 Session `518–525`，按表顺序各自只写所属报告。
- 八份报告均已 `REVIEW READY`；协调者重新解析实际 coverage 表，确认 205 个
  分配项目加 Integration 恰好覆盖 206 个入口，无遗漏或重复计数。
- 审查及结果交付完成，九份审查任务归档 done；**归档的是审查工作，不是认定
  全仓架构/行为通过**。迁移总任务保持打开，修复提案仍在 Planning。

## Coordinator independent evidence — initial pass

- 重新读取全部 206 个 build 文件并区分 main/test source-set：
  639 条直接主项目依赖、222 条直接测试项目依赖；没有缺失，主图无环。
  八线与协调者的实际清单互斥且完整，数量与上表逐项吻合。
- spec 的显式主项目依赖没有 impl/旧混合项目边；**这只证明直接项目声明**。
  外部模块依赖、全局 substitution、generated code 和实际装配仍需逐项验收。
- 7 个 fixture/test-support 项目没有被非 fixture 生产 main 直接依赖；
  测试 source-set 指向它们的声明为 45 条。fixture 内部主依赖构成测试支撑，
  不与正常应用的生产依赖混计。
- `utils-host-test-support/{spec,impl}` 无源码：当前实际行为分别为导出 Ktor mock
  测试依赖、保留原 fixture 坐标。源码为空不自动成为伪契约；是否仍需兼容坐标
  由其他 utils 线结合当前消费者判定，不能仅凭目录生成新接口。
- convention 使用量：host 92、cli 44、view 29、viewmodel 27、shared 13、
  cli-executable 1。原编译/target/source-set 由现有约定配置；
  CLI entryPoint 为原 `io.github.stream29.kodex.cli.app.main`，
  Home 当前版本仍由唯一 `GenerateKodexVersion` 生成，没有第二 provider。

### Global substitution — explicit graph caveat

- [根构建](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/build.gradle.kts#L3)
  在所有子项目 configurations 把外部 `kotlinx-rpc-utils` 替换成实际
  `rpc-impl-krpc-utils-patch`。这是已存在的 Native collections 修补，不是新迁移。
- [RPC spec](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/spec/contract/build.gradle.kts#L19)
  使用外部 rpc-core；本机 0.10.3 Native variant metadata 确实声明 rpc-utils
  依赖（macOS/Windows/Linux arm64 等缓存可读）。因此直接 `project(...)` 扫描
  没有 spec→impl 边，**不能**据此声称 resolved/transitive graph 没有实现支撑。
- 当前将其作为明确的全局构建例外/待复核边界，而非凭空判定生产错误；
  后端线审 patch 真实接口与调用方。若最终需要实际 resolved configuration
  证据，协调者集中验证；本轮尚未运行 Gradle 或下载依赖。

### Integration and external boundaries

- `integration-test` 有 14 个 Kotlin 源文件，依赖都位于 test source-set，
  没有自己的生产业务模型。真实 Agent/Session/MCP、OpenAI/probe 与 Patch
  fixture 直接消费迁移后的项目；JVM Responses probe 已使用实际
  BackendFileSystemAuthStore，不再使用旧只读认证服务。
- 这些源码包含会访问账号/网络的 suites；本轮不运行。Model Catalog suite
  明确默认 disabled，Patch probe 有显式环境开关；短路或被禁用的绿色不能当
  实际链路通过。上一批只证明启用的 Patch 单次 probe，非整套 live Integration。
- 外部 included-build pins：Mosaic `7b1a4129`、LuceneKmp `549f8afc`、
  KotlinMcpSdk `53f71766`。本轮只审本仓实际使用边界；外部内部全树、live
  provider 与所有平台运行结果均未作为已覆盖范围。
- 源码基线仍为 `6b7129fa` 且干净；当前证据是静态初审，不是最终全仓 verdict。

## Coordinator final verdict — 2026-10-07

| 问题 | 本轮结论 | 限定范围 |
| --- | --- | --- |
| 目录级硬迁移是否收口 | **基本收口** | 没有另一批未迁移的 Agent/Session/OpenAI 业务根；剩余非配对项目是实际入口、测试或 UI 基础设施 |
| 新 spec 是否真实接管 | **主体已接管，严格收尾未通过** | Shell/Process/MCP 的原伪接口已退役，生产工厂与消费者使用真实类型；仍有闲置兼容 API、转发投影和规范文档债务 |
| 全仓行为终验是否通过 | **不通过** | 有确定的契约/实现矛盾与既有行为缺陷；多个具体竞态仍需运行实验 |
| 全平台/live 是否完成验证 | **没有** | 本轮只读，没有新增构建或测试；外部 included builds、真实账号/服务器和全部 actual 平台没有全量运行认证 |

- 八线逐项静态覆盖成立；`R` 只是相应关键路径未发现所列问题，
  不意味着每条分支或每个平台都已执行。不能用历史测试总数抵消当前发现。
- 协调者复核了契约、实际调用方、捕获/清理路径与引入历史；Home/lease
  跨线影响只归同一根因。没有机械相加各线 B1/B2 项目数作为“故障总数”。
- 已发现的问题大多早于对应迁移，最新目录移动保留了它们；这不要求全面回滚
  已正确接管的类型，也不允许用“原有问题”把当前全仓验收写成通过。
- History 导航饱和疑点涉及 `ec9bf1c4` 的新 intent 逻辑，但尚未运行确定性
  饱和实验；不是已复现回归，更不是已证明的历史压力 timeout 根因。

### Gate A — minimal structural ablation and normative contracts

- [Session Settings B1-C](2026-10-07-final-audit-settings-components.md#b1-c--current-strict-compatibilityobsolete-buffer-gate-is-not-closed)：
  `.viewModel` 兼容 getter、Rename effect/channel 在真实 renderer 中没有职责，
  仅旧测试消费。直接删除闲置路径、改用真实 child/typed dependencies；
  保留原 rename/cwd 生命周期和 revision/FIFO 规则。
  便利工厂重载调用的是同一个实现，不能把它误说成第二套 ViewModel。
- [Home path D1](2026-10-07-final-audit-platform-utils.md#d1--homes-single-path-projection-is-boundary-debt-not-a-second-resource-owner)、
  [utility ablation](2026-10-07-final-audit-other-utils.md#t4--host-test-support)：
  原默认 Path 仍经未被生产消费者使用的 Provider 投影，SafeRw 仅测试使用，
  fixture impl 只转发测试依赖。逐项确认维护承诺和消费者后消融，不造替代 DI。
- Codec 的已知异常、Shell 生命周期、Patch 文件操作和已退役 Hook 文案等需要
  [规范校正](2026-10-07-final-audit-platform-utils.md#p3--b1-documentation-gate-codec-io-spec-omits-known-observable-exceptions)。
  缺 `@throws` 是文档门槛，不等同于已观察到图片处理故障。
- 直接项目图结论保留；kRPC 全局 substitution 是另列的已存在构建例外，
  没有 resolved configuration 运行证据，不把它伪装成完全无实现支撑的 spec 图。

### Gate B — established behavior and ownership discrepancies

| 根因/证据 | 已确认的范围 | 最小修正与验收门槛 |
| --- | --- | --- |
| [Home/lease L1/L2](2026-10-07-final-audit-other-utils.md#l1--b1-confirmed-preexisting-publication-orphan-shared-home-root) | 取消交接可留下 owner；等待 Job 完成不等于观察到清理成功，既有 characterization 已记录错误 | 原资源交接与等待结果直接修正，主异常/清理异常故障矩阵；冻结 migration 不动 |
| [State B1-1](2026-10-07-final-audit-backend-roots.md#b1-1--confirmed-unfixed-baseline-compaction-overwrites-accepted-settings) | 压缩可覆盖期间成功 CAS 的 settings；绿色用例目前明确断言旧值返回 | 原 State commit 保留最新非压缩字段；把 characterization 改成正确行为回归 |
| [Runtime cancellation](2026-10-07-final-audit-runtime-openai.md#b1-cancel--resume-cleanup-can-erase-the-original-cancellation)、[backend auth cleanup](2026-10-07-final-audit-backend-roots.md#b1-3--auth-temporary-cleanup-can-replace-primary-errorcancellation) | pending/临时文件清理失败可掩盖原取消或 I/O 异常 | 原 owner 内保留 primary、附加 suppressed；分别注入原失败和清理失败，不新增通用清理层 |
| [Frontend F1](2026-10-07-final-audit-frontend-ownership.md#f1--b1-ordinary-historycatalog-failures-swallowed-under-a-nonexistent-reporting-boundary) | History/catalog 的若干直接命令吞错，所谓上层报告并不存在 | 真实输入驱动拒绝/失败，接入现有准确 owner 的一个错误出口；保留取消传播，不新增 failure VM |
| [Frontend F2](2026-10-07-final-audit-frontend-ownership.md#f2--b1-draft-composer-is-outside-the-supplied-new-sessionfixture-owner) | RpcSessionDraft 的 Composer scope 无父 Job；只取消工厂 owner 不会关闭它 | 使用原 owner 的结构化 scope，测试仅取消父 owner 与 fixture close；正常 CLI 显式关闭已存在，不声称已复现普通退出泄漏 |
| [Shell P1](2026-10-07-final-audit-platform-utils.md#p1--b1-shell-exit-status-and-nonwaiting-close-guarantees-exceed-implementation) | exitCode 失败条件比文档宽；UNDISPATCHED close 进入可能阻塞的 raw close | 明确真实契约并验证关闭时序/失败；不凭 dispatcher 名称推断 caller 不等待 |
| [Node P2](2026-10-07-final-audit-platform-utils.md#p2--b1-node-direct-process-cleanup-escapes-its-owner-and-close-leaves-raw-outputs-usable)、[curl C1](2026-10-07-final-audit-other-utils.md#c1--b1-preexisting-native-curl-cleanup-lies-outside-resource-owner) | Node cleanup 与 Linux curl GlobalScope 清理在原 owner 之外；Node raw output 未随 close 全部失效 | 对真实平台做 parent cancel/join、进程和 I/O 清理实验；构造关系已证实，实际泄漏规模/阻塞未测 |
| [SearchIndex S1](2026-10-07-final-audit-other-utils.md#s1--b1-preexisting-fixed-snapshot-contract-retains-caller-owned-list) | fixed-snapshot API 留存可变调用方 List，JS/Lucene 与契约不符 | 原 factory 捕获快照并测输入后续修改；当前 ToolSearch 创建新 List，不声称其普通用户路径已失败 |

### Gate C — policy/capability discrepancies require explicit reconciliation

- [Remote compaction](2026-10-07-final-audit-runtime-openai.md#b1-retry--existing-remote-compaction-default-violates-two-retry-sop)：
  实际默认 20 与 SOP 最多 2 不符；20 来自迁移前明确代码调整，
  需要确认哪一方为准，不能自动改数值。普通 Agent 采样重试不与其混算。
- [Initial backend startup](2026-10-07-final-audit-backend-roots.md#b1-2--backend-startup-eagerly-createsscans-session-root)：
  eager repository 会创建/扫描 sessions，与 Home 初始草稿按需规范不符。
  这是启动契约门槛，不是已证明的数据丢失；只能调整原 repository 的获取时机。
- [Tool capability gaps](2026-10-07-final-audit-tool-mcp.md#b1-i1--advertised-history-image-edit-always-fails)：
  宣称的历史图片编辑始终失败，Request User Input 的 `autoResolutionMs`
  没有生产 timer 消费者。需确认真实能力/超时答案语义，不静默删冻结 wire 字段。
- [MCP routing](2026-10-07-final-audit-tool-mcp.md#b1-i3--legal-raw-mcp-names-collapse-to-duplicate-executable-routes)：
  合法 raw names `a-b/a_b`、`x-y/x_y` 可归一化为重复路由并被 dispatcher 拒绝；
  先定 existing projection/validation 的消歧规则，离线真实路由 fixture 验证，
  不引入第二 registry。

### Gate D — experiments, not confirmed regressions

- 优先复现 [History F3](2026-10-07-final-audit-frontend-ownership.md#f3--b2-latest-navigation-intent-can-be-dropped-when-history-command-buffer-is-full)：
  gate 原 source read 后超过 buffer 容量发送导航，检查最新 intent 与 stale ack。
- [Settings B2-D/B2-Q](2026-10-07-final-audit-settings-components.md#b2-d--draft-source-can-use-an-observed-revision-with-a-full-stale-configuration)：
  实际 draft/source observer 延迟时的字段覆盖、单次 operation cancellation 后
  尚存活 owner 的队列 admission；不要用 fake CAS 或父 scope 取消代替相应实验。
- 其他报告的 equal Session twins、account A→B→A、MCP reconnect/refresh、
  disk publication 后 cleanup failure/cache、settings append 后 timestamp failure、
  Native EOF、进程获取交接和日志 fixture 隔离分别保留为待验证路径。
  现有 pending-tool late-success、suggestion 非事务以及窄屏策略作为明确债务，
  没有在本轮重写其产品语义。

### Handoff and scope

- 后续从“机械目录迁移”改为上述 gate 的定向闭环；原 Home/State Planning
  不足以覆盖本次全部发现，主计划已补入其他修复/实验/政策门槛。
- 未授权开始修复，未运行新测试、构建或真实 CLI；八份原报告作为固定证据保存，
  其分级与协调者的功能/文档/政策分类不机械等同。
- 最终仍固定 `6b7129fa`，内层干净；本轮只更新审查记录和 Planning，
  不提交/推送，也不关闭迁移总任务。
- 归档后核验 11 份相关记录的 961 个本地链接/URI 行号及锚点，通过；
  旧 executable 入站链接已退役，内外层 diff 检查通过，其他用户文件未调整。

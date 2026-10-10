# Task Tree

- `Verify official contracts and pinned upstream source`() // complete
- `Trace Kodex authentication, settings, UI, requests and diagnostics`() // complete
- `Specify the subscription-only plan and validation matrix`() // complete
- `Receive authorization for branch, worktree and reviewable spec changes`() // complete
- `Create feat/ultra-fast and an isolated temporary Kodex worktree`() // complete
- `Prepare the Ultrafast value, KDoc and spec tests for review`() // complete; static checks only
- `Receive user acceptance of the spec changes`() // complete
- `Receive authorization for implementation after spec review`() // complete
- `Implement the accepted storage, UI and transport changes`() // complete; runtime validation pending
- `Receive explicit authorization to commit and push the feature before runtime validation`() // complete
- `Commit and push only the Ultrafast feature branch`() // complete; 2399a6ea on origin/feat/ultra-fast
- `Receive authorization to include the ProMax plan value in this branch`() // complete
- `Recognize ProMax without changing authentication or entitlement behavior`() // complete; static checks only
- `Commit and push the authorized ProMax compatibility supplement`() // complete; d85e0e61 on origin/feat/ultra-fast
- `Validate merged fixtures and RPC/UI behavior on Xiaoxin`() // 694 JVM/Linux tests pass
- `Receive main integration authorization`() // user explicitly requests merging feat/ultra-fast
- **`Receive runtime acceptance beyond fixtures and existing CLI resource blockers`()**
- `Integrate changes into main without disturbing Gradle work`() // local signed merge c92fdc59
- `if (mergedAndWorktreeIsCleanAndOwningSessionReleasedIt()) {`
  - `Remove the temporary worktree and release task-owned resources`()
- `} else {`
  - `Retain unmerged changes and report the integration blocker`()
- `}`

# Details

## 状态与授权

- 当前 main 已生成签名合并 `c92fdc59`，保留 Ultrafast/ProMax 两个 feature
  提交及 main 的显式测试配置。合并后的官方 KGP 验证通过694项 JVM/Linux
  测试、CLI JVM及受影响 client/RPC Native 编译；没有付费请求或 main 推送。
  窄菜单提示裁切和跨模块 nullable 测试编译错误已按实际红→绿结果修正。
  完整原生 CLI 链接/smoke、真实权限及速度均未因此通过，原资源门槛保留。
- 2026-10-11 用户明确要求把 `feat/ultra-fast` 合入当前 main。
  [集成任务](../done/2026-10-11-integrate-ultrafast-main.md)记录最新门槛；本轮可生成
  合并提交并更新外层指针，重型验证只在 Xiaoxin。下面“尚未授权合并”
  的叙述保留为当时历史边界，不再阻止这次明确授权；不因此授权付费调用、
  新版发布、推送 main 或删除其他 Session 仍可能使用的 worktree。
- 用户已选择本任务，并于 2026-10-10 明确要求推进到 Planning、细化调研方案。已从 discussion 移入 planning，稳定文件名不变。
- 用户随后指定先创建 `feat/ultra-fast`、再创建 worktree，并执行 spec 变更供其 review；spec 无问题后才落地实现。任务进入 executable 时仅执行 spec 草案，没有提前实施消费者。
- 分支/worktree 已从 `5b41be52d2f64a12ea3671bcecb9151fd2d1fa60` 创建；代码位置为 `~/ACodeSpace/local/kodex-ultrafast-736`，主 `Kodex/` 保持 main 且干净。用户最新已授权提交+推送 feature branch；不合并、构建或真实付费调用。
- 调研基线：Kodex `5b41be52`，本地 Codex `604061ce51`，额外核对上游 `rust-v0.162.0` 三个文件。详细证据和来源见 [调研记录](../../shared-context/findings/2026-10-10-ultrafast-contract-and-kodex-gaps.md)。
- 关联：[Codex 对齐](../discussion/2026-09-30-align-kodex-with-codex-rs.md)。本任务不扩大为全面模型、重试或传输对齐。
- 用户随后确认 spec 没有问题；依此前“spec 无问题后再落地”的指令，已解除本任务实现门禁。实施下方对应配置、UI、传输及 best-effort 结果提示，不扩展认证、RPC 方法、重试或迁移。最新“可以提交+推送”只推进本任务 feature branch 的发布，不视为运行验收通过或 main 合并授权。
- 前轮 Planning 文档的本地链接/模块检查通过；没有用 Gradle 查询 task 或执行测试。临时调研下载已清理，参考 Codex 工作区未修改。
- 用户在核对新套餐兼容缺口后要求放入同一分支一起实现：仅扩展 `OpenAiSubscriptionPlan.ProMax("promax")` 及对应解析/JWT/摘要编解码/展示测试。该跨线枚举扩展已获具体授权，不新增其他套餐、RPC 字段或权限判定。实现交付后，用户再次明确要求“提交推送”，已授权本轮独立补丁提交及推送，不改写已发布的 Ultrafast 提交。

## 已通过的 spec 审阅边界

- 用户已确认 `ServiceTier.Ultrafast` 跨线枚举扩展及相关 spec，随后授权 `OpenAiSubscriptionPlan.ProMax` 兼容补充；不授权其他 RPC 字段/方法或错误语义调整。main 的集成仍须通过实施验收。
- Spec 审阅范围是 `openai/spec/models`、`openai/spec/client` 及直接相关的 New session defaults / Runtime configuration spec；包括值声明、相关 KDoc 与 spec 内的编解码/能力选择测试。
- Spec-only 阶段未修改 impl。用户确认后实施本任务的 UI、YAML decoder、内置目录、HTTP header 和本地 Warning；Gradle、gitlinks、Home migrations 及其他项目 IDE 仍不动。
- 本机 IDEA 进程启动于 CtripContext，IDE MCP 当前未提供可用工具；本轮不打开项目、不启动 Sync，原项目窗口保持不动。
- Spec-only 快照留下的四处 UI exhaustive `when` 和 runtime impl 全枚举断言，已在获准实施中适配；编译及运行通过与否仍以真实验证为准。
- Spec 已通过不表示实现已满足；运行验收与平台缺口仍须分别记录。

## Spec review 交付

- 审阅快照：spec-only 草案已交付且用户已确认没有问题；以下范围/静态检查是该快照的记录，不是后续实现的测试结果。审阅交付时内层改动未暂存/提交，最新提交状态另列。
- 范围：7 个 spec 主源码文件与 3 个 spec 测试文件，共新增 8 个 test case；没有修改任何 impl 或 build script。
- [ServiceTier 声明与选择语义：20–46 行](file:///home/stream/ACodeSpace/local/kodex-ultrafast-736/openai/spec/models/src/commonMain/kotlin/io/github/stream29/kodex/openai/OpenAiCommonModels.kt#L20-L46)：追加 Ultrafast，旧枚举顺序/wire 值不变；选择不表示权限或实际执行档位。
- [模型能力投影：80–101 行](file:///home/stream/ACodeSpace/local/kodex-ultrafast-736/openai/spec/models/src/commonMain/kotlin/io/github/stream29/kodex/openai/ModelCatalogModels.kt#L80-L101)：只认 exact id、枚举顺序、去重，不由模型名推断。
- [请求/返回值文档：19–79 行](file:///home/stream/ACodeSpace/local/kodex-ultrafast-736/openai/spec/models/src/commonMain/kotlin/io/github/stream29/kodex/openai/ResponsesApiModels.kt#L19-L79)和 Agent settings KDoc：请求值与返回值独立，实际值保持 nullable/open string。
- [Client 契约：32–49 行](file:///home/stream/ACodeSpace/local/kodex-ultrafast-736/openai/spec/client/src/commonMain/kotlin/io/github/stream29/kodex/openai/client/contract/OpenAiClient.kt#L32-L49)、[compaction 契约：172–180 行](file:///home/stream/ACodeSpace/local/kodex-ultrafast-736/openai/spec/client/src/commonMain/kotlin/io/github/stream29/kodex/openai/client/contract/OpenAiClient.kt#L172-L180)：请求档位不静默替换；两条请求使用同快照 body/routing hint；不向其他请求泄漏 header。
- New session defaults KDoc 保留 Default、缺字段和新旧 Session 边界；Runtime configuration KDoc 明确请求配置、能力不是权限，以及 Ultrafast 点选前的使用量提示。
- [新 ServiceTierSerializationTest：14–90 行](file:///home/stream/ACodeSpace/local/kodex-ultrafast-736/openai/spec/models/src/commonTest/kotlin/io/github/stream29/kodex/openai/ServiceTierSerializationTest.kt#L14-L90)含 5 项；catalog 新增 2 项、defaults 新增 1 项。旧 golden 未改。
- 审阅时检查：git diff 空白检查、新文件空白检查、spec-only 路径范围、无 impl/build-script 改动、未暂存、main 基线与工作区未动。源码/KDoc 与测试静态审阅完成。
- **审阅时未验证**：没有执行 Gradle、JVM/Native 测试或 CLI 编译；8 项测试当时只是已编写，不能报为通过。实现后的当前验证状态另列，不能用此快照替代。

## 当前实现与验证

- Ultrafast 主批已在原 `feat/ultra-fast` worktree 完成 scoped implementation；共 33 个 Kotlin 文件变更（16 个主源码、17 个测试文件，包含先前通过的 spec），已提交并推送为 `2399a6ea`。随后 ProMax 补丁以 `d85e0e61` 提交+推送，当前 worktree 干净；原 main 仍是 `5b41be52` 且干净。
- YAML decoder 接受 `ultrafast`，旧 `fast` / `priority` 别名保留。没有重写旧 Home 或 migrations，也不新增持久化配置真源。
- 四处 UI exhaustive 标签已补齐。两个 settings 页用 supportingText 提示使用量/权限；Runtime 和 Suggest subagent 菜单在 opt-in 前显示分行提示，短状态标签仍表示请求配置。内置 Astra / Sol 6.1 只补模型能力，远端成功完整替换，不合并补丁。
- [普通生成与 compaction 接线：245–282 行](file:///home/stream/ACodeSpace/local/kodex-ultrafast-736/openai/impl/client/src/commonMain/kotlin/io/github/stream29/kodex/openai/client/OpenAiClient.kt#L245-L282)调用同一个[私有 routing helper：366–371 行](file:///home/stream/ACodeSpace/local/kodex-ultrafast-736/openai/impl/client/src/commonMain/kotlin/io/github/stream29/kodex/openai/client/OpenAiClient.kt#L366-L371)。未改变旧 tier、body 默认省略、identity、beta、turn state 或重试预算；未向 Codex backend 增加 `OpenAI-Service-Tier`。
- [本地提示 cursor：8–45 行](file:///home/stream/ACodeSpace/local/kodex-ultrafast-736/app/impl/rpc/src/commonMain/kotlin/io/github/stream29/kodex/cli/rpc/UltrafastTierWarnings.kt#L8-L45)按 token-count index/nonce 和相邻 responseId 去重；初始历史、revert 基线、缺实际值/usage 记录、synthetic compaction 不提醒。[binding 观察与 Warning 接线：193–215 行](file:///home/stream/ACodeSpace/local/kodex-ultrafast-736/app/impl/rpc/src/commonMain/kotlin/io/github/stream29/kodex/cli/rpc/RpcAgentViewModel.kt#L193-L215)归本地 owner，关闭即取消；Warning 不覆盖尚未确认的 Error。无逐请求日志、额外轮询或新 RPC 契约。
- 已编写/扩展：旧值与 Ultrafast JSON/YAML、catalog 远端覆盖、四处 UI、compaction 固定快照、自动标题 Default，以及[真实 client loopback fixture](file:///home/stream/ACodeSpace/local/kodex-ultrafast-736/openai/impl/client/src/jvmTest/kotlin/io/github/stream29/kodex/openai/client/UltrafastRoutingHttpTest.kt#L20-L91)和[真实 JSON RPC fixture](file:///home/stream/ACodeSpace/local/kodex-ultrafast-736/app/impl/rpc/src/commonTest/kotlin/io/github/stream29/kodex/cli/rpc/UltrafastRpcIntegrationTest.kt#L18-L137)。HTTP 覆盖两模型、四 tier、普通/compaction 同 client header 隔离、models/search 无泄漏、400/401/403 不降级；RPC 覆盖 defaults CAS、draft 物化、持久化/CAS、fork、子 Session、重开、设置变更后的结果归属与 Error 优先级。
- 静态检查通过：tracked/new 文件空白、33 文件范围检查、无 build-script/gitlink/migration 改动、main 未动；源码/KDoc/fixture 交叉审查完成。确认的 enum 与请求边界已同步对应 OpenAI checklist。
- **未运行**：Gradle、JVM/Native 测试、CLI 编译/smoke 或 live 调用。[当前资源门禁](2026-10-04-plan-kodex-gradle-optimization.md#current-implementation-status)仍要求本机不构建、不接管 IDE；worktree 不解锁这些资源。未获取其他设备重型验证授权，不转移到共享设备抢资源。上文测试是代码覆盖，不是通过结果。
- 用户已允许在运行验证前提交+推送；保持 executable 和临时 worktree，不判定验收完成或可合并。有可用获准资源后执行下方定向测试与剩余验收矩阵，再等待运行验收与 main 集成。

## 提交与推送

- 用户明确要求“可以提交+推送”；已创建 [2399a6ea — feat: support ultrafast service tier](https://github.com/Stream29/Kodex/commit/2399a6eadcfd9a2c38452a69fc3e212a7b5161e8)，包含本任务全部 33 个 Kotlin 文件（848 行新增、25 行删除）。Spec/消费者/测试作为同一功能提交，不留下需补消费者的独立中间提交。
- 已推送至 Kodex `origin` 的 `feat/ultra-fast` 并设置 upstream；`git ls-remote` 确认远端完整 hash 与本地相同，ahead/behind 为 `0/0`，worktree 干净。没有 force push、合并 main、建立 PR 或发布 release。
- 提交正文明确写明仅做静态检查，Gradle/JVM/Native/CLI 运行验证未执行。推送成功不视为测试或验收通过。
- 用户再次明确要求提交推送 ProMax 补丁；已创建 [d85e0e61 — feat: recognize ProMax subscription plan](https://github.com/Stream29/Kodex/commit/d85e0e6179faaac08f8adb1da46a002669291f47)，仅 5 个声明/测试文件（106 行新增、3 行删除），普通 push 到同一 `origin/feat/ultra-fast`。远端完整 hash 核对一致，upstream ahead/behind 为 `0/0`；未 amend/force push，父提交仍为 `2399a6ea`。提交正文同样注明测试未运行。
- 外层 BuildKodex 的任务、调研与 checklist 记录保留在原工作区，本轮没有外层提交/推送或 gitlink 变更，未夹带其他任务现场。尚未合入 main，因此保留临时 worktree。

## ProMax 套餐兼容补充

- 用户听取“未知新套餐不阻断登录/请求，但套餐摘要缺少识别”的说明后，明确要求并入本分支；授权包括认证摘要复用枚举的 `ProMax` 扩展，不包括改名旧值、猜测价格映射或批量补齐其他套餐。
- 依据固定本地 Codex `604061ce51` 的 [auth.rs:74–95](file:///home/stream/ACodeSpace/push/BuildKodex/shared-context/codex/codex-rs/protocol/src/auth.rs#L74-L95)，只识别 `promax`；不把产品名 `pro500` / `pro_max` 当作未经证实的 JWT 别名，不固化套餐价格或能力。
- [套餐 spec:43–78](file:///home/stream/ACodeSpace/local/kodex-ultrafast-736/openai/spec/models/src/commonMain/kotlin/io/github/stream29/kodex/openai/OpenAiSubscriptionAuthState.kt#L43-L78)追加 `ProMax("promax")`，保留旧条目顺序/raw 值和摘要 wire 名称。既有大小写归一化自动接入；未知 raw claim 仍为 `null` 且不改变凭据可用性。原设置页直接显示 `rawValue`，无需新增展示 mapper。
- 本轮代码仅改 5 文件：1 个 spec 枚举/KDoc、4 个测试文件；新增 8 个测试用例（值模型 2、JWT/source/refresh 4、摘要 JSON 编解码 1、renderer 1）。摘要序列化固定为 `"ProMax"` 而非 JWT 的 `"promax"`；保留旧 nullable/no-token 形状。JSON 值往返不冒称真实 RPC 连接验收。
- JWT 测试覆盖两种凭据格式的 ProMax、大小写、未知套餐仍 Loaded、部分续期保留套餐以及新 ID token 更新套餐；renderer 测试断言 `Plan: promax`，没有套餐时不显示 Plan。没有新增按套餐判断模型权限、请求路由或本地用量配额的分支。
- 静态差异/空白、变更路径与消费者检查通过；Gradle、测试、CLI 和真实新套餐账号调用仍未执行，资源门禁不变。ProMax 已按用户最新授权提交+推送为 `d85e0e61`，HEAD/upstream 一致且 worktree 干净，保留已发布的 `2399a6ea` 历史。

## 实施范围

- 首期支持当前 **ChatGPT 订阅认证 → Codex backend → HTTP/SSE** 路径；两种现有凭据来源的所有权、刷新和登录规则不变。
- 接入既有 tier 选择、backend defaults、NewSession 草稿、persisted Session 和建议子 Session；不建立第二套 Ultrafast 配置。
- 主任务及 remote compaction 按实际请求快照发送选择；自动标题仍使用既有 Default，不因 defaults 选择而切换 tier。
- API-key 接入是独立认证/endpoint 工程，本次不实现。WebSocket、Responses Lite、模型退休、速度 benchmark、client-version 升级和 Gradle 修改不纳入。
- Ultrafast 是 service tier；ReasoningEffort 的 `max` / 历史 `ultra` 兼容逻辑不改。

## 已通过的契约调整

- 用户已接受 `openai/spec/models` 的 ServiceTier 增加 `Ultrafast("ultrafast")`，`@SerialName("ultrafast")`。
- 用户随后接受同分支的套餐兼容补充：`OpenAiSubscriptionPlan.ProMax("promax")`，认证摘要仍沿用既有枚举名序列化；具体边界见上方 ProMax 章节。
- ServiceTier 被 BackendSettings、KodexAgentSettings、create/fork Session 参数等跨线值复用，套餐枚举被认证摘要复用；扩展枚举取值也是 [RPC 契约冻结](../../checklist/rpc-architecture.md#rpc-契约冻结)内的调整，不能视为“RPC 文件没动所以无需审批”。
- 审批范围限于上述两个新增枚举取值；原 tier/套餐名称、序列化值、字段、RPC 方法、CAS 和取消语义均保持。
- 旧值照常读写；缺字段仍为 Default。旧二进制不保证能读取新写出的 `ultrafast`，因此测试使用隔离 Home，部署/回退前需显式移回旧支持档位，不自动改写用户配置。
- 无须 Home migration、设置文件版本或批量重写；既有历史 migrations 不修改。

## 模型与权限策略

- Runtime 三级菜单继续由 `ModelInfo.availableServiceTiers()` 派生；新增枚举后远端 `service_tiers` 的 `ultrafast` 可以被识别，未知 id 仍保留而不发送。
- 给内置 Astra / Sol 6.1 添加 Ultrafast **模型能力**描述，说明 higher usage / access required；这是根据官方产品资料的补充，不冒称它来自上游 bundled JSON，也不代表账号有权限。
- 同步修正 BuiltInModelCatalog 的来源说明。远端成功响应仍完整替换内置目录；远端没有 Ultrafast 时，不把内置项重新合并回去。
- Current session / New session defaults 保留当前完整枚举菜单，允许显式配置；不新增套餐推断、权限缓存或跨线 capability 服务。Runtime 缺元数据时仍只展示 Default。
- `planType`、内置目录和曾成功的一次请求都不等于当前 workspace entitlement；权限最终由服务端判断。账号切换不自动把旧 tier 改成 Default，也不声称缓存目录是新账号权限证明。
- 单独修改 model 保留 tier；可能形成不支持组合，需显示“access/model support required”，由用户更改。Runtime 三级菜单只在用户明确点选最终 `(model, effort, tier)` 时成组更新，不因目录刷新自动写设置。

## 请求协议

| 路径 | 请求 body | 本次新增 header | 保留行为 |
| --- | --- | --- | --- |
| 订阅普通生成，Ultrafast | `service_tier: "ultrafast"` | `x-codex-routing-hint: model=<请求 model>;tier=ultrafast` | SSE、identity、turn state、工具与重试 |
| 订阅 remote compaction，Ultrafast | 同上，使用 compaction 快照 | 同上，使用该快照 model | beta feature、20 次默认 retry budget、外部取消 |
| Default / Fast / Flex | 现有省略/`priority`/`flex` | 不自动新增 routing hint 或 API tier header | 已有 wire fixtures 保持 |
| 自动标题、models、usage、search、image 等 | 原请求 | 不泄漏上述 Ultrafast header | 原功能与认证 |
| API-key | 不新增产品路径 | 不套用订阅规则 | 仍 UnsupportedAuthMode |

- 在 `openai/impl/client` 内用一个私有请求 helper 处理两处 `/responses` 的 Ultrafast routing hint；不增加公共 transport API、注入外部 HttpClient 或模块。
- header 与 body 从同一个已经固定的请求值生成，不重读 live settings；每次请求局部设置，不装进全局 defaultHeaders。
- 不向订阅路径添加 `OpenAI-Service-Tier`。API FAQ 的 alpha header 规则与当前 API guide 的覆盖差异见调研记录，不能混用认证路径。
- 这仍是基于上游源码的接入方案，尚未证明真实 backend 接受；fixture 通过不得写成真实权限/速度验证。若有权限账户返回必须使用另一种 transport/header 的明确证据，先回到 Planning，不偷偷扩大本批。

## 展示、失败与实际档位

- 四处 exhaustive 标签补齐 `ultrafast`：Runtime configuration、Session settings、New session defaults、Suggest subagent task。状态栏继续表示**请求配置**，不是服务端实际执行证明。
- 两个 settings 页面使用既有 supportingText 提示更高使用量与权限要求；Runtime 的 Ultrafast 菜单项在点击前显示同类短提示，状态栏短标签保持可测量。
- 显式点选即 opt-in，不新建确认记录、永久“已读”字段或自动启用逻辑；取消菜单不写入，defaults 更新不改已有 Session。
- 不在客户端回退到 Fast/Default，也不通过省略 body 掩盖用户的 Ultrafast 选择。服务端实际返回别的 tier 与客户端主动降级是两件事。
- HTTP 400/401/403 等沿用既有失败路径；验证错误到达 Session 本地提示，宿主不退出，配置不改变。429/5xx/transport 保留现有有界重试与取消。
- `response.failed` 仍遵循既有有界 Agent retry，不能伪称本批已有精确 entitlement 错误分类。没有权威错误码 fixture 前，不按 message 子串猜错误，不修改所有档位的失败语义；最终错误仍须可见。
- 对已采集 TokenCountDiagnostics，保留 requested/returned 两个值；请求 Ultrafast、实际非空且不同时，在 RpcAgentViewModel 复用本地 AgentNotification.Warning 提示记录结果。
- Warning 只消费现有 binding 的完整 token-count 数据，不新建 RPC/后端 Notification。用现有 timeline 索引/cacheNonce 与 responseId 区分记录；忽略初次加载、revert 导致的旧记录和重复发布，覆盖 settings 已改变时的记录归属，避免把历史结果说成当前运行档位。
- 同一个 Session 的本地 Warning 不覆盖尚未确认的 Error；取消、缺实际 tier、synthetic compaction 0 不生成降级提醒。闭合 binding 时释放观察，不启动额外后台轮询。
- **可见性限制需接受**：diagnostics 是既有 best-effort 数据，usage 缺失或前端没观察到某个中间记录时，不承诺逐请求提醒；不为补提醒伪造 token count。需要可靠逐请求结果展示时，另行审批结果契约，不在本批暗加字段。

## 具体改动与验证落点

下表代码路径均相对 `Kodex/`，列实施与验证落点；调研时的基线行号见调研记录，当前实现入口见上方。

| 领域 | 改动落点 | 回归重点 |
| --- | --- | --- |
| 原值模型 | `openai/spec/models/.../OpenAiCommonModels.kt` | 新 enum 的 JSON round-trip；旧三值不变；catalog 未知值、重复值、缺字段 |
| Settings YAML | `app/settings/impl/filesystem/.../SplitSettingsFiles.kt` | 显式 decoder 加 ultrafast；fast/priority 别名不变；读取不重写、非法值仍失败 |
| 内置/远端目录 | `openai/impl/model-catalog/.../BuiltInModelCatalog.kt` | 两个模型的显式补充；远端覆盖而非合并；refresh 失败保留快照 |
| HTTP/SSE | `openai/impl/client/.../OpenAiClient.kt` | 普通生成/compaction 同源 body+hint；同 client 跨 tier、不同 model 的 header 隔离 |
| 四处 View | `app/component/{runtime-configuration,session-settings,new-session-defaults,suggest-subagent-task}/impl/view` | 标签、点击前提示、窄终端、取消不写入、子 Session 配置 |
| Runtime VM | `app/component/runtime-configuration/impl/viewmodel` | metadata 识别；修正“entries 等于 fixture 三值”测试，明确缺 Ultrafast 时不出现 |
| RPC 前端 | `app/impl/rpc/.../RpcAgentViewModel.kt` | 既有 diagnostics → Warning；旧记录/重复/Error 优先级、关闭后无观察 |
| 实际业务测试 | `rpc/impl/server`、`app/impl/rpc`、`agent-state/impl/state` | defaults/草稿/创建/CAS/fork/子 Session/RPC 字符串往返，请求/实际 tier 区别 |

- 不机械修改纯转发层。AgentState 普通请求和 compaction 当前已传 tier，优先加实际投影回归，而不是重写状态机。
- 复用 `OpenAiSubscriptionSerializationTest`、`ModelCatalogModelsSerializationTest`、`OpenAiModelCatalogTest`、`SplitSettingsStoreTest` 和相关组件测试。
- HTTP 测试仿照 [ExplicitReasoningHttpTest.kt:40–104](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/impl/client/src/jvmTest/kotlin/io/github/stream29/kodex/openai/client/ExplicitReasoningHttpTest.kt#L40-L104)的真实 client + loopback server；不为了测试暴露生产 HttpClient 构造。
- 既有 `response.serviceTier` 与 diagnostics 已可保存任意字符串，不增 DTO mapper、第二套实际 tier 状态或新的存储 schema。

## 验收矩阵

| 层级 | 必须覆盖 | 通过标准 |
| --- | --- | --- |
| 编解码 | Default/Fast/Flex/Ultrafast JSON；YAML fast/priority/ultrafast；稀疏旧文件、未知字段、非法 tier | 新值可往返；旧 golden 不变；不隐式迁移/改默认 |
| RPC/持久化 | backend defaults CAS；草稿不落盘；首消息创建；persisted Session CAS、重开、fork；子 Session | 原值不丢失，不绕过真实 RPC；旧契约只增加获批 enum 值 |
| Catalog/UI | 远端广告/缺失/未知/重复 tier；内置 fallback；tuple 选择；四处标签、窄屏提示 | 不猜账号权限、不自动写设置；当前选择仍可见 |
| 真实 HTTP fixture | 每 tier 普通请求与 compaction；Ultrafast→Default→Fast→Flex；两个 model；title/search 无泄漏 | exact body+header 正确；旧请求语义和 identity/turn state 不变 |
| 错误 | loopback 400/401/403、不支持组合；429/5xx；SSE failed；取消和 resume | 无客户端静默降级；失败提示可见；宿主可继续操作；重试按既有有界策略 |
| 返回档位 | 同值、不同值、未知字符串、缺值、有/无 usage、synthetic 0；重放/revert/关闭/Error | 记录真实值，Warning 不误归属、不重复、不覆盖 Error；缺数据不造结果 |
| CLI smoke | 临时 Home 的 defaults→新建→当前设置→子 Session→重开/退出；fixture-driven 请求与拒绝 | UI 能选、能保存、报错后可继续；不使用用户真实 Home |
| 平台 | 定向 JVM tests；可用 host Native 编译/测试；Linux CLI 链接与 smoke | 分别列实际运行、未覆盖与受阻平台；编译不替代 smoke |
| 可选 live | 有权限账号、极小普通生成；remote compaction 单独批准 | 先获付费授权；记录返回档位/请求 ID，不记录凭据，不承诺速度倍率 |

以下定向 JVM 入口已在合并验证 R03 执行；Native 另测 protocol models 并编译
受影响 client/RPC。完整 CLI 链接/smoke 仍不是这次通过项：

```text
:openai-spec-models:jvmTest
:openai-impl-client:jvmTest
:openai-impl-model-catalog:jvmTest
:app-settings-impl-filesystem:jvmTest
:app-component-new-session-defaults-spec:jvmTest
:app-component-new-session-defaults-impl-viewmodel:jvmTest
:app-component-new-session-defaults-impl-view:jvmTest
:app-component-session-settings-impl-viewmodel:jvmTest
:app-component-session-settings-impl-view:jvmTest
:app-component-runtime-configuration-impl-viewmodel:jvmTest
:app-component-runtime-configuration-impl-view:jvmTest
:app-component-authentication-settings-impl-view:jvmTest
:app-component-suggest-subagent-task-impl-view:jvmTest
:agent-state-impl-state:jvmTest
:rpc-spec-contract:jvmTest
:rpc-impl-server:jvmTest
:app-impl-rpc:jvmTest
:app-cli:compileKotlinJvm
```

- 真正 RPC fixture 用既有 in-memory JSON transport，mock 外部供应商而不 mock 声称已验证的 settings/storage 适配。
- Native target/tasks 与生产 CLI 链接命令在获准资源上核对，不为了取得绿灯减少 target 或 source set。

## 临时 worktree 与集成安排

- 用户指定的 `feat/ultra-fast` 与 `~/ACodeSpace/local/kodex-ultrafast-736` 已从 main `5b41be52` 创建；后续 spec 和获准 impl 都在此 worktree，不再创建原提议的 `feature/openai-ultrafast`。
- 创建前读 main、已有 worktrees、submodule 状态；主 `Kodex/` 不切分支，不复制或修改 Gradle 实验现场。初始化 fork 子模块/二进制消费按现行配置，不更新 gitlinks。
- 外层任务文档继续由 BuildKodex 保存；产品代码只在临时内层 worktree。方案落实不等于允许更改外层子模块指针或夹带其他 Session 文档。
- Worktree 只隔离源码。构建仍须遵守设备资源锁、已有 Gradle Daemon JVM 和用户占用规则；本机重型构建暂停约束不因 worktree 消失，不创建第二个 daemon 抢资源。
- 若资源忙，保留实现并报告验证受阻，不把 static review 当运行验收。
- 先前授权仅限提交+推送 feature；2026-10-11 最新授权已推进本地 main 合并
  及外层 gitlink 更新，不含 main 推送或 release。合并保留其他 Gradle 改动。
  临时 worktree 由原 Session 持有，仅在确认它已释放使用且没有未保存内容时清理。
- 只有确认改动已经合入 main、worktree 无未保存内容且没有活跃任务进程，才正常 `git worktree remove`；分支按可验证的已合并状态清理，不用 force 删除。
- 清理只针对本任务拥有的临时 Home、测试 server、构建/fixture 资源及 worktree，不执行全局 `gradle --stop` 或清除共享缓存。

## 实施与验收门槛

- Spec review 已通过，ProMax 补充亦获具体授权；实施订阅 SSE、新 tier、内置模型能力、best-effort 本地提醒及套餐识别，不增加其他跨线契约。
- 运行验证遵守当前本机构建暂停和设备锁，不因 worktree 取消资源限制；受阻检查列明，不能将静态检查作为测试通过。
- Feature 提交+推送已获授权，运行验收仍待完成；合并前复核最新 main，确认已合入且临时 worktree 干净后才清理。

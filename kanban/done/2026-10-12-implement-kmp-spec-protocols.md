# Task Tree

- `Load authorized scope, canonical rules and pilot state`()
- `Classify every protocol spec and freeze local destinations`()
- `Implement protocol source sets and five OpenAI consolidations`()
- `Check static source preservation and build declarations`()
- `Hand off settings and cross-domain ID reconciliation`()
- `Accept scoped repository validation and archive the implementation task`()

# Details

## 收尾验收（当前）— Session 787 / 2026-10-13

- 用户授权“收尾，没有问题则标记完成”；协调者已完成冻结 `bbbc9525` 的
  全仓迁移检查，以及 remote MacBook 的真实 IDEA/导航、同预算暖 Sync、
  增量对照、metadata、完整 Release CLI 与 112 次原生测试；本线纳入主任务 scoped 验收，
  已移入 done。证据已保留并核对，私有验证副本及拥有资源已清理。
- 原四个运行时失败与两个测试编译缺口已在干净基线复现，不算通过；
  live/原禁用 IO、不可用宿主及外部已发布二进制兼容不在完成声明内。
- 详细结果见 [主任务](2026-10-12-discuss-kmp-spec-impl-source-sets.md#L1)；
  下文是实施与交接历史，旧“未验收/留 executable”不再代表当前状态。

## 协调者整合回填（历史）

- settings/全仓消费者已切换，完整模型、生产图和实际 JVM/JS/Linux main 编译
  已验证。测试与干净基线缺口见 [主任务](2026-10-12-discuss-kmp-spec-impl-source-sets.md#L1)。
- 下文静态计数、待注册/动态验证清单是本线交接时记录；重型操作由协调者
  在授权私有副本执行。本线仍等待整仓人工验收，不把子线静态核查写成全绿构建。

- 状态：源码完成、静态核验完成，**待验收**；留在 executable，不标 done。
  本文件为协议子线独占报告，不修改主任务或全局 checklist。
- 用户授权：仅 `Kodex/openai`、`rpc`、`mcp`、`hook` 产品与本报告。
  只用 apply_patch 编辑；不运行 Gradle、编译、IDE、远程，不提交或创建 Session。
- 基线：`450e3bd7eeef460c7293d6f6fedb55cc1a4242c5` 加现有试点；
  `hook/spec/notification` 已在 specMain，不回退。
- 已加载 AGENTS、buildkodex-change、checklist/kanban/programmatic-planning、
  ask-user、ide-collaboration、better-document-workflow、toolchain-preference。
  主任务已记录 IDEA 打开项目；本轮不调用或控制 IDE。
- 输入：
  [主任务](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-12-discuss-kmp-spec-impl-source-sets.md#L1)、
  [canonical 规范](file:///home/stream/ACodeSpace/push/BuildKodex/checklist/kmp-spec-impl-source-sets.md#L1)、
  [历史静态审查](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-12-audit-kmp-spec-protocols.md#L1)。
  Draft 无待提取项；历史报告的 commonMain 原位保留建议不覆盖新的源集迁移授权。

## 完整逐 spec 去向

- 路径相对 `Kodex/`；所有包、公开 Kotlin 名称、serial 形状和默认行为保持。
- 新项目 ID 按目录平坦化；settings 和跨域旧 ID → 新 ID 由协调者统一。
  本子线可暂留旧引用，必须删掉合并自依赖。

| 旧 spec / 配对 | 目标项目 / ID | source 分类 | 当前 Kotlin 文件数 |
| --- | --- | --- | --- |
| `openai/spec/account-usage` + `impl/account-usage` | `openai/account-usage` / `:openai-account-usage` | Store、状态、快照及 snapshotOrNull 模型投影进入 specMain；观察/刷新实现 commonMain | S1 / M1 / T1 |
| `openai/spec/client` + `impl/client` | `openai/client` / `:openai-client` | 四个 contract 文件及 compaction exceptions 进入 specMain；HTTP/config/login 实现 commonMain；close 默认行为不改 | S5 / M3 / T6 / JT3 |
| `openai/spec/client-test` + `impl/client-test` | `openai/client-test` / `:openai-client-test` | Mock builder 配置契约进入 specMain；mock 与 in-memory auth 实现 commonMain；仍为可消费 main fixture 库 | S1 / M2 / T1 |
| `openai/spec/codex-cli-storage` + `impl/codex-cli-storage` | `openai/codex-cli-storage` / `:openai-codex-cli-storage` | 只读 port、模型及 untagged TOML serializer 进入 specMain；文件 I/O commonMain | S2 / M1 / T1 |
| `openai/spec/model-catalog` + `impl/model-catalog` | `openai/model-catalog` / `:openai-model-catalog` | Store port 进入 specMain；内建目录及在线 store commonMain | S1 / M2 / T1 |
| `openai/spec/models` | 保留 / `:openai-spec-models` | 模型及必需 serializer 进入 specMain；OpenAiResult 的 getOrThrow / throwIfFailure 操作放 commonMain | S20 / M1 / T14 |
| `openai/spec/json-codec` | 保留 / `:openai-spec-json-codec` | OpenAiJsonCodec 的 Json 配置实现保留 commonMain；specMain 无业务声明，不造空接口 | M1 |
| `rpc/spec/contract` | 保留 / `:rpc-spec-contract` | 8 个 @Rpc 服务、TimelineRpc、SuspendMutableStateFlow、SessionTitleGenerator 共 11 文件进入 specMain | S11 / T14 |
| `rpc/spec/models` | 保留 / `:rpc-spec-models` | 9 个协议/持久化模型文件进入 specMain | S9 / T7 |
| `mcp/spec/contract` | 保留 / `:mcp-spec-contract` | Client/Manager/Service/Settings/Tool 共 5 文件进入 specMain；DraftConfiguration/ModelToolNames 转换与算法保留 commonMain | S5 / M2 / T2 |
| `mcp/spec/stdio` | 保留 / `:mcp-spec-stdio` | 真实进程 transport 全部保留 commonMain；specMain 无业务声明，不拆占位 port | M1 / T1 / JT2 / JST1 |
| `mcp/spec/streamable-http` | 保留 / `:mcp-spec-streamable-http` | HttpClient/SSE/授权拦截/transport 实现全部保留 commonMain；engine 原平台层；specMain 无业务声明 | M1 / T1 |
| `hook/spec/notification` | 保留 / `:hook-spec-notification` | NotificationHook 原试点 specMain 保持不变，executor 仍独立 | S1 / T1 |

- S/M/T/JT/JST = specMain/commonMain/commonTest/jvmTest/jsTest 文件数，
  不是测试用例数或执行结果。

## 分类依据与不变量

- 不是把旧 spec 中有函数体的声明都视作契约：
  Json 配置、真实 transport、编辑转换、route 算法、Flow adapter 是实现。
  本域没有独立 prompt DSL 源码；MockOpenAiClientBuilder 是真实配置接口，
  其执行 builder 在旧 impl，不把实现 builder 移进 specMain。
- 模型注解直接引用的 serializer 必须与模型一起位于 specMain：
  OpenAiJson、CodexResponsesMetadataSerializer、OpenAiResultSerializer、
  SearchInputSerializer、CodexCliMcpServerSerializer 及 McpSecret serializer。
  不允许 specMain 反向引用 commonMain serializer。
- snapshotOrNull、模型计算属性、context-window/service-tier 的规范化投影、
  codexRequestWindowId 与模型工厂作为现有模型语义保留；
  不改变无副作用的契约默认行为。
- 13 个原 spec owner 均使用前置 `kodex.kmp-spec-impl`；
  真实 commonMain → specMain 由现有 convention 提供。
  无本地契约的三个实现库仍保留原项目，空 specMain 不等于制造空接口。
- 五个合并项目保留 Host targets 和插件并集，契约依赖迁至 named("specMain")，
  实现和测试保持原源集。commonMain 可继承契约依赖，但不逆向供给。
- RPC contract 保留 serialization/kRPC/patch plugins；RPC 实现项目不合并。
  `rpc/impl/krpc-utils-patch`、substitution 和 Native moduleName 不变。
- 保留 commonTest/jvmTest/jsTest 及 fixture；不新建 specTest、不提升测试依赖。
  不移动 build 下产物，不触碰工具链和根构建。

### 混合项目的具体落点

- OpenAI models 的 specMain 文件：`AgentModels`、`CodexAccountUsageModels`、
  `CodexResponsesMetadataModels`、`CodexResponsesMetadataSerializer`、`CompactionModels`、
  `FunctionCallOutputModels`、`ImageGenerationModels`、`ModelCatalogModels`、
  `OpenAiAuthState`、`OpenAiCommonModels`、`OpenAiJson`、`OpenAiLoginModels`、
  `OpenAiResultModels`、`OpenAiSubscriptionAuthState`、`PlanToolModels`、
  `ResponseItemId`、`ResponseItemModels`、`ResponsesApiModels`、`ToolSpecModels`、
  `WebSearchModels`（均 `.kt`）。
- 原 `OpenAiResult.kt` 中模型、typealias、异常、serializer 进入
  [OpenAiResultModels.kt](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/spec/models/src/specMain/kotlin/io/github/stream29/kodex/openai/OpenAiResultModels.kt#L1)；
  两个操作函数及其 KDoc 保留在
  [commonMain/OpenAiResult.kt](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/spec/models/src/commonMain/kotlin/io/github/stream29/kodex/openai/OpenAiResult.kt#L6)。
  保留函数原 JVM 文件门面名称，避免两层同名 OpenAiResultKt；未改 serializer 身份。
- FunctionCallOutput 的转换和私有辅助函数保留在模型层：
  它们是现有 DTO 的成员语义及 serializer 所需 wire 投影，不依赖网络/进程/存储实现。
  不为将成员下移而改成扩展函数或把整个 DTO 归入实现。
- MCP 契约的源码位置为
  [specMain/McpClient.kt](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/spec/contract/src/specMain/kotlin/io/github/stream29/kodex/mcp/contract/McpClient.kt#L1)
  等五文件；编辑转换的全部私有 secret helpers 跟随
  [commonMain/McpDraftConfiguration.kt](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/spec/contract/src/commonMain/kotlin/io/github/stream29/kodex/mcp/contract/McpDraftConfiguration.kt#L1)，
  路由算法在
  [commonMain/McpModelToolNames.kt](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/spec/contract/src/commonMain/kotlin/io/github/stream29/kodex/mcp/contract/McpModelToolNames.kt#L1)。
  没有在五个 specMain 文件中发现对这些实现 helpers 的引用。
- transport/codec 的实际声明仍在
  [McpStdioTransport.kt](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/spec/stdio/src/commonMain/kotlin/io/github/stream29/kodex/mcp/stdio/McpStdioTransport.kt#L36)、
  [McpStreamableHttp.kt](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/spec/streamable-http/src/commonMain/kotlin/io/github/stream29/kodex/mcp/streamablehttp/McpStreamableHttp.kt#L28)、
  [OpenAiJsonCodec.kt](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/spec/json-codec/src/commonMain/kotlin/io/github/stream29/kodex/openai/jsoncodec/OpenAiJsonCodec.kt#L5)。

## 协调者接线

| 两个旧 ID | 唯一新 ID |
| --- | --- |
| `:openai-spec-account-usage`、`:openai-impl-account-usage` | `:openai-account-usage` |
| `:openai-spec-client`、`:openai-impl-client` | `:openai-client` |
| `:openai-spec-client-test`、`:openai-impl-client-test` | `:openai-client-test` |
| `:openai-spec-codex-cli-storage`、`:openai-impl-codex-cli-storage` | `:openai-codex-cli-storage` |
| `:openai-spec-model-catalog`、`:openai-impl-model-catalog` | `:openai-model-catalog` |

- 将 OpenAI 五对注册改为五个新目录，保留 models/json-codec 两个旧项目；
  不注册兼容转发项目。
- 统一全部域内与跨域 project 引用；本线仅移除五个合并自依赖。
  当前局部目录迁移不等于 settings 已接好或仓库已可构建。
- RPC/Hook/MCP 保持原边界，避免 Hook → RPC models → Hook 合并环
  和 MCP composition → transport → composition 半合并环。
- 测试图独立核验：client.commonTest → client-test.main → client.main
  不是生产环；不能提升为 main 依赖。

## 检查与风险

- 静态核验已完成：以 HEAD 的跟踪文件逐项映射到当前路径；
  四域基线 209 个 Kotlin 文件，208 个在目标位置逐字节相同。
  另一个 OpenAiResult 文件仅作原片段拆分，函数体/KDoc/模型/serializer
  均与原片段一致，仅 spec 文件尾部空行归一化；当前共 210 文件。
  本轮移动 77 文件，另 1 个 Hook 路径变化来自已有试点，不重复计为本轮实施。
- 所有 commonTest/jvmTest/jsTest 文件逐字节保持；五个合并项目的测试依赖块
  与原 impl 相同，保留项目测试依赖块也与基线相同。
- 5 个合并项目插件并集完整、前置 convention 顺序正确；
  契约依赖块等于原 spec，完整依赖声明 multiset 等于两侧并集减自依赖。
  7 个本轮迁移的保留项目依赖声明 multiset 不变。
  不移除 commonMain 与 specMain 之间原本各自声明的重复依赖。
- 13 个 spec owner 的 convention 均先于 Host/CLI；原 Host/CLI targets convention
  保留，HTTP 的 JVM/linux/macosArm64/mingwX64 engine 配置块逐字相同。
  四域原脚本未声明单独 languageSettings/生成目录配置；源码 @OptIn 保持，
  未改变全局 compiler/tests convention，不宣称生成插件已验收。
- patch build 与 substitution convention 均与基线逐字节一致，
  `moduleName = "org.jetbrains.kotlinx:utils"`、group、atomicfu、expect/actual 原位保留；
  本轮没有修改 RPC 实现插件，也没有改 Hook 产品文件。
- 文本候选检查未发现同项目/同包/同 basename 且包含顶层声明的 Main 文件门面冲突。
  这不是 Kotlin 编译器的完整符号/重载/JVM 签名证明。
- 对四域当前 19 个构建脚本按五对映射投影：31 条域内 Main 边、
  15 条域内 Test 边，Main 未发现自边或环；不包含跨域反馈路径和动态 variant，
  不声称 settings 已注册 19 个项目或全仓无环。
- `git diff --check -- openai rpc mcp hook` 无输出；额外检查覆盖未跟踪目标源码，
  不是仅靠 Git diff 的删除统计断言搬迁成功。静态脚本以内存读比较运行，
  没有写临时脚本、探针或测试产物。
- 待授权构建：specMain 的 serializer/kRPC 生成、metadata、真实消费者、
  所有原目标测试及 JS/Native variants；不能外推 Hook 试点证据。
- 待协调者：settings、全仓旧 ID 替换、全局 checklist 与分源集依赖图。
- 待用户：IDE Sync/导航/索引、wire/持久化回归及最终验收。
- 本报告不记录编译、测试、性能“通过”。所有编辑都在四域及本报告；
  并行子线的其他域和协调者文件修改不接管、不回退。
  没有启动设备、端口、后台进程或共享计算任务，无临时资源需释放。

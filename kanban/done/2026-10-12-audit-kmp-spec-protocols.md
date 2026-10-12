# Task Tree

- `Inventory and review every protocol spec project`()
- `Propose consolidation mapping and identify concrete hazards`()
- `Record source evidence and validation requirements`()

# Details

- Parent: [KMP source-set migration plan](../executable/2026-10-12-discuss-kmp-spec-impl-source-sets.md).
- Scope: all `openai`, `rpc`, `mcp`, and `hook` projects in `Kodex/`,
  including implementations, test support, and patch/substitution consumers.
- Baseline: `4388871137c56bd1b73d5b739cc98f036a790f3c`; verify HEAD before/after review.
- Read [confirmed design](../../checklist/kmp-spec-impl-source-sets.md).
  Target is platform/intermediate -> commonMain (impl) -> specMain.
- Static audit only. No build, product edits, commits, device/IDE control, or new Sessions.
- Only this report is writable by this review line.
- Deliver a complete per-spec inventory, concrete old->new project/source mapping,
  mechanical moves vs decisions, local and cross-domain edges, and file/line evidence.
- Review wire compatibility, serialization/RPC generators, substitution, pure model
  specs, multi-implementation pairs, platform coverage, test-only specs, and cycles.
- Distinguish observed blockers, plausible unverified risks, and checks needed.
  Do not claim compilation or performance results from source inspection.

## Review result

### 1. 边界、基线和结论级别

- 审查日期：2026-10-12。开始时 `Kodex/` HEAD = `4388871137c56bd1b73d5b739cc98f036a790f3c`，工作区干净。
  外层 BuildKodex HEAD = `c0a7d34c4d8a5a7e69538e68d80b56d5fd0bc671`，已有其他任务的修改；二者不能混为一个基线。
- 仅使用跟踪文件清单、源码、Gradle 脚本文本及依赖声明做静态检查。
  目录里残留的旧 `build/`、旧模块产物不算当前项目，未用旧测试报告或二进制推断本轮通过。
  未运行 Gradle、编译、IDE、远程设备或新 Session；仅此文件可写。
- 采用已经确认的 `platform/intermediate -> commonMain(impl) -> specMain`；
  真正创建 `specMain`，不创建 `implMain`。消费者使用组合项目及其实现 API 合法。
  `api`/`implementation` 不代替契约职责约束，也不提供 spec-only 产物。
- 接受精确配置 `kotlin.internal.suppressGradlePluginErrors=CommonMainOrTestWithDependsOnDiagnostic`。
  其含义、升级风险、同 ID 对 commonTest 的影响以
  [已确认设计 L50–73](file:///home/stream/ACodeSpace/push/BuildKodex/checklist/kmp-spec-impl-source-sets.md#L50) 为准；
  不是本子任务重新验证的官方工具链行为，更不是 Kodex 构建/性能验收。
- 下面 **M** = 边界决定后可机械搬迁；**D** = 必须由用户决定合并单元/职责；
  **B** = 后续获准构建才可证明。标为“确定阻断”的是给定候选映射下的文本依赖环，不是声称当前基线已坏。

**摘要**

- 注册叶项目共 **24 个：13 spec + 11 impl**，没有遗漏未配对 spec。
- 相对直接候选是 OpenAI 的 account-usage、client、client-test、codex-cli-storage、model-catalog 五组；
  “client-test”必须保留为独立可复用测试支持库，而非移入 client 的不可消费 commonTest。
- `openai/models`、`rpc/models` 没有同名 impl；`openai/json-codec` 是有意义的可执行协议策略。
  不为这些项目凭空制造 impl。
- Hook notification 一对一合并会与 RPC models 形成确定主依赖环。
  MCP contract 只合 composition 而不合 transports 也形成确定主依赖环，并存在 Host/CLI 平台差异。
- RPC contract 是多个服务及本地 port 的集合，不是 server 的一对一镜像。
  RPC utils patch 是外部 ABI 替换，不能当普通 impl 顺手并入。

### 2. 全部 spec 清单、类型与目标路径

计数为跟踪 Kotlin 文件数，不是声明数。`M/T/JT/ST` 分别指 commonMain/commonTest/jvmTest/jsTest。
以下项目 ID 均为真实旧 ID；目录到平坦 ID 的规则见
[settings L52–86](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/settings.gradle.kts#L52)。
13 个注册点可交叉核对
[RPC/MCP/OpenAI L317–357](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/settings.gradle.kts#L317)
及 [Hook L432–438](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/settings.gradle.kts#L432)。

| 旧 spec（构建证据） | 完整主源码文件清单；测试规模 | 分类、配对与候选目标 |
| --- | --- | --- |
| [openai-spec-account-usage](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/spec/account-usage/build.gradle.kts#L1) | `CodexAccountUsageStore.kt`；1M/0T | 状态/操作契约及可序列化快照；配 `openai-impl-account-usage`。**M/B** → `:openai-account-usage`，`Kodex/openai/account-usage/src/specMain/kotlin`；impl →同项目 `src/commonMain/kotlin`。 |
| [openai-spec-client](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/spec/client/build.gradle.kts#L1) | `OpenAiRemoteCompactionExceptions.kt`；`contract/{OpenAiAuthStore,OpenAiClient,OpenAiLoginClient,OpenAiResponseHeaders}.kt`；5M/0T | 多 port；HTTP client/login 在 `openai-impl-client`，AuthStore 另在 RPC server/test-support 实现。**M/D/B** → `:openai-client`，`Kodex/openai/client/src/specMain/kotlin`；只移本地 impl 到 commonMain，不吞并 backend auth 或 mocks。 |
| [openai-spec-client-test](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/spec/client-test/build.gradle.kts#L1) | `MockOpenAiClientBuilder.kt`；1M/0T | **测试专用 spec**，配 `openai-impl-client-test` 的 builder/mock/auth store。**M/B** → `:openai-client-test`，`Kodex/openai/client-test/src/specMain/kotlin`；可复用实现仍在 commonMain，自己的测试在 commonTest。 |
| [openai-spec-codex-cli-storage](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/spec/codex-cli-storage/build.gradle.kts#L1) | `CodexCliStorage.kt`、`CodexCliStorageModels.kt`；2M/0T | 只读存储 port + JSON/TOML 模型/serializer；配 `openai-impl-codex-cli-storage`。**M/B** → `:openai-codex-cli-storage`，`Kodex/openai/codex-cli-storage/src/specMain/kotlin`；impl commonMain。 |
| [openai-spec-json-codec](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/spec/json-codec/build.gradle.kts#L1) | `OpenAiJsonCodec.kt`；1M/0T | 共享 wire JSON policy，无 impl 项目。**D**：默认保留旧路径/ID/commonMain。若仅统一布局，可另选 `:openai-json-codec` / `Kodex/openai/json-codec/src/specMain/kotlin`，但这不是消除 spec/impl 配对；不能吸入 HTTP client 后让纯模型测试依赖整个 client。 |
| [openai-spec-model-catalog](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/spec/model-catalog/build.gradle.kts#L1) | `OpenAiModelCatalogStore.kt`；1M/0T | port，配 `openai-impl-model-catalog`；另有 session-test 的静态实现。**M/B** → `:openai-model-catalog`，`Kodex/openai/model-catalog/src/specMain/kotlin`；impl commonMain。 |
| [openai-spec-models](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/spec/models/build.gradle.kts#L1) | `AgentModels`、`CodexAccountUsageModels`、`CodexResponsesMetadataModels`、`CodexResponsesMetadataSerializer`、`CompactionModels`、`FunctionCallOutputModels`、`ImageGenerationModels`、`ModelCatalogModels`、`OpenAiAuthState`、`OpenAiCommonModels`、`OpenAiJson`、`OpenAiLoginModels`、`OpenAiResult`、`OpenAiSubscriptionAuthState`、`PlanToolModels`、`ResponseItemId`、`ResponseItemModels`、`ResponsesApiModels`、`ToolSpecModels`、`WebSearchModels`（均 `.kt`）；20M/14T | **模型/协议语义库**，含定制 serializer、值计算、Flow 失败语义，不是只含 DTO 的空壳。无同名 impl。**D**：默认保留；独立改布局候选 `:openai-models` / `Kodex/openai/models/src/specMain/kotlin`，测试仍 commonTest。不建议合入 client。 |
| [rpc-spec-contract](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/spec/contract/build.gradle.kts#L1) | `AgentRuntimeRpc`、`GlobalRpc`、`IndexTimelineRpc`、`SettingsTimelineRpc`、`SuspendMutableStateFlow`、`TimelineRpc`、`TimestampTimelineRpc`、`TokenCountTimelineRpc`、`UnstableTimelineRpc`、`WorkTimelineRpc`、`cli/sessiontitle/SessionTitleGenerator`（均 `.kt`）；11M/14T | 多服务 + 普通通用接口 + backend-local port。**D/B**：默认独立保留；候选与 `rpc-impl-client` 合为 `:rpc-client` / `Kodex/rpc/client/src/specMain/kotlin`，adapter 在 commonMain，server/in-memory/patch 保持外部。另一个候选是 contract+client+server → `:rpc-service` / `Kodex/rpc/service/`，须独立全图复审，不能称一对一机械合并。 |
| [rpc-spec-models](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/spec/models/build.gradle.kts#L1) | `AgentStateValue`、`BackendSettings`、`CliFrontendSettings`、`CliSidebarSettings`、`CreatedSuggestedSession`、`Notification`、`OAuthAuthorization`、`OAuthTarget`、`ShellSessionState`（均 `.kt`）；9M/7T | **模型库**，但兼有前端持久化配置，不能按“只走 RPC”裁剪。无 impl。**D**：默认保留；独立布局候选 `:rpc-models` / `Kodex/rpc/models/src/specMain/kotlin`；不与 notification executor 做机械折叠。 |
| [mcp-spec-contract](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/spec/contract/build.gradle.kts#L1) | `McpClient`、`McpDraftConfiguration`、`McpManager`、`McpModelToolNames`、`McpService`、`McpSettings`、`McpTool`（均 `.kt`）；7M/2T | 多 port、模型及纯规则；配 `mcp-impl-composition` 的多实现。**D/B**：默认保留；完整候选须 contract+composition+stdio+streamable-http → `:mcp-composition` / `Kodex/mcp/composition/src/specMain/kotlin`（契约）、`src/commonMain/kotlin`（真实 transport/composition），并解决 JS。仅 contract+composition 不可行，见 F2。 |
| [mcp-spec-stdio](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/spec/stdio/build.gradle.kts#L1) | `McpStdioTransport.kt`；1M/1T/2JT/1ST | **名称是 spec、内容是具体传输实现**，无同名 impl。**D**：不原样全移 specMain；完整 MCP 候选中该文件去 commonMain、测试保持相应 Test 源集。若保留独立项目，可候选 `:mcp-stdio` / `Kodex/mcp/stdio/src/commonMain/kotlin`，不伪造空接口以凑 specMain。 |
| [mcp-spec-streamable-http](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/spec/streamable-http/build.gradle.kts#L1) | `McpStreamableHttp.kt`；1M/1T | **具体 HTTP/SSE/授权重试实现**，无同名 impl。**D**：完整 MCP 候选中去 commonMain；独立整理候选 `:mcp-streamable-http` / `Kodex/mcp/streamable-http/src/commonMain/kotlin`，engine 依赖仍在原平台层。 |
| [hook-spec-notification](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/hook/spec/notification/build.gradle.kts#L1) | `NotificationHook.kt`（注意包名为 `rpc.models`）；1M/1T | **纯配置模型**，不是 executor interface。`hook-impl-notification` 消费它与 RPC models。**D，直接合并确定阻断**：`hook/spec/notification + hook/impl/notification → :hook-notification / Kodex/hook/notification/{specMain,commonMain}` 会成环；默认保留原模型边界。另选把模型归入 rpc-models 须用户确认所有权。 |

所有选中配对遵循统一搬迁规则：

- 旧 spec `src/commonMain/kotlin/<包路径>` → 表中目标 `src/specMain/kotlin/<原包路径>`。
  保留包、声明名、KDoc、注解、默认值；**目录移动不等于 package 改名**。
- 旧 impl `src/commonMain/kotlin` → 目标同名源集；旧 spec/impl 测试汇入对应
  `commonTest`/`jvmTest`/`jsTest`，先核对重复顶层私有声明和 Test suite 名称。
- 旧项目依赖指向新 ID，移除合并单元内的自依赖；原 spec 的库/项目依赖迁到 specMain，
  原 impl 的依赖留 commonMain。不可只把两份 commonMain 依赖合并后丢失 spec 的输入。
- 名称和目录是**候选**，不是已批准迁移；保留项目无需为统一外观强行增加空层。
- “有函数体”不自动等于实现层：`McpDraftConfiguration`、`McpModelToolNames` 的纯校验/投影
  和模型 serializer 是已有契约语义，默认随契约保留；stdio/HTTP 的进程与网络资源管理才是这里识别的实现。

### 3. 实现、测试支持与一对多关系

| 实现项目（全部 11 个） | 内容、测试和依赖关注点 |
| --- | --- |
| [openai-impl-account-usage](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/impl/account-usage/build.gradle.kts#L1) | 1M/1T：`CodexAccountUsageStoreImpl` / `CodexAccountUsageSnapshotTest`。主依赖 account-usage spec、client spec、models。原 impl **没有 serialization 插件**，合并时须保留 spec 插件。 |
| [openai-impl-client](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/impl/client/build.gradle.kts#L1) | 3M/6T/3JT：HTTP client、config、login；依赖 client/models/json-codec、utils-ktor-client-ext-impl、utils-os-environment-spec，测试依赖 client-test。真实 HTTP/登录/重试/SSE，不是接口占位。 |
| [openai-impl-client-test](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/impl/client-test/build.gradle.kts#L1) | 2M/1T：`MockOpenAiClient.kt`、`InMemoryOpenAiAuthStore.kt` / `MockOpenAiClientTest`。main 依赖 client-test/client/models；测试库消费者包括 agent-session/test 和 app/test-support/rpc 的 **commonMain**，不是全部 commonTest。 |
| [openai-impl-codex-cli-storage](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/impl/codex-cli-storage/build.gradle.kts#L1) | 1M/1T：`CodexCliStorageImpl` / `CodexCliStorageTest`；另依赖 utils-kotlinx-io-coroutines-impl、json-codec。文件 I/O 不是模型职责。 |
| [openai-impl-model-catalog](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/impl/model-catalog/build.gradle.kts#L1) | 2M/1T：`BuiltInModelCatalog`、`OpenAiModelCatalogStoreImpl` / `OpenAiModelCatalogTest`；依赖 client、model-catalog spec；测试用 client-test。 |
| [rpc-impl-client](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/client/build.gradle.kts#L1) | 4M/9T：`RestoringRpcClient`、`RpcCachedIndexVersioned`、`RpcStateIn`、`SuspendMutableStateFlow`；依赖 contract、agent-storage-spec-contract、utils-rpc-exception-spec、cache4k；测试依赖 in-memory。不是全部 RPC 服务的唯一业务实现。 |
| [rpc-impl-in-memory](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/in-memory/build.gradle.kts#L1) | 2M/2T：`ChannelRpcTransport`、`WithInMemoryRpc` 及对应测试。main **不依赖 rpc-spec-contract/models**，是可复用 kRPC transport/注册 harness，应用主图也使用；不能仅因名称归入业务 contract 实现或移至 test。 |
| [rpc-impl-server](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/server/build.gradle.kts#L1) | 21M/17T：backend services/runtime/global/timelines/session host，外加 auth、标题生成；依赖 session/state/storage、settings persistence/filesystem、MCP composition、OpenAI 四种运行时实现及 client/models/json-codec、shell/logging/I/O utils。测试又依赖 rpc-client/in-memory、app-impl-rpc、session-test/in-memory、client-test；不是只依赖自身 spec 的简单配对。 |
| [rpc-impl-krpc-utils-patch](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/krpc-utils-patch/build.gradle.kts#L1) | 7M/2 jvmMain/2 nativeMain/1T：`expect/actual` map/thread-local +快照 map，外部 kRPC ABI 替换；无本域 spec，保持独立，见 F5。 |
| [mcp-impl-composition](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/impl/composition/build.gradle.kts#L1) | 6M/4T/3JT：client/manager/service/tool/config validation/OAuth；主依赖 contract、两个 transports、SDK、utils-coroutines/logging/process/read-write-mutex、Ktor server CIO。测试依赖 agent runtime decorator-tool、state/state-test、storage-in-memory、tool-plan、OpenAI mock/codec；不能把这些测试边当主图边。 |
| [hook-impl-notification](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/hook/impl/notification/build.gradle.kts#L1) | 1M/2T：`NotificationHooks` / `NotificationHooksTest`、`NotificationHooksRpcTest`；主依赖 notification model、RPC models、shell impl/coroutines，测试依赖 RPC 三项目和 OpenAI mock。serialization 插件在 spec，不在当前 impl。 |

一对多/跨域实现证据：

- `OpenAiClient` 有
  [真实 HTTP 实现 L90–94](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/impl/client/src/commonMain/kotlin/io/github/stream29/kodex/openai/client/OpenAiClient.kt#L90)
  与 [mock factory/builder L31–40](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/impl/client-test/src/commonMain/kotlin/io/github/stream29/kodex/openai/client/test/MockOpenAiClient.kt#L31)。
  `OpenAiAuthStore` 还由
  [BackendFileSystemAuthStore L44–53](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/server/src/commonMain/kotlin/io/github/stream29/kodex/cli/auth/BackendFileSystemAuthStore.kt#L44)
  实现；合 client 不是把所有这些实现收入一个项目。
- `OpenAiModelCatalogStore` 除在线实现外，有
  [session 测试支持 L16–35](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-session/test/src/commonMain/kotlin/io/github/stream29/kodex/agentsession/test/TestKodexAgentDependencies.kt#L16)
  的静态实现；保留组合和注入，不引入 expect/actual。
- RPC 的 8 个 `@Rpc` 具体服务为 Global、AgentRuntime、Index、Work、Settings、Timestamp、TokenCount、Unstable；
  普通 `TimelineRpc<T>` 及 `SuspendMutableStateFlow<T>` 不是要自动生成的新服务。
  [BackendServices L102–117](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/server/src/commonMain/kotlin/io/github/stream29/kodex/rpc/server/BackendServices.kt#L102)
  注册真实 backend；
  [RpcServices L16–23](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/rpc/src/commonMain/kotlin/io/github/stream29/kodex/cli/rpc/RpcServices.kt#L16)
  获取代理；`SessionTitleGenerator` 则明确是
  [backend-local port L7–14](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/spec/contract/src/commonMain/kotlin/io/github/stream29/kodex/cli/sessiontitle/SessionTitleGenerator.kt#L7)。
  个别契约旧 KDoc 仍写“没有实现”，不能据此覆盖上述代码证据。
- MCP `McpClient` 同时有
  [正常与认证阻断实现 L51–82](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/impl/composition/src/commonMain/kotlin/io/github/stream29/kodex/mcp/impl/McpClientImpl.kt#L51)；
  manager/service/OAuth factory 是不同 port，并有 agent-state/test 的测试替身。
  stdio 是 [启动真实 process 的实现 L36–71](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/spec/stdio/src/commonMain/kotlin/io/github/stream29/kodex/mcp/stdio/McpStdioTransport.kt#L36)，
  HTTP 是 [client/SSE 与 401 重试实现 L28–86](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/spec/streamable-http/src/commonMain/kotlin/io/github/stream29/kodex/mcp/streamablehttp/McpStreamableHttp.kt#L28)。
  这两项不能因在 spec 目录就全部认定为纯接口。

### 4. 依赖折叠排雷

#### F1 — 确定阻断：Hook notification 的主图二元环（D）

- 现有 `hook-impl-notification -> rpc-spec-models`：
  [hook build L5–8](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/hook/impl/notification/build.gradle.kts#L5)；
  `rpc-spec-models -> hook-spec-notification`：
  [models build L9–19](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/spec/models/build.gradle.kts#L9)。
- 合并 hook 两项目后必得 `hook-notification -> rpc-models -> hook-notification`，
  即使 `rpc-models` 原封不动保留也成环。
  `CliFrontendSettings.hooks` 是真实属性：
  [模型 L17–26](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/spec/models/src/commonMain/kotlin/io/github/stream29/kodex/rpc/models/CliFrontendSettings.kt#L17)；
  executor 真正消费 `CliFrontendSettings` 和 `Notification`：
  [实现 L44–75](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/hook/impl/notification/src/commonMain/kotlin/io/github/stream29/kodex/cli/notification/NotificationHooks.kt#L44)。
- 可选：保留独立配置模型项目；或把 `NotificationHook` 归入独立 rpc-models，再使 executor 单向消费；
  或另设计前端配置模型边界。后两项是所有权决定，不是换 `implementation`、移动 sourceSet 就能解环。
  不应通过让 specMain 依赖 commonMain，或复制/别名 DTO 来掩盖环。

#### F2 — 确定阻断：MCP 半合并与目标差异（D/B）

- `composition -> stdio/streamable-http`：
  [composition L9–13](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/impl/composition/build.gradle.kts#L9)；
  每个 transport 又 `-> contract`：
  [stdio L8–12](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/spec/stdio/build.gradle.kts#L8)、
  [http L8–13](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/spec/streamable-http/build.gradle.kts#L8)。
  只合 contract+composition 必得两个主图环：`composition -> transport -> composition`。
- 合四项目可消掉域内环，但 contract/stdio/http 当前用 Host，composition 用 CLI。
  [Host L8–21](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/build-logic/src/main/kotlin/kodex.kmp-host.gradle.kts#L8)
  比 [CLI L8–11](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/build-logic/src/main/kotlin/kodex.kmp-cli.gradle.kts#L8)
  多 JS/Node；共同目标见 [Host helper L22–33](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/build-logic/src/main/kotlin/KodexHostKmp.kt#L22)：
  JVM、linuxX64、linuxArm64、macosArm64、mingwX64。
- 选 CLI 宿主会删掉原契约/传输的 JS 覆盖，违反本轮保留 targets 约束；
  选 Host 则把 composition 的 CLI-only 依赖带到 JS 解析，不能静态声称支持。
  中间层可以保留平台专有实现，但所选 `commonMain(impl)` 会参与全部下游目标；
  要把部分实现移出 commonMain 或重划模块，必须作为额外设计，不是此次机械迁移。
- 默认保留四项目是可行规划边界，不是宣称它们必须永远独立。
  同时保留 stdio 的 Node 测试和 HTTP 的 linux/平台 engine 层；不为合项目减少目标。

#### F3 — 主图与测试图必须分开（M/B）

- OpenAI 五组局部折叠后，按当前声明的 **Main→Main** 边未发现域内环。
  `client.commonTest -> client-test.commonMain -> client.commonMain` 是分层测试消费，
  不能仅按无源集的项目图误报为确定编译环；需要后续任务图验证。
  [client L21–25](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/impl/client/build.gradle.kts#L21)、
  [test-support L8–12](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/impl/client-test/build.gradle.kts#L8)。
- 同理，`rpc-server.commonTest -> app-impl-rpc.main -> rpc-client.main`、
  `mcp-stdio.jvmTest -> mcp-composition.main`、`mcp-composition.commonTest -> agent-state/...`
  不等于 Main→Main 环。若将测试依赖错误提升到 commonMain，会真的引入反馈边。
- `rpc-contract` 选合 client 后，原 server main→contract 应改为 server→rpc-client，
  server 测试里的 client 也去重；不能移入 client 的 commonMain。
  contract 选合 server 或再合 models 时，须重投影全部 app/settings/session 边，不在此直接批准。

#### F4 — 跨域重点反馈边（D/B）

- RPC contract 导出 frontend component 契约（authentication/account-usage/session-catalog）；
  RPC models 导出 frontend preferences/new-session/title、agent-context/storage、MCP、Hook、shell。
  这不是单向“后端底层协议”：
  [contract L12–22](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/spec/contract/build.gradle.kts#L12)、
  [models L9–19](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/spec/models/build.gradle.kts#L9)。
- 例如再选择 **rpc models 与 rpc server 同项目**，会出现确定的跨域折叠环：
  `rpc(server+models) -> app-settings-spec-persistence -> rpc(server+models)`。
  现有 server→persistence 见
  [server L15–16](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/server/build.gradle.kts#L15)，
  persistence→models 见
  [persistence L6](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/settings/spec/persistence/build.gradle.kts#L6)。
  不要求对侧先合并就已触发；因此“整个 rpc 目录合一个项目”不是安全默认项。
- MCP contract→tool-contract→storage-clean-models/OpenAI-models；
  runtime/session/state 又消费 MCP：
  [MCP L10–11](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/spec/contract/build.gradle.kts#L10)、
  [tool L10–11](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/spec/contract/build.gradle.kts#L10)、
  [runtime L8–29](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-runtime/impl/composition/build.gradle.kts#L8)。
  合 OpenAI models 与 client、合 tool-contract 与多个高层 tools 会扩大这些底层入口的依赖闭包；
  不能凭“允许依赖实现”推论没有图环，也不能凭有实现依赖就认定非法。
- 本报告只核验明确候选和这几条反馈路径；最终跨五个审查域的 SCC/分源集任务图需协调者依据最终映射汇总，
  不宣称全仓已无环。

### 5. 编译插件、substitution 与 sourceSet 输入

#### F5 — kRPC patch 必须独立保留 ABI 身份（M/B）

- [substitution convention L1–8](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/build-logic/src/main/kotlin/kodex.krpc-utils-patch.gradle.kts#L1)
  对应用它的项目所有 configurations 把 `org.jetbrains.kotlinx:kotlinx-rpc-utils`
  替换为 `:rpc-impl-krpc-utils-patch`。它不是只作用于 rpc 目录，也不是全局自动开启。
- [patch build L6–17](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/krpc-utils-patch/build.gradle.kts#L6)
  明确设置 group、Native `moduleName = "org.jetbrains.kotlinx:utils"`、atomicfu 依赖。
  即使统一目录也必须改 convention 引用而保留 unique name；更不能把源码并入业务 rpc 的 KLIB。
- [map L52–61](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/krpc-utils-patch/src/commonMain/kotlin/kotlinx/rpc/internal/utils/map/SynchronizedHashMap.kt#L52)
  的 entries/keys/values 是锁内快照；[README L11–20](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/krpc-utils-patch/README.md#L11)
  说明替换目的和临时性质。保留 license、expect/actual 的 jvmMain/nativeMain 接线。
- 当前应用 patch 的全部项目：rpc contract/client/in-memory/server、hook notification impl、
  app/cli、app/impl/{application,rpc,session,view}、app/test-support/rpc、
  app/component/agent/impl/view、app/component/new-session/impl/viewmodel、
  app/component/session-sidebar/impl/view、app/component/settings/impl/{view,viewmodel}、
  integration-test。直接声明见对应构建文件第 1–2 行及上述 convention。
  这些消费方更名/合并后需继续生效；仅 rpc 项目保留插件不足以覆盖最终 app/test 解析。
- **B**：核对各目标实际 dependencyInsight/解析结果只含替换后的 utils，
  不出现原版与 patch 双份 ABI、Native unique name 漂移或未来 JS 目标要求不存在的 patch variant。
  本轮没有执行这些命令。

#### F6 — serialization 与 kRPC 不是移动目录即可保证（M/B）

- 基线版本从
  [libs L2–9、L23](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/gradle/libs.versions.toml#L2)
  读取：Kotlin 2.4.0、serialization 1.11.0、kRPC 0.10.3、MCP SDK 为本仓锁定 fork；
  不在迁移中顺便升级或换上游 SDK。
- serialization 插件现用于 OpenAI account-usage/models/json-codec/codex-cli-storage spec、
  client/codex-cli-storage impl、RPC contract/models/client/in-memory/server、MCP contract、Hook spec。
  account-usage impl、MCP composition、Hook impl 不能仅保留原 impl 插件集合后丢弃 spec 插件。
- 8 个 RPC 服务源码移动到 specMain 后，保留 contract 的 kRPC 插件；
  client/in-memory 自身含测试 `@Rpc` 服务，不能因主契约搬走就删除它们的插件。
  server 当前没有 kRPC 编译插件；保留 contract 与 server 分离时不能无故要求 server 添加，
  真合并 contract 时则必须带入。
- [TimelineRpc L32–34](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/spec/contract/src/commonMain/kotlin/io/github/stream29/kodex/rpc/contract/TimelineRpc.kt#L32)
  要求各具体 RPC 接口显式覆写泛型方法供生成器处理。
  保持注解、具体 override、方法顺序/名称/签名及 package；不能顺手消重泛型声明。
- [OpenAiJson L20–24](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/spec/models/src/commonMain/kotlin/io/github/stream29/kodex/openai/OpenAiJson.kt#L20)
  调用生成 serializer；
  [metadata serializer L13–14](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/spec/models/src/commonMain/kotlin/io/github/stream29/kodex/openai/CodexResponsesMetadataSerializer.kt#L13)
  调用 `generatedSerializer()`。这是新源集必须参与插件处理的具体编译输入，不是只需 visibility 的普通 Kotlin 类。
- **B**：证明插件能处理 specMain 的 metadata/JVM/各 Native/JS 输入，生成服务与 serializer 不缺失/不重复；
  main 与 test 编译的插件仍生效；consumer 能调用 serializer/withService/registerService。
  不能从 `commonMain.dependsOn(specMain)` 可抑制 ERROR 外推出这些结论。
- convention 的 `explicitApi`、JDK25、warnings-as-errors 来自
  [helper L13–19](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/build-logic/src/main/kotlin/KodexHostKmp.kt#L13)；
  test-balloon/Kotlin test 注入到
  [commonTest L50–55](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/build-logic/src/main/kotlin/KodexHostKmp.kt#L50)。
  若新 convention 只配置 commonMain，specMain 的必要语言/opt-in/库输入必须单独核对，不能期待反向继承。
- HTTP engine 现分布于
  [jvmMain/linuxMain/macosArm64Main/mingwX64Main L15–25](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/spec/streamable-http/build.gradle.kts#L15)。
  保留 Native/linux 层次，不改放到 specMain；本域没有业务 cinterop，但下游 process/shell 仍有平台实现。
  构建脚本没有声明 JS HTTP engine 是基线事实，本轮不推断其当前运行成败，也不借迁移默许缺失目标。

### 6. Wire / 持久化兼容检查

| 兼容面与观察证据 | 机械迁移不应改变的内容；后续证明 |
| --- | --- |
| [OpenAI codec L5–16](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/spec/json-codec/src/commonMain/kotlin/io/github/stream29/kodex/openai/jsoncodec/OpenAiJsonCodec.kt#L5) | 同一个 policy：忽略未知 key、省略 explicit null、编码默认值。client 在 [L130–132](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/impl/client/src/commonMain/kotlin/io/github/stream29/kodex/openai/client/OpenAiClient.kt#L130) 使用它；不能各合并项目各写一个不同 Json。 |
| [ResponseItem/stream serializer L20–24、L89–116](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/spec/models/src/commonMain/kotlin/io/github/stream29/kodex/openai/OpenAiJson.kt#L89)；[metadata L15–44](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/spec/models/src/commonMain/kotlin/io/github/stream29/kodex/openai/CodexResponsesMetadataSerializer.kt#L15) | 已知/未知 tagged payload、Other 原样保留、tool-search 私有/外部 tag 转换、metadata flattening/Memory 去身份字段必须保持；serializer 及依赖模型一起迁，不从 specMain 引用 commonMain serializer。 |
| [ReasoningEffort L169–190](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/spec/models/src/commonMain/kotlin/io/github/stream29/kodex/openai/OpenAiJson.kt#L169) | `ultra -> Max` 兼容解析、Custom 未知值保留；勿顺手改枚举或 legacy spelling。 |
| [CodexCliStorageModels L91–145](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/spec/codex-cli-storage/src/commonMain/kotlin/io/github/stream29/kodex/openai/codexclistorage/CodexCliStorageModels.kt#L91) | 无 tag TOML 以 command 选择 subtype；`OPENAI_API_KEY`、`auth_mode`、`last_refresh` 等键及缺失默认值保留。不是迁到 MCP settings 的同形模型。 |
| [CodexCliStorageImpl L21–39、L147–165](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/impl/codex-cli-storage/src/commonMain/kotlin/io/github/stream29/kodex/openai/codexclistorage/CodexCliStorageImpl.kt#L21)；[backend auth 写入 L205–245](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/server/src/commonMain/kotlin/io/github/stream29/kodex/cli/auth/BackendFileSystemAuthStore.kt#L205) | `auth.json`/`config.toml` 只读视图与 backend auth 原子写回是不同所有者；目录迁移不得改变真实 Home 路径、Codex metadata 保留、私有文件写入及导入不支持字段的脱敏规则。 |
| [MCP settings L21–26、L55–91、L121–154](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/spec/contract/src/commonMain/kotlin/io/github/stream29/kodex/mcp/contract/McpSettings.kt#L21) | `McpSecret` 仍字符串序列化/诊断脱敏；`streamable_http`/`stdio`、OAuth tags、working_directory 的 Path serializer、默认值不变。部分 sealed 子类无显式 SerialName（如 McpClientState），更必须保持 FQCN，不能因移动项目改包导致默认 discriminator 改变。 |
| [Hook model L6–43](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/hook/spec/notification/src/commonMain/kotlin/io/github/stream29/kodex/rpc/models/NotificationHook.kt#L6)；[executor L61–75](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/hook/impl/notification/src/commonMain/kotlin/io/github/stream29/kodex/cli/notification/NotificationHooks.kt#L61) | 保留旧 package `rpc.models` 和四个 stop tag、顺序/唯一名规则；backend notification 用默认 Json，frontend unhandled_error 手工 payload，不改成 OpenAI codec，也不让 Hook 执行回流控制 Agent。 |
| [前端模型 L6–26](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/spec/models/src/commonMain/kotlin/io/github/stream29/kodex/rpc/models/CliFrontendSettings.kt#L6)；[SplitSettingsFiles L81–115](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/settings/impl/filesystem/src/commonMain/kotlin/io/github/stream29/kodex/cli/settings/SplitSettingsFiles.kt#L81) | RPC models 同时服务 `settings.backend.yml` 与 `settings.frontend.cli.yml`；包名/RPC 目录不能决定持久化归属。保留 schema/default/migration 语义；跨域 app/settings 测试由协调者纳入，不在此修改。 |
| [RPC 泛型契约 L32–44](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/spec/contract/src/commonMain/kotlin/io/github/stream29/kodex/rpc/contract/TimelineRpc.kt#L32)；[注册 L108–116](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/server/src/commonMain/kotlin/io/github/stream29/kodex/rpc/server/BackendServices.kt#L108) | 生成服务身份、完整方法签名、流/取消/异常语义不变；不要把持久化对象或 service instance 直接跨 transport 传递来“修复测试”。 |

这些均是已有兼容约束，不是本轮发现了 wire 已变化。目录/Gradle ID 本身不是 wire 字段；
是否保持生成 descriptor、metadata 和实际 round-trip 仍属 **B**。

### 7. 测试与资源迁移检查（未运行）

- 本域跟踪的 `src/` 文件全为 `.kt`，没有独立 `resources/` 文件要搬；
  **不等于没有 fixture 或 I/O 资源**。测试以源码内 JSON/TOML/YAML、动态文件、HTTP server、子进程和端口构造 fixture。
- 13 spec 的测试分别为：OpenAI models **14** 个、RPC contract **14** 个、RPC models **7** 个、
  MCP contract **2** 个、stdio common **1**/JVM **2**/JS **1**、HTTP **1**、Hook **1**；
  其余 spec 无自有测试。实现测试规模见第 3 节；所有旧 Test 源集均须保留。
- OpenAI：保留 model 序列化 suites（含 `CodexResponsesMetadataSerializationTest`、
  `ResponseItemSerializationTest`、`ExplicitReasoningSerializationTest`、`ServiceTierSerializationTest`）；
  client 的 `OpenAiSubscriptionSerializationTest`、`OpenAiResponseResultTest`、
  retry/login/compaction tests 及 3 个 JVM HTTP tests；catalog/account snapshot/storage/mock suites。
  [models test 依赖 L16–18](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/spec/models/build.gradle.kts#L16)
  的 codec 保持测试作用域；[storage test L18 起](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/impl/codex-cli-storage/src/commonTest/kotlin/io/github/stream29/kodex/openai/codexclistorage/CodexCliStorageTest.kt#L18)
  会创建临时 Home，搬迁不能依赖原项目 cwd。
- RPC：保留 contract 的 12 个 values suites + `InMemoryRpcBoundaryTest`/`InMemoryRpcFixture`；
  models 的 7 个 suites；client 缓存/Flow/异常恢复、in-memory transport、server 生命周期/设置/OAuth/标题相关 suites。
  [contract fixture L33–42、L106–143](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/spec/contract/src/commonTest/kotlin/io/github/stream29/kodex/rpc/contract/InMemoryRpcFixture.kt#L33)
  有测试专用 `@Rpc`、自己的 JSON transport；
  与 [生产 in-memory harness L25–49](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/in-memory/src/commonMain/kotlin/io/github/stream29/kodex/rpc/inmemory/WithInMemoryRpc.kt#L25)
  分开，不能用直接调用替代原 wire boundary。
- MCP：stdio 的
  [JVM test L93–110](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/spec/stdio/src/jvmTest/kotlin/io/github/stream29/kodex/mcp/stdio/McpStdioTransportIoTest.kt#L93)
  由 fixture class `codeSource` 与 Kotlin runtime 构造子进程 classpath，不是硬编码目录；
  仍要保证 `McpStdioServerFixture` 随 JVM test 搬迁，不进入产品 jar 或 specMain。
  [JS test L21–34](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/spec/stdio/src/jsTest/kotlin/io/github/stream29/kodex/mcp/stdio/McpStdioTransportIoTest.js.kt#L21)
  用 Node `process.execPath`/`-e` 内嵌脚本；
  composition 的 JVM OAuth/service/owner-publication tests 用真实本地 HTTP。
  这些都要求实际平台执行后才有验收价值，本轮没有占用端口或启动子进程。
- Hook：保留 model 验证、executor 假进程生命周期、RPC wire 到通知命令 stdin 的 suites。
  合并模型不得把这些测试主依赖提前引入 specMain。
- 合并测试时，除了文件名冲突还应检测同包顶层声明/Test suite ID。
  当前保守候选未见需要删除测试的理由；**B** 才能证明重新发现的 suite 数、平台 test task、资源释放和结果相同。

### 8. 域外直接消费者完整矩阵

此表覆盖所有跟踪 Gradle 脚本里显式指向本域 24 个项目的**域外**依赖；
域内消费见第 2–3 节各 build 链接，patch convention 见 F5。
不把传递消费伪称直接依赖：例如 app frontend 的 RPC 服务由 `rpc-impl-client` 导出取得。
矩阵是静态声明，不是 Gradle 已解析的 variant 图。

缩写：`O.x` = `:openai-spec-x`，`Oi.x` = `:openai-impl-x`；
`R.x`/`Ri.x` = rpc spec/impl；`M.x`/`Mi.x` = mcp spec/impl；
`H.x`/`Hi.x` = hook spec/impl。未标源集均 commonMain；`T`=commonTest、`JT`=jvmTest、
`MT`=mosaicTest、`MM`=mosaicMain。链接指向本行构建声明起点，括号 L 为同文件具体行。

| 消费项目（源码证据） | 本域直接依赖 |
| --- | --- |
| [agent-context/impl/prefix-render](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-context/impl/prefix-render/build.gradle.kts#L11) | O.models |
| [agent-context/impl/prefix](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-context/impl/prefix/build.gradle.kts#L11) | O.models |
| [agent-runtime/impl/composition](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-runtime/impl/composition/build.gradle.kts#L16) | M.contract(L16)、O.client(L17)、O.model-catalog(L18)、O.models(L19) |
| [agent-runtime/impl/decorator/compact](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-runtime/impl/decorator/compact/build.gradle.kts#L12) | O.model-catalog；T: Oi.client-test(L23)、Oi.model-catalog(L24) |
| [agent-runtime/impl/decorator/tool](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-runtime/impl/decorator/tool/build.gradle.kts#L21) | T: M.contract(L21)、Oi.client-test(L22)、O.json-codec(L23) |
| [agent-session/impl/filesystem](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-session/impl/filesystem/build.gradle.kts#L21) | T: Oi.client-test |
| [agent-session/impl/in-memory](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-session/impl/in-memory/build.gradle.kts#L19) | T: Oi.client-test(L19)、O.json-codec(L20) |
| [agent-session/spec/contract](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-session/spec/contract/build.gradle.kts#L11) | M.contract(L11)、O.client(L12)、O.model-catalog(L13) |
| [agent-session/test](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-session/test/build.gradle.kts#L10) | Oi.client-test(L10)、O.model-catalog(L11)、O.models(L12) |
| [agent-state/impl/state](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-state/impl/state/build.gradle.kts#L12) | M.contract(L12)、O.client(L13)、O.json-codec(L29)；T: Oi.client-test(L36) |
| [agent-state/spec/context-window](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-state/spec/context-window/build.gradle.kts#L10) | O.models(L10)、O.model-catalog(L11)；T: Oi.client-test(L18)、Oi.model-catalog(L19) |
| [agent-state/spec/contract](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-state/spec/contract/build.gradle.kts#L11) | O.models |
| [agent-state/test](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-state/test/build.gradle.kts#L9) | M.contract |
| [agent-storage/impl/filesystem](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-storage/impl/filesystem/build.gradle.kts#L27) | O.json-codec |
| [agent-storage/spec/clean-models](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-storage/spec/clean-models/build.gradle.kts#L12) | O.models(L12)；T: O.json-codec(L22) |
| [agent-storage/spec/contract](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-storage/spec/contract/build.gradle.kts#L11) | O.models |
| [app/component/account-usage/spec](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/account-usage/spec/build.gradle.kts#L9) | O.account-usage |
| [app/component/agent/impl/view](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/agent/impl/view/build.gradle.kts#L31) | MT: Ri.server(L31)、Ri.in-memory(L32) |
| [app/component/agent/spec](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/agent/spec/build.gradle.kts#L15) | O.models(L15)、R.models(L16) |
| [app/component/authentication-settings/spec](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/authentication-settings/spec/build.gradle.kts#L10) | O.models |
| [app/component/composer/spec](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/composer/spec/build.gradle.kts#L7) | O.models |
| [app/component/history/impl/viewmodel](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/impl/viewmodel/build.gradle.kts#L19) | T: O.json-codec |
| [app/component/hook-settings/spec](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/hook-settings/spec/build.gradle.kts#L6) | H.notification |
| [app/component/mcp-settings/spec](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/mcp-settings/spec/build.gradle.kts#L6) | M.contract |
| [app/component/new-session-defaults/spec](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/new-session-defaults/spec/build.gradle.kts#L11) | O.models |
| [app/component/new-session/impl/viewmodel](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/new-session/impl/viewmodel/build.gradle.kts#L13) | O.models(L13)；T: Oi.client-test(L23) |
| [app/component/runtime-configuration/spec](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/runtime-configuration/spec/build.gradle.kts#L6) | O.models |
| [app/component/session-settings/spec](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-settings/spec/build.gradle.kts#L6) | O.models |
| [app/component/session-title-settings/spec](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/session-title-settings/spec/build.gradle.kts#L11) | O.models |
| [app/component/settings/spec](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/settings/spec/build.gradle.kts#L22) | R.models(L22)、M.contract(L23)、O.account-usage(L24)、O.models(L25) |
| [app/component/suggest-subagent-task/spec](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/suggest-subagent-task/spec/build.gradle.kts#L10) | O.models |
| [app/component/usage-reset/spec](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/usage-reset/spec/build.gradle.kts#L7) | O.account-usage |
| [app/impl/application](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/application/build.gradle.kts#L16) | Ri.server(L16)、Ri.in-memory(L17)、Hi.notification(L19)、M.contract(L28)、O.account-usage(L29)、O.models(L30)、Oi.client(L34)；T: Oi.client-test(L50) |
| [app/impl/rpc](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/rpc/build.gradle.kts#L29) | Ri.client(L29)；T: Ri.server(L45)、Ri.in-memory(L46)、Oi.client-test(L47) |
| [app/impl/session](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/session/build.gradle.kts#L17) | T: Ri.server(L17)、Ri.in-memory(L18)、Oi.client-test(L27) |
| [app/impl/view](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/view/build.gradle.kts#L51) | MM: O.models(L51)；MT: Ri.server(L63)、Ri.in-memory(L64) |
| [app/settings/spec/persistence](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/settings/spec/persistence/build.gradle.kts#L6) | R.models |
| [app/spec/session](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/spec/session/build.gradle.kts#L9) | O.models |
| [app/test-support/rpc](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/test-support/rpc/build.gradle.kts#L12) | Oi.client-test(L12)、Ri.server(L13)、Ri.in-memory(L14) |
| [integration-test](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/integration-test/build.gradle.kts#L29) | T: M.contract(L29)、Oi.client(L30)、O.client(L31)、Oi.client-test(L32)、Oi.codex-cli-storage(L33)、O.json-codec(L34)、Oi.model-catalog(L35)；JT: Ri.client(L65)、Ri.in-memory(L66)、Ri.server(L68)、Mi.composition(L70) |
| [tool/impl/get-context-remaining](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/get-context-remaining/build.gradle.kts#L11) | O.models(L11)、O.model-catalog(L12)；T: Oi.client-test(L23)、O.json-codec(L24)、Oi.model-catalog(L25) |
| [tool/impl/image-generation](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/image-generation/build.gradle.kts#L10) | O.client(L10)、O.models(L11)；T: Oi.client-test(L21)、O.json-codec(L22) |
| [tool/impl/multi-agent](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/multi-agent/build.gradle.kts#L10) | O.models(L10)；T: O.json-codec(L14) |
| [tool/impl/plan](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/plan/build.gradle.kts#L11) | O.models(L11)；T: Oi.client-test(L21)、O.json-codec(L22) |
| [tool/impl/request-user-input](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/request-user-input/build.gradle.kts#L10) | O.models(L10)；T: O.json-codec(L14) |
| [tool/impl/tool-search](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/tool-search/build.gradle.kts#L9) | O.models |
| [tool/impl/web-run](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/web-run/build.gradle.kts#L10) | O.client(L10)、O.models(L11)；T: Oi.client-test(L17)、O.json-codec(L18) |
| [tool/spec/builder](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/spec/builder/build.gradle.kts#L9) | O.models |
| [tool/spec/contract](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/spec/contract/build.gradle.kts#L11) | O.models |
| [tool/spec/current-time](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/spec/current-time/build.gradle.kts#L8) | O.models |
| [tool/spec/get-context-remaining](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/spec/get-context-remaining/build.gradle.kts#L8) | O.models |
| [tool/spec/plan](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/spec/plan/build.gradle.kts#L8) | O.models |
| [tool/spec/tool-search](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/spec/tool-search/build.gradle.kts#L9) | O.models |

#### 8.1 通过导出依赖消费源码的补充清单

不能把上面的 53 个显式依赖项目当作全部源码消费方。另对跟踪 `.kt` 的 import 做了文本扫描：
匹配本域 `openai.*`、`rpc.*`、`mcp.*`、`cli.notification.*`，
发现以下额外 **42 个项目**。它们在 Gradle 中没有直接依赖本域项目，主要经自己的 spec/共享模型导出而消费。
这是 import 清单，不是符号解析证明；不涵盖不写 import 的全限定名或动态/生成代码。
没有把 `cli.auth.KodexAuthLoginAttempt`、`cli.sessiontitle.DefaultSessionTitleModel` 算作本域：
它们虽和 RPC server 代码共享包前缀，实际声明在 app component spec。

下表按目录前缀列全这 42 项，证据链接为该组的代表性 import；
迁移验收不得只编译 53 个直接依赖项目而忽略这些组件、持久化和迁移测试。

| 额外消费项目（`{a,b}` 表示两项） | 源码证据/关注点 |
| --- | --- |
| `agent-runtime/impl/steer`、`agent-runtime/spec/contract`、`agent-runtime/spec/decorator/compact` | [AgentRuntime L4–6](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-runtime/spec/contract/src/commonMain/kotlin/io/github/stream29/kodex/agentruntime/contract/AgentRuntime.kt#L4)：Compaction 模型由 storage 导出；steer 测试也有 OpenAI import。 |
| `agent-storage/impl/in-memory`、`agent-storage/spec/contract-ext` | [in-memory L6](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-storage/impl/in-memory/src/commonMain/kotlin/io/github/stream29/kodex/agentstorage/inmemory/InMemoryKodexAgentStorage.kt#L6)：模型并非只用于网络 client。 |
| `app/component/account-usage/impl/{view,viewmodel}`、`app/component/authentication-settings/impl/{view,viewmodel}` | [account usage view L19](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/account-usage/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/accountusage/AccountUsageComponent.kt#L19)：UI 与测试经各自 spec 使用 OpenAI 值。 |
| `app/component/composer/impl/{view,viewmodel}`、`app/component/history-index/impl/viewmodel`、`app/component/history/impl/view`、`app/component/history/spec` | [composer view L33](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/composer/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/agent/ComposerView.kt#L33)：历史/输入模型及序列化值。 |
| `app/component/hook-settings/impl/{view,viewmodel}` | [viewmodel L3](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/hook-settings/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/app/hooksettings/HookSettingsViewModel.kt#L3)：`NotificationHook` 的真实 consumer。 |
| `app/component/mcp-settings/impl/{view,viewmodel}` | [McpEditor L3](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/mcp-settings/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/app/mcpsettings/McpEditor.kt#L3)：MCP draft/configuration helper 必须仍可消费。 |
| `app/component/new-session-defaults/impl/{view,viewmodel}`、`app/component/new-session/impl/view`、`app/component/new-session/spec` | [defaults viewmodel L5](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/new-session-defaults/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/app/settings/NewSessionSettingsViewModel.kt#L5)：模型与 provider 相关值。 |
| `app/component/request-user-input/impl/viewmodel` | [request-user-input test L14](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/request-user-input/impl/viewmodel/src/commonTest/kotlin/io/github/stream29/kodex/cli/agent/RequestUserInputViewModelTest.kt#L14)：测试通过组件依赖消费 OpenAI 值。 |
| `app/component/runtime-configuration/impl/{view,viewmodel}`、`app/component/session-settings/impl/{view,viewmodel}`、`app/component/session-title-settings/impl/{view,viewmodel}` | [runtime configuration L3](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/runtime-configuration/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/app/runtimeconfiguration/RuntimeConfigurationViewModel.kt#L3)：保留模型的传递导出。 |
| `app/component/settings/impl/{view,viewmodel}`、`app/component/suggest-subagent-task/impl/{view,viewmodel}`、`app/component/usage-reset/impl/{view,viewmodel}` | [settings test L8](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/settings/impl/viewmodel/src/commonTest/kotlin/io/github/stream29/kodex/app/settings/WorkingDirectoryOwnershipTest.kt#L8)：不能只验 spec 的编译，Mosaic 与 commonTest 也消费模型。 |
| `app/migration/impl`、`app/settings/impl/filesystem` | [migration test L10–15](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/migration/impl/src/commonTest/kotlin/io/github/stream29/kodex/app/migration/MigrateToV047CurrentStoresTest.kt#L10)、[store L3](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/settings/impl/filesystem/src/commonMain/kotlin/io/github/stream29/kodex/cli/settings/FileSystemBackendSettingsStore.kt#L3)：wire/持久化回归必须包括这两个消费方。 |
| `tool/impl/{apply-patch,builder,current-time,unified-exec,view-image}` | [builder L10](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/builder/src/commonMain/kotlin/io/github/stream29/kodex/tool/builder/JsonToolBuilder.kt#L10)：通过 tool spec 消费 OpenAI 工具模型。 |

### 9. 移交清单与完成边界

- **M，边界确定后**：优先分批规划 OpenAI 五组；保留真实 package/KDoc/wire、Host 的六目标、
  implementation/sourceSet 原方向与所有测试，修复 settings 注册和所有直接消费 ID。
  五组都减少一个叶项目只是静态计数，不据此预报 Sync/内存/构建性能改善。
- **D，用户决定**：是否采纳五组中各组；model/codec 保留或单独整理；
  RPC contract 与 client 是否同项目；Hook 模型归属；MCP 保留拆分还是重新设计支持相同目标的完整组合。
  不把本子任务候选当已授权实施。
- **B，需另获授权**：最终 main/平台/test 图和 SCC；新源集依赖可见性与精确抑制的正负控制；
  serializer/kRPC 的生成、发布 metadata 与消费者；各原目标编译/测试及 fixture 资源释放；
  独立消费者的 wire/persistence 回归；真实 IDE Sync/导航/索引与失效、性能另测。
- **本轮完成项**：13/13 spec、11/11 impl、全部域外显式 Gradle 消费者、substitution 消费者、
  42 个额外 import 消费项目，已列合并候选与确定成环条件、平台差异、wire 和测试风险。
  没有提供编译/测试/性能通过结论，没有改源码或其他任务文件。
- **静态审查已完成，Task Tree 已走完**。归档到 done 作为父任务审查输入，不表示迁移已执行或获准。
- 结束复核：`Kodex/` HEAD 仍为 `4388871137c56bd1b73d5b739cc98f036a790f3c`，工作区仍干净；
  外层 HEAD 仍为 `c0a7d34c4d8a5a7e69538e68d80b56d5fd0bc671`。
  已复核全文、文件 URI/行号、13/11 项目录计数和 53 项直接消费者覆盖；本轮只写本文件。
  无临时文件、后台任务、端口或设备资源需要清理。

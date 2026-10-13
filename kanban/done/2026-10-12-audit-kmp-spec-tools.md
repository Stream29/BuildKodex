# Task Tree

- `Inventory and review every tool spec project`()
- `Propose consolidation mapping and identify concrete hazards`()
- `Record source evidence and validation requirements`()

# Details

- Parent: [KMP source-set migration plan](2026-10-12-discuss-kmp-spec-impl-source-sets.md#L1).
- Scope: all `tool` projects in `Kodex/`, implementations and their consumers.
- Baseline: `4388871137c56bd1b73d5b739cc98f036a790f3c`; verify HEAD before/after review.
- Read [confirmed design](../../checklist/kmp-spec-impl-source-sets.md).
  Target is platform/intermediate -> commonMain (impl) -> specMain.
- Static audit only. No build, product edits, commits, device/IDE control, or new Sessions.
- Only this report is writable by this review line.
- Deliver a complete per-spec inventory, concrete old->new project/source mapping,
  mechanical moves vs decisions, local and cross-domain edges, and file/line evidence.
- Review DTO/handler dependencies, reverse dependencies from storage/runtime,
  schema/serialization generators, hierarchy/interop, resources and tests, and cycles.
- Distinguish observed blockers, plausible unverified risks, and checks needed.
  Do not claim compilation or performance results from source inspection.

## Review result — 2026-10-12 / Session 777

### 结论与证据边界

- **不能把全部工具按同名 spec/impl 无条件合并。** 仅折叠工具项目，
  `image-generation`、`unified-exec`、`view-image` 就各自经
  `tool-spec-contract -> agent-storage-spec-clean-models` 成环，见 B1。
- `plan`、`get-context-remaining` 单独合并没有新增生产项目环；
  **若同时把 agent-state contract/state 折叠，则确定成环**，见 B2。
  它们不是当前基线已经存在的环。
- 其余候选不是构建已通过：schema、serialization、Native/JS 依赖、
  测试和 IDE/metadata 均有独立检查门槛。合法消费者依赖实现不算缺陷。
- 起始 `git -C Kodex rev-parse HEAD`：
  `4388871137c56bd1b73d5b739cc98f036a790f3c`；`git -C Kodex status --short`
  为空。外层 BuildKodex HEAD 为 `c0a7d34c4d8a5a7e69538e68d80b56d5fd0bc671`，
  外层原有多份改动和未跟踪报告，未将它们误认为 Kodex 基线差异。
- 方法：读全部 tool 构建脚本、契约和实现源码，检索全仓项目引用、Kotlin
  imports、测试与约定；用临时 stdin Python 脚本静态投影项目依赖图，
  不落脚本文件、不调用 Gradle。扫描 203 个非 root/build-logic/fork
  构建脚本、879 个字面 `project(":…")` 出现点，按源集区分生产与 Test。
- 图是**源码推导**，不是 Gradle 解析模型。动态依赖、插件生成任务和
  publication variant 没有执行确认；main SCC 不混入 commonTest 反向测试边。
  图推导结果：原生产图无环；全工具折叠产生 B1 的五节点 SCC；
  留下三组阻塞配对不折叠、只折叠其余九组时，该静态生产图无环。

### 1. 固定的迁移结构

- 遵循 [已确认设计](file:///home/stream/ACodeSpace/push/BuildKodex/checklist/kmp-spec-impl-source-sets.md#L5-L52)：
  `platform/intermediate -> commonMain (impl) -> 真正的 specMain`。
  不另建 `implMain`，也不把另一个目录冒充源集。
- `commonMain.dependsOn(specMain)` 采用精确 internal 诊断抑制：
  `kotlin.internal.suppressGradlePluginErrors=CommonMainOrTestWithDependsOnDiagnostic`。
  不关闭全部诊断；此 ID 也覆盖 commonTest，须另查是否出现多余测试边。
  本报告承接 [KGP 例外及层次约束](file:///home/stream/ACodeSpace/push/BuildKodex/checklist/kmp-spec-impl-source-sets.md#L52-L73)，没有重新做工具链实验。
- 现有 [gradle.properties](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/gradle.properties#L1-L10)
  没有该抑制项。后续应在 convention/属性的最小约定位置接入，不在每个工具
  中复制整套配置。增加上游 specMain 不能消除跨项目依赖环。
- 消费者可以依赖合并产物并使用公开实现。`api` 改成 `implementation`、
  换源集名、保留两个同包目录都不是 B1/B2 的解法。

### 2. 全体 spec 清单及合并路径

- 注册事实：[settings 中 12 impl / 13 spec](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/settings.gradle.kts#L441-L470)；
  **没有 `tool/impl/contract`**。所有 25 个工具项目都使用 `kodex.kmp-host`。
- 以下 `S/I` 是当前 spec/impl 构建脚本；文件名均相对于各自的
  `src/commonMain/kotlin/io/github/stream29/kodex/tool/<包名>/`。
  表中源文件计数为 `spec commonMain / impl commonMain / commonTest总数`，
  unified-exec 另有四个 Main actual 和四个 Test actual 文件。
- 每一行的候选（非执行决定）：
  `:tool-spec-X + :tool-impl-X -> :tool-X`，
  `Kodex/tool/spec/X + Kodex/tool/impl/X -> Kodex/tool/X`。
  完整目录规则：
  - `tool/spec/X/src/commonMain/kotlin/** -> tool/X/src/specMain/kotlin/**`；
  - `tool/impl/X/src/commonMain/kotlin/** -> tool/X/src/commonMain/kotlin/**`；
  - 原 `src/commonTest/**`、平台和中间源集按原名称进入 `tool/X/src/**`；
  - 保留 package、KDoc、序列化名；删原项目边，重定向全部消费者。
    settings 用 [扁平 ID 规则](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/settings.gradle.kts#L59-L85)
    注册新目录，不保留转发项目或复制声明。

| X / 当前项目证据 | spec 全部生产文件 | impl 全部 commonMain 文件 | 计数 / 候选判断 |
| --- | --- | --- | --- |
| apply-patch — [S][s-apply] / [I][i-apply] | ApplyPatchToolSpec.kt | ApplyPatchToolClient.kt、ApplyPatchTools.kt | 1/2/1；`tool/apply-patch`，保留 freeform，不转 JSON builder |
| builder — [S][s-builder] / [I][i-builder] | JsonToolBuilderSpec.kt | JsonToolBuilder.kt | 1/1/1；`tool/builder`，独立共享适配器 |
| contract — [S][s-contract] | Tool.kt、ToolName.kt | 无同名 impl；typedTool 目前也在 Tool.kt | 2/0/1；默认保留 `:tool-spec-contract` / `tool/spec/contract`，不虚造 impl |
| current-time — [S][s-time] / [I][i-time] | CurrentTimeSchemas.kt | CurrentTimeToolClient.kt、CurrentTimeTools.kt | 1/2/1；`tool/current-time` |
| get-context-remaining — [S][s-context] / [I][i-context] | GetContextRemainingSchemas.kt | GetContextRemainingTool.kt、GetContextRemainingTools.kt | 1/2/2；`tool/get-context-remaining`，附 B2 门槛 |
| image-generation — [S][s-image] / [I][i-image] | ImageGenerationToolModels.kt | GeneratedImageArtifacts.kt、ImageGenerationSchemas.kt、ImageGenerationToolClient.kt、ImageGenerationTools.kt | 1/4/3；`tool/image-generation` 仅目标候选，B1 阻塞，暂留原两项目 |
| multi-agent — [S][s-multi] / [I][i-multi] | SuggestSubagentTaskModels.kt | SuggestSubagentTaskSchemas.kt、SuggestSubagentTaskTools.kt | 1/2/1；`tool/multi-agent`，impl 只有 schema，不含 Session 执行器 |
| plan — [S][s-plan] / [I][i-plan] | PlanSchemas.kt | PlanTools.kt、UpdatePlanTool.kt | 1/2/2；`tool/plan`，附 B2 门槛 |
| request-user-input — [S][s-input] / [I][i-input] | RequestUserInputModels.kt | RequestUserInputSchemas.kt、RequestUserInputTools.kt | 1/2/1；`tool/request-user-input`，impl 只有 schema，不含 UI 执行器 |
| tool-search — [S][s-search] / [I][i-search] | ToolSearchModels.kt | ToolSearchConstants.kt、ToolSearchDocumentConversion.kt、ToolSearchEngine.kt、ToolSearchSchemas.kt、ToolSearchTools.kt | 1/5/1；`tool/tool-search`，不要把 runtime adapter 拉入 |
| unified-exec — [S][s-exec] / [I][i-exec] | UnifiedExecClient.kt、UnifiedExecLimits.kt、UnifiedExecModels.kt | ExecCommandShellDescription.kt、UnifiedExecSchemas.kt、UnifiedExecToolClient.kt、UnifiedExecTools.kt | 3/4/2 + 4 Main/4 Test actual；`tool/unified-exec` 仅候选，B1 阻塞 |
| view-image — [S][s-view] / [I][i-view] | ViewImageModels.kt | ViewImageSchemas.kt、ViewImageToolClient.kt、ViewImageTools.kt | 1/3/2；`tool/view-image` 仅候选，B1 阻塞 |
| web-run — [S][s-web] / [I][i-web] | WebRunSchemas.kt | WebRunToolClient.kt、WebRunTools.kt | 1/2/2；`tool/web-run`，SearchCommands 等 DTO 仍由 OpenAI 拥有 |

合计 16 个 spec 生产 Kotlin 文件、31 个 impl commonMain 文件、4 个平台
actual 文件、20 个 commonTest 文件、4 个 Test actual 文件；本地 tool 树未见
独立资源文件、nativeInterop 或自定义生成脚本。计数是文件覆盖检查，不是编译量。

- **可机械处理（在边界获准之后）**：保持包名的目录搬移、原 S/I 依赖按层迁移、
  插件去重保留、settings/消费者 project ID 重写、测试目录和任务地址重定向。
- **不可机械处理**：B1/B2 的分组与 DTO/handler 所有权、混合文件的规范声明提取、
  平台相关 schema 的参数化。KGP 编译/metadata/IDE 和传递 Native 依赖属于
  **未验证风险**，不能写成已失败或已通过。

#### 2.1 specMain 所需依赖与插件（不可只移动源码）

| spec | 迁移时须随契约上移的当前声明 | impl 仍保留的关键依赖 |
| --- | --- | --- |
| apply-patch | `tool-spec-contract`（当前 api，源码本身是 grammar/description）[S][s-apply] | contract、kotlinx-io-coroutines impl、patch impl [I][i-apply] |
| builder | OpenAI models、contract、serialization core；serialization plugin [S][s-builder] | serialization JSON [I][i-builder] |
| contract | storage clean-models、OpenAI models；serialization plugin、tests [S][s-contract] | 无配对 |
| current-time | OpenAI models、contract、schema JSON [S][s-time] | datetime、builder impl、serialization core [I][i-time] |
| get-context-remaining | OpenAI models、contract、schema JSON [S][s-context] | state contract/context-window、model-catalog、builder impl [I][i-context] |
| image-generation | serialization core + plugin [S][s-image] | OpenAI client/models、contract、images spec、images-codec impl、IO impl、schema [I][i-image] |
| multi-agent | serialization core + plugin [S][s-multi] | OpenAI models、schema [I][i-multi] |
| plan | OpenAI models、contract、schema [S][s-plan] | state contract、builder impl、serialization JSON [I][i-plan] |
| request-user-input | serialization core + plugin [S][s-input] | OpenAI models、schema [I][i-input] |
| tool-search | OpenAI models、serialization core + plugin [S][s-search] | search-index impl、coroutines、schema [I][i-search] |
| unified-exec | shell-client spec、coroutines、serialization core + plugin [S][s-exec] | contract、shell-client impl、schema、JS node wrapper [I][i-exec] |
| view-image | serialization core + plugin [S][s-view] | contract、images spec、images-codec impl、IO impl、schema [I][i-view] |
| web-run | schema JSON [S][s-web] | OpenAI client/models、contract [I][i-web] |

这是保留当前依赖语义的清单，不是要求无条件清理当前未使用 api。特别是只将
impl 脚本作为新脚本会丢掉 image-generation/multi-agent/request-user-input/
tool-search/unified-exec/view-image 原 spec 上的 serialization plugin。
依赖应在 specMain 重建，不能指望 commonMain 逆向向 specMain 提供类库。

### 3. 已证实的静态阻塞与条件性阻塞

#### B1 — storage DTO 回路，三组工具不能机械折叠

令 `C = :agent-storage-spec-clean-models`、`T = :tool-spec-contract`。
旧图合法：`T -> C -> spec(X)`，`impl(X) -> T`、`impl(X) -> spec(X)`。
折叠 spec(X)/impl(X) 后变成 `C -> X -> T -> C`。

| X | C -> spec(X) | impl(X) -> T | T -> C |
| --- | --- | --- | --- |
| image-generation | [clean-models:13][storage-build] | [impl:12][i-image] | [contract:10][s-contract] |
| unified-exec | [clean-models:17][storage-build] | [impl:24][i-exec] | 同左 |
| view-image | [clean-models:18][storage-build] | [impl:10][i-view] | 同左 |

- 不是依赖声明多余造成的假阳性。`Tool.handle` 直接以
  `PendingToolEvent -> StableCleanEvent.CompletedTool` 为公共签名
  [Tool.kt:3-23][tool-contract-code]；image/view/exec 实现都构造 storage 事件：
  [image handler:58-100][image-handler]、
  [view handler:27-50][view-handler]、
  [exec handler:41-105][exec-handler]。
- storage 反向边也真实：例如
  [StableImageGenerationToolEvent:11-39][stored-image]
  保存参数并调用 `ImageGenToolArguments.serializer()`；
  [stable command event:8-10](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-storage/spec/clean-models/src/commonMain/kotlin/io/github/stream29/kodex/agentstorage/cleanmodels/stable/StableCommandExecutionToolEvent.kt#L8-L10)
  与 [stable image-view event:11-13](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-storage/spec/clean-models/src/commonMain/kotlin/io/github/stream29/kodex/agentstorage/cleanmodels/stable/StableImageViewToolEvent.kt#L11-L13)
  同样依赖相应 DTO。
- 三组一起折叠时，五节点 SCC 为 `C、T、image-generation、unified-exec、view-image`；
  任意一组单独折叠也有三节点环。这里的 T 仍可保留原 ID，重命名不是环的根因。
- **规划选项，不在此实施：**
  1. 保留这三组现有低层 spec 项目，暂不合并，代价是无法一次减完项目。
  2. 单独设计 handler/event adapter 边界，将依赖 storage 的组装放在上层；
     底层合并工具仅含 client、DTO、schema。需同步处理 runtime 和
     state 的 schema 消费，不能把整个 handler 文件随意塞回 specMain。
  3. 由 backend/protocol 线统一重选 clean-models/工具值模型归属。
     这是跨域契约调整，不能复制 DTO、类型别名转发或改变 wire 来绕环。

#### B2 — agent-state 联动，plan/context 的“先能合并”不等于全仓能合并

- state impl 依赖两工具：[state build:23-24][state-build]。
  工具又依赖 state contract：[plan build:10][i-plan]、
  [context build:10,15][i-context]。
- 源码对应：[UpdatePlanTool:6-31][plan-handler] 使用 `KodexAgentState`
  和 `appendPlanUpdate`；[GetContextRemainingTool:3-25][context-handler]
  使用 `tokensUntilCompaction`。context-window 的
  [生产边:9-12][context-window-build] 再指向 state contract/storage contract。
- 若 backend 将 `agent-state-spec-contract + agent-state-impl-state -> A`，
  则 `A -> plan -> A`、`A -> get-context-remaining -> A` 成环；
  context-window 同属 SCC。即使本线不合并这两个工具，backend 的折叠也会
  通过原 `tool-impl-*` 出现相同问题，须共享为 backend 规划输入。
- schema 收集是具体原因，不是要求禁用所有 impl 消费：
  [KodexVisibleToolSpecs:24-61][visible-specs] 同时读取 `PlanTools.spec`、
  `GetContextRemainingTools.spec` 及其他工具静态定义；这与
  [runtime 绑定 handler:65-68][runtime-tools] 的职责不同。
  候选解法是把纯 schema/name 与 state-bound handler 拆清或调整组合归属，
  不是把 state impl 大量搬入 specMain。

#### B3 — 不应误报为阻塞的边

- `multi-agent`、`request-user-input` 的 impl **目前不依赖 contract/storage**，
  只有 schema 和 OpenAI `ResponsesApiTool`：
  [multi build][i-multi]、[input build][i-input]；
  虽然 C 消费两者 DTO，折叠这两组不构成 B1。
- `tool-search` impl 只依赖 search-index/模型/schema，不依赖 T/C；
  真正的 `PendingToolSearchEvent -> StableToolSearchEvent` adapter 在
  [runtime decorator:166-174][runtime-handler]，不能因“工具相关”就下移回 search
  项目，否则会人为创建 `C -> search -> C`。
- runtime composition 消费实现是必要组装，不是反向耦合错误。
  当前工具生产脚本未见指向 agent-runtime 项目的边；本线不把
  runtime consumer 本身判成新增 SCC。

### 4. 消费者覆盖与边界

#### 4.1 全部直接 Gradle 工具消费者

表中省略 `:tool-` 前缀；main/test 指源集，不混为生产环。
工具内部各边已经逐行收录于 2.1。

| 外部项目 | main 的直接工具边 | test 的直接工具边 / 构建证据 |
| --- | --- | --- |
| agent-runtime-impl-composition | spec-contract；impl apply-patch/current-time/get-context-remaining/image-generation/plan/tool-search/unified-exec/view-image/web-run | [build:20-29][runtime-build] |
| agent-runtime-impl-decorator-tool | spec-contract、impl-tool-search | impl apply-patch/plan/unified-exec；[build:11-26][runtime-decorator-build] |
| agent-runtime-spec-contract | spec-unified-exec | [build:8-11][runtime-spec-build] |
| agent-state-impl-state | spec-tool-search；impl 的 11 个业务工具（除 builder） | [build:17-28][state-build] |
| agent-storage-spec-clean-models | spec image-generation/request-user-input/multi-agent/tool-search/unified-exec/view-image | [build:13-18][storage-build] |
| app-spec-application | spec-multi-agent | [build:13](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/spec/application/build.gradle.kts#L13) |
| app-component-agent-spec | spec-request-user-input/multi-agent/unified-exec | [build:17-19](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/agent/spec/build.gradle.kts#L17-L19) |
| app-component-request-user-input-spec | spec-request-user-input | [build:9](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/request-user-input/spec/build.gradle.kts#L9) |
| app-component-suggest-subagent-task-spec | spec-multi-agent | [build:9](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/suggest-subagent-task/spec/build.gradle.kts#L9) |
| mcp-spec-contract | spec-contract | [build:10](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/spec/contract/build.gradle.kts#L10) |
| mcp-impl-composition | 无 | impl-plan；[build:29](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/impl/composition/build.gradle.kts#L29) |
| integration-test | 无 | impl image-generation/request-user-input/view-image/web-run；[build:36-39](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/integration-test/build.gradle.kts#L36-L39) |

#### 4.2 不能只改 build 引用：跨域源码与间接消费

- runtime contract 真正向外暴露 `UnifiedExecClient`：
  [AgentRuntime:7,23-34,70][runtime-contract-code]。
  runtime composition 负责 client 生命周期和工具组合
  [KodexAgentTools:51-97][runtime-tools]，decorator 消费 `Tool`
  并防止已被 state-bound handler 完成的事件二次追加
  [runtime handler:101-113][runtime-handler]。
- storage 除 clean-models 外，contract-ext 也处理 user-input/multi-agent 值：
  [RemoteCompactionRetention:16-23](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-storage/spec/contract-ext/src/commonMain/kotlin/io/github/stream29/kodex/agentstorage/contract/ext/RemoteCompactionRetention.kt#L16-L23)。
  存储兼容性不能由工具单元测试替代。
- MCP 的 [McpTool:3](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/spec/contract/src/commonMain/kotlin/io/github/stream29/kodex/mcp/contract/McpTool.kt#L3)
  使用共享 Tool；RPC 的
  [GlobalRpc:17](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/spec/contract/src/commonMain/kotlin/io/github/stream29/kodex/rpc/contract/GlobalRpc.kt#L17)、
  [CreatedSuggestedSession:3-16](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/spec/models/src/commonMain/kotlin/io/github/stream29/kodex/rpc/models/CreatedSuggestedSession.kt#L3-L16)、
  [BackendGlobalRpc:14](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/server/src/commonMain/kotlin/io/github/stream29/kodex/rpc/server/BackendGlobalRpc.kt#L14)
  消费 multi-agent；[ShellSessionState:3-16](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/spec/models/src/commonMain/kotlin/io/github/stream29/kodex/rpc/models/ShellSessionState.kt#L3-L16)
  消费 exec 值。RPC 模型的依赖传播必须保持，不只检查直接 project 边。
- app contract 和实现存在不同消费：
  [AgentState:3](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/agent/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/agent/contract/AgentState.kt#L3)、
  [RequestUserInputViewModel contract:3-4](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/request-user-input/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/agent/contract/RequestUserInputViewModel.kt#L3-L4)、
  [SuggestSubagentTaskViewModel contract:10-11](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/suggest-subagent-task/spec/src/commonMain/kotlin/io/github/stream29/kodex/app/agent/contract/SuggestSubagentTaskViewModel.kt#L10-L11)；
  UI 执行部分见下节，不能把它们合并进工具项目。
- 显示与持久化投影还包括
  [history view 的六类值 imports:84-91](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/history/CleanEventView.kt#L84-L91)、
  [history viewmodel:37-39](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/history/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/cli/history/HistoryItemHeaderFactory.kt#L37-L39)、
  [shared history views](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/view/components/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/history/RequestUserInputHistoryView.kt#L13-L14)。
- app 组合/远程实现也间接消费：
  [ApplicationViewModel:37-38](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/application/src/commonMain/kotlin/io/github/stream29/kodex/cli/app/ApplicationViewModel.kt#L37-L38)、
  [RpcAgentViewModel:37-44](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/rpc/src/commonMain/kotlin/io/github/stream29/kodex/cli/rpc/RpcAgentViewModel.kt#L37-L44)、
  [RPC 通知 pending 分流:36-37](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/server/src/commonMain/kotlin/io/github/stream29/kodex/rpc/server/BackendNotifications.kt#L36-L37)。

### 5. spec-only、一对多与 schema/DTO 归属

- `contract` 是多工具/MCP 共用契约，不是第十三个“一对一工具”。
  [Tool.kt:13-47][tool-contract-code] 同时有接口和构造匿名实现的 `typedTool`；
  如另行要求该项目也使用 specMain，可选同项目拆出 factory 到 commonMain，
  **但不能将所有 Tool 实现并成 contract 的 impl**。默认保留其现有项目，
  这不影响其他选定项目的目标结构。
- `builder` 的 [JsonToolHandlerResult:3-28][builder-contract-code]
  是普通结果代数，没有生成的 DTO；serialization plugin 是当前脚本事实，
  不等于该文件需要 `@Serializable`。其
  [JSON/text/functionOutput adapters:22-150][builder-code]
  依赖 Tool/storage completion，不应把整份 JsonToolBuilder.kt 移到 specMain。
  current-time/context/plan 依赖它的实现是合法组合，[exec 仅测试使用][i-exec]。
- `apply-patch` 为 [freeform grammar:3-29][apply-contract-code]，
  [handler:18-56][apply-handler] 区分 JSON 误调用与 parsed patch；
  不能为了“统一 builder”改输入协议。
- multi-agent/user-input 的工具 impl 只构建规范与模型可见说明；
  实际完成事件由 [subagent ViewModel:116-154][subagent-vm]、
  [input ViewModel:107-150][input-vm] 负责，包含用户确认、修订号/重复提交保护。
  这是多个领域消费一个工具契约，而不是多个同名 impl 目录。
  合并工具脚本不授权下移 UI、Session 创建或 RPC 交互逻辑。
- unified-exec 一个 `UnifiedExecClient` 服务 exec/write 两个工具，
  [createTools:41-105][exec-handler] 和 [close:109-120][exec-handler]
  共享同一 client。不能按两个 tool name 拆成独立 client 或改变关闭语义。
  还有 [生产 ShellClient-backed 实现:28-45][exec-client]
  与测试替身；平台 actual 是实现层次，不是四个额外合并项目。
- schema 归属当前不一致：current-time/context/plan/web-run schema 在 spec，
  image/multi-agent/user-input/search/exec/view-image 在 impl。只移动旧 spec 文件
  可以保留既有行为，但“规范代码都在 specMain”还需要按职责拆文件，
  不是扫描到 `*Schemas.kt` 就全部上移。
- 保留 schema 与普通工具输出的既有区别，不在搬迁时顺手“修正”协议：
  [current-time 的 outputSchema 与 textTool:18-38](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/current-time/src/commonMain/kotlin/io/github/stream29/kodex/tool/currenttime/CurrentTimeTools.kt#L18-L38)
  同时存在；context 也用 [textTool 返回文字][context-handler]。
  是否另改为 JSON 不是本次源集迁移事项。
- 具体陷阱：[UnifiedExecSchemas:19-20][exec-schema] 调用 impl 的
  `execCommandHostPlatform` 和 renderer；
  [ExecCommandShellDescription:9-25][exec-description] 还引用 `UnifiedExecTools`。
  整体搬 schema 到 specMain 会出现 spec -> common 符号引用，须把纯契约
  参数化/拆分后单独审查；保留动态 schema 构造在 commonMain 则无这条新增边。
- [ToolSearchModels:45-52][search-models] KDoc 链接 `ToolSearchDefaultLimit`，
  常量实际在 [impl:3-4](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/tool-search/src/commonMain/kotlin/io/github/stream29/kodex/tool/toolsearch/ToolSearchConstants.kt#L3-L4)。
  是文档归属/链接检查项，不凭 KDoc 链接宣称 Kotlin 编译已阻塞。
- DTO 需保留 `SerialName`、默认值、sealed discriminator、package：
  [image-generation:13-30][image-models]、
  [multi-agent:20-62][multi-models]（accepted/rejected、EncodeDefault）、
  [user-input:7-53][input-models]（autoResolutionMs、options nullable）、
  [search:49-80][search-models]、
  [exec:30-93][exec-models]、
  [view-image:12-34][view-models]。web-run 参数 DTO 则在
  [WebRunToolClient:3-29][web-client] 引入的 OpenAI SearchCommands/Request/Response；
  plan handler 也使用 [OpenAI UpdatePlanArgs:8,39-41][plan-handler]，不应复制到工具 spec。

### 6. 平台、interop、测试和任务风险（尚未运行验证）

- **直接平台一致，传递平台不一致。** tool 的 S/I 都是 host convention：
  [JVM + 四 Native targets][host-targets]、
  [JS Node library / Mocha][host-convention]；不能宣称它们覆盖 Wasm/iOS。
  shell-client/images/search-index 的 spec 使用 shared，
  [shared:19-39][shared-convention] 额外有 JS browser、wasmJs、iosArm64。
  工具合并应保持 host，不应为了接收 shared spec 擅增/擅删 targets。
- unified-exec 的 [显式 posix 模板:11-19][i-exec]
  覆盖 Linux x64/arm64、macOS arm64；原有
  [posix actual:7-13](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/unified-exec/src/posixMain/kotlin/io/github/stream29/kodex/tool/unifiedexec/ExecCommandShellDescription.posix.kt#L7-L13)、
  [JVM actual:3-9](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/unified-exec/src/jvmMain/kotlin/io/github/stream29/kodex/tool/unifiedexec/ExecCommandShellDescription.jvm.kt#L3-L9)、
  [JS actual:3-11](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/unified-exec/src/jsMain/kotlin/io/github/stream29/kodex/tool/unifiedexec/ExecCommandShellDescription.js.kt#L3-L11)、
  [MinGW actual:3](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/unified-exec/src/mingwX64Main/kotlin/io/github/stream29/kodex/tool/unifiedexec/ExecCommandShellDescription.mingw.kt#L3)
  全须保留，不把 posix 误改成任意 Native。
- 间接 Native/JS 成本与失败面真实存在，但未测：
  [shell-client:18-50,62-79][shell-impl] 有 processClient/posix、pty4j、
  node-pty、shellClientSpawn/shellClientConPty cinterop 和 Linux `-lutil`；
  [images-codec:17-45][image-codec] 有 skikoNative、JPEG ImageIO、JS sharp；
  [search-index:11-27][search-index] 有 lucene 中间层，JVM/Native 与 JS 路由不同。
  从原 DTO-only 消费变成合并项目消费可能扩大这些解析/链接任务与依赖，
  不等于违反允许依赖实现的决定，也不能据此声称已降低构建内存。
- 手工 `dependsOn` 后需检查原 hierarchy 是否仍生效。
  尤其不能只关闭模板然后忘接 posix/平台边；KGP internal 抑制只解诊断，
  不保证 metadata、IDE、publication 或 cinterop transform 正确。
- tool 本地是手写 `PropertyBuilder` schema，不见 KSP/schema 生成任务配置。
  serialization 则通过编译器插件生成 serializer，不能按“无 KSP”省去插件。
  当前版本见 [catalog:2-16,41-45,93](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/gradle/libs.versions.toml#L2-L16)
  与上述各 S/I 脚本；未验证自定义 specMain 的插件参与和 metadata 暴露。

#### 6.1 本地测试清单与任务迁移

文件名相对于对应项目 `src/commonTest/kotlin/…/<包名>/`。
全部当前 `kodex.kmp-tests` 及源集测试依赖须保留；实际任务由插件创建，
以下不是运行结果或已核验的 task graph。

| 原项目 | 全部 commonTest 文件 |
| --- | --- |
| spec/contract | ToolContractTest.kt |
| impl/apply-patch | ApplyPatchToolsTest.kt |
| impl/builder | JsonToolBuilderTest.kt |
| impl/current-time | CurrentTimeToolsTest.kt |
| impl/get-context-remaining | GetContextRemainingToolBehaviorTest.kt、GetContextRemainingToolsTest.kt |
| impl/image-generation | GeneratedImageArtifactsTest.kt、ImageGenerationToolClientTest.kt、ImageGenerationToolsTest.kt |
| impl/multi-agent | SuggestSubagentTaskToolsTest.kt |
| impl/plan | PlanToolsTest.kt、UpdatePlanToolTest.kt |
| impl/request-user-input | RequestUserInputToolsTest.kt |
| impl/tool-search | ToolSearchToolsTest.kt |
| impl/unified-exec | UnifiedExecTestShell.kt、UnifiedExecToolsTest.kt |
| impl/view-image | ViewImageToolClientTest.kt、ViewImageToolsTest.kt |
| impl/web-run | WebRunRequestControlsTest.kt、WebRunToolsTest.kt |

- [tests convention:1-7](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/build-logic/src/main/kotlin/kodex.kmp-tests.gradle.kts#L1-L7)
  与 [commonTest 库注入:50-55][host-targets] 使用 TestBalloon；
  没有根据已有文件自行增设 specTest。commonTest 仍测两个共享层；
  需确认测试对新 specMain internal 符号的 friend 可见性。
- unified-exec 有 [四个 expect test shell 值][exec-test-shell] 及
  `jvmTest/posixTest/mingwX64Test/jsTest` 四份 actual。真实进程/PTY 测试：
  [UnifiedExecToolsTest:252-328,369-400][exec-tests]。
  但它们引用的 [realIoTestConfig:53-55](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/unified-exec/src/commonTest/kotlin/io/github/stream29/kodex/tool/unifiedexec/UnifiedExecToolsTest.kt#L53-L55)
  **当前设置 `isEnabled = false`**；以后默认测试通过也不能证明真实进程/PTY
  路径通过。是否在隔离环境启用需单独授权，不应只验序列化用例。
- 当前测试数据部分内嵌、部分写文件：
  [ViewImageToolClientTest:20-42][view-tests] 在相对 `build/tmp/...` 建目录、
  内嵌 PNG 并清理；
  [GeneratedImageArtifactsTest:13-30][artifact-tests] 在系统临时目录写图片。
  重定位项目改变 working directory 和输出路径，需保持清理与 fixture 行为。
  “本地无资源目录”不代表测试无 IO。
- context/plan 的 commonTest 反向依赖 state impl、storage in-memory、
  OpenAI test client，[context:19-26][i-context]、[plan:17-23][i-plan]；
  这是测试闭环，不能直接混入 main SCC。需后续看实际测试 compilation/task
  是否引用新产物、是否意外循环构建。
- 下游回归不止 tool：storage 的
  [PendingToolEventSerializationTest:10-17](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-storage/spec/clean-models/src/commonTest/kotlin/io/github/stream29/kodex/agentstorage/cleanmodels/unstable/PendingToolEventSerializationTest.kt#L10-L17)
  和 [StableSpecializedToolEventSerializationTest:15-22](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-storage/spec/clean-models/src/commonTest/kotlin/io/github/stream29/kodex/agentstorage/cleanmodels/stable/StableSpecializedToolEventSerializationTest.kt#L15-L22)；
  state 的 schema/projection tests；runtime 的 routing/compaction tests；
  MCP route admission、integration conversation、RPC completed values/shell sessions；
  app 的 input/subagent/history/history-index/sidebar、RPC 和 docs recording tests；
  session in-memory、filesystem storage、hook notification tests。
  这些是静态 imports 扫描命中的回归域，不宣称都应在首批全量执行。
- 额外跨域测试证据：
  [session test:36-39](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-session/impl/in-memory/src/commonTest/kotlin/io/github/stream29/kodex/agentsession/inmemory/AgentRuntimeRunningTurnTest.kt#L36-L39)、
  [filesystem:8](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-storage/impl/filesystem/src/commonTest/kotlin/io/github/stream29/kodex/agentstorage/filesystem/FileSystemAgentStorageTest.kt#L8)、
  [hook notification:16-20](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/hook/impl/notification/src/commonTest/kotlin/io/github/stream29/kodex/cli/notification/NotificationHooksTest.kt#L16-L20)、
  [integration:67-76](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/integration-test/src/commonTest/kotlin/io/github/stream29/kodex/integrationtest/MinimalAgentConversationTest.kt#L67-L76)、
  [MCP admission:22-23](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/mcp/impl/composition/src/commonTest/kotlin/io/github/stream29/kodex/mcp/impl/McpRouteAdmissionTest.kt#L22-L23)。
- 任务路径候选机械映射：`:tool-impl-X:<task>` / `:tool-spec-X:<task>` 到
  `:tool-X:<task>`（保留项目除外）；须后续解析实际 TestBalloon/JVM/JS/Native、
  metadata/publication 任务，不能直接假设原两个 project 的任务集合并集。
  本轮未发现 tool 脚本定义额外 Copy/Exec/生成资源 task。

### 7. 建议分批与后续检查（未获实施授权）

1. **先冻结环处理策略。** 协调 backend 确认 B1/B2；
   拒绝全 12 对机械折叠。暂留三组阻塞 pair 和共享 contract 时，
   九组配对可作为无新增静态 main 环的候选，不代表必须在一批迁完。
2. **低耦合验证候选：multi-agent / request-user-input。**
   两组不含 Tool/storage adapter，适合检验 specMain serializer 与
   storage/RPC/app 独立消费；但仍需单独审批工具链实验及资源预算。
   builder/current-time/apply-patch 可随后检验共享适配、文本输出与 freeform。
3. **带外部实现候选：tool-search / web-run。**
   单测和 schema 稳定后，核查 Lucene/JS 分层与 OpenAI DTO 暴露；
   search 的 storage adapter 留 runtime。plan/context 以 B2 已有解法为进入条件。
4. **三组阻塞工具最后重新设计。** 没有获准的 DTO/handler 归属方案前保持原状，
   不通过转发项目、复制模型或抑制环检查来完成数字上的项目压缩。

验收必须分清静态检查和随后授权的动态检查：

- [ ] 对最终全部领域的合并映射重新投影 main 图；每条 SCC 给真实引用证据；
  commonTest/平台依赖与生成边另核查。
- [ ] 源集只增加预期 `commonMain -> specMain`；无 specMain 反向实现引用；
  原 posix/Native/JS/测试边和 targets 保留。检查精确抑制的正反控制及 commonTest。
- [ ] 检查插件、依赖配置与 `serializer()` 在 specMain、platform、外部 consumer
  的可见性；保存 JSON/schema golden、default/nullable/sealed name 和 KDoc。
- [ ] 检查 Tool 事件唯一完成、plan 不二次持久化、Session 审批、不自动答题、
  exec 两工具共享 client 与最终关闭、图片保存/清理。
- [ ] 经用户授权后在隔离资源上分别验证 metadata、JVM、JS Node、四个 Native、
  cinterop/链接、TestBalloon 发现与实际任务路径、Maven publication 与独立消费者。
- [ ] IDE Sync/导航/internal 可见性/内存、增量失效范围单独测；不拿项目数减少
  或 help 成功冒充开发体验结果。

### 8. 交付核验

- 本线仅修改本文件，任务树在静态审查交付后结束；不移动任务阶段、
  不改主任务/其他报告/产品。迁移与动态验收仍待后续明确授权。
- 静态审查已完成。结束 `Kodex` HEAD：
  `4388871137c56bd1b73d5b739cc98f036a790f3c`，与起始一致；
  `git -C Kodex status --porcelain` 为空。外层 HEAD 也仍为
  `c0a7d34c4d8a5a7e69538e68d80b56d5fd0bc671`。
- 文档检查：113 处文件 URI 存在且行号有效，68 个引用定义无缺失，
  全部本地 tool Kotlin 文件名覆盖；无尾随空格/损坏字符。
  已作删减、事实/提案分离和全文一致性复核；未生成临时文件或保留后台进程。

### 证据索引

[s-apply]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/spec/apply-patch/build.gradle.kts#L1-L11
[i-apply]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/apply-patch/build.gradle.kts#L1-L15
[s-builder]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/spec/builder/build.gradle.kts#L1-L14
[i-builder]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/builder/build.gradle.kts#L1-L18
[s-contract]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/spec/contract/build.gradle.kts#L1-L14
[s-time]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/spec/current-time/build.gradle.kts#L1-L13
[i-time]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/current-time/build.gradle.kts#L1-L20
[s-context]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/spec/get-context-remaining/build.gradle.kts#L1-L13
[i-context]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/get-context-remaining/build.gradle.kts#L1-L29
[s-image]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/spec/image-generation/build.gradle.kts#L1-L12
[i-image]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/image-generation/build.gradle.kts#L1-L25
[s-multi]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/spec/multi-agent/build.gradle.kts#L1-L12
[i-multi]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/multi-agent/build.gradle.kts#L1-L17
[s-plan]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/spec/plan/build.gradle.kts#L1-L13
[i-plan]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/plan/build.gradle.kts#L1-L26
[s-input]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/spec/request-user-input/build.gradle.kts#L1-L12
[i-input]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/request-user-input/build.gradle.kts#L1-L17
[s-search]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/spec/tool-search/build.gradle.kts#L1-L13
[i-search]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/tool-search/build.gradle.kts#L1-L16
[s-exec]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/spec/unified-exec/build.gradle.kts#L1-L14
[i-exec]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/unified-exec/build.gradle.kts#L1-L37
[s-view]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/spec/view-image/build.gradle.kts#L1-L12
[i-view]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/view-image/build.gradle.kts#L1-L17
[s-web]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/spec/web-run/build.gradle.kts#L1-L11
[i-web]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/web-run/build.gradle.kts#L1-L21
[storage-build]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-storage/spec/clean-models/build.gradle.kts#L9-L24
[state-build]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-state/impl/state/build.gradle.kts#L8-L38
[context-window-build]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-state/spec/context-window/build.gradle.kts#L8-L21
[runtime-build]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-runtime/impl/composition/build.gradle.kts#L8-L35
[runtime-decorator-build]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-runtime/impl/decorator/tool/build.gradle.kts#L8-L26
[runtime-spec-build]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-runtime/spec/contract/build.gradle.kts#L7-L12
[tool-contract-code]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/spec/contract/src/commonMain/kotlin/io/github/stream29/kodex/tool/contract/Tool.kt#L3-L47
[builder-contract-code]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/spec/builder/src/commonMain/kotlin/io/github/stream29/kodex/tool/builder/JsonToolBuilderSpec.kt#L3-L28
[builder-code]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/builder/src/commonMain/kotlin/io/github/stream29/kodex/tool/builder/JsonToolBuilder.kt#L22-L150
[apply-contract-code]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/spec/apply-patch/src/commonMain/kotlin/io/github/stream29/kodex/tool/applypatch/ApplyPatchToolSpec.kt#L3-L29
[apply-handler]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/apply-patch/src/commonMain/kotlin/io/github/stream29/kodex/tool/applypatch/ApplyPatchTools.kt#L18-L56
[image-handler]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/image-generation/src/commonMain/kotlin/io/github/stream29/kodex/tool/imagegeneration/ImageGenerationTools.kt#L58-L100
[view-handler]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/view-image/src/commonMain/kotlin/io/github/stream29/kodex/tool/viewimage/ViewImageTools.kt#L27-L50
[exec-handler]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/unified-exec/src/commonMain/kotlin/io/github/stream29/kodex/tool/unifiedexec/UnifiedExecTools.kt#L41-L120
[exec-client]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/unified-exec/src/commonMain/kotlin/io/github/stream29/kodex/tool/unifiedexec/UnifiedExecToolClient.kt#L28-L45
[exec-schema]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/unified-exec/src/commonMain/kotlin/io/github/stream29/kodex/tool/unifiedexec/UnifiedExecSchemas.kt#L6-L27
[exec-description]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/unified-exec/src/commonMain/kotlin/io/github/stream29/kodex/tool/unifiedexec/ExecCommandShellDescription.kt#L9-L25
[plan-handler]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/plan/src/commonMain/kotlin/io/github/stream29/kodex/tool/plan/UpdatePlanTool.kt#L3-L46
[context-handler]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/get-context-remaining/src/commonMain/kotlin/io/github/stream29/kodex/tool/getcontextremaining/GetContextRemainingTool.kt#L3-L25
[stored-image]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-storage/spec/clean-models/src/commonMain/kotlin/io/github/stream29/kodex/agentstorage/cleanmodels/stable/StableImageGenerationToolEvent.kt#L11-L39
[visible-specs]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-state/impl/state/src/commonMain/kotlin/io/github/stream29/kodex/agentstate/tool/KodexVisibleToolSpecs.kt#L24-L61
[runtime-tools]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-runtime/impl/composition/src/commonMain/kotlin/io/github/stream29/kodex/agentruntime/impl/KodexAgentTools.kt#L51-L97
[runtime-handler]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-runtime/impl/decorator/tool/src/commonMain/kotlin/io/github/stream29/kodex/agentruntime/decorator/tool/KodexToolRuntimeImpl.kt#L48-L174
[runtime-contract-code]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-runtime/spec/contract/src/commonMain/kotlin/io/github/stream29/kodex/agentruntime/contract/AgentRuntime.kt#L7-L70
[subagent-vm]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/suggest-subagent-task/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/cli/agent/SuggestSubagentTaskViewModel.kt#L116-L154
[input-vm]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/component/request-user-input/impl/viewmodel/src/commonMain/kotlin/io/github/stream29/kodex/cli/agent/RequestUserInputViewModel.kt#L107-L150
[image-models]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/spec/image-generation/src/commonMain/kotlin/io/github/stream29/kodex/tool/imagegeneration/ImageGenerationToolModels.kt#L13-L30
[multi-models]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/spec/multi-agent/src/commonMain/kotlin/io/github/stream29/kodex/tool/multiagent/SuggestSubagentTaskModels.kt#L20-L62
[input-models]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/spec/request-user-input/src/commonMain/kotlin/io/github/stream29/kodex/tool/requestuserinput/RequestUserInputModels.kt#L7-L53
[search-models]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/spec/tool-search/src/commonMain/kotlin/io/github/stream29/kodex/tool/toolsearch/ToolSearchModels.kt#L45-L80
[exec-models]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/spec/unified-exec/src/commonMain/kotlin/io/github/stream29/kodex/tool/unifiedexec/UnifiedExecModels.kt#L30-L93
[view-models]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/spec/view-image/src/commonMain/kotlin/io/github/stream29/kodex/tool/viewimage/ViewImageModels.kt#L12-L34
[web-client]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/web-run/src/commonMain/kotlin/io/github/stream29/kodex/tool/webrun/WebRunToolClient.kt#L3-L29
[host-targets]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/build-logic/src/main/kotlin/KodexHostKmp.kt#L22-L55
[host-convention]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/build-logic/src/main/kotlin/kodex.kmp-host.gradle.kts#L1-L22
[shared-convention]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/build-logic/src/main/kotlin/kodex.kmp-shared.gradle.kts#L19-L39
[shell-impl]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/shell-client/impl/build.gradle.kts#L18-L79
[image-codec]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/images-codec/impl/build.gradle.kts#L17-L45
[search-index]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/utils/search-index/impl/build.gradle.kts#L11-L27
[exec-test-shell]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/unified-exec/src/commonTest/kotlin/io/github/stream29/kodex/tool/unifiedexec/UnifiedExecTestShell.kt#L3-L11
[exec-tests]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/unified-exec/src/commonTest/kotlin/io/github/stream29/kodex/tool/unifiedexec/UnifiedExecToolsTest.kt#L252-L400
[view-tests]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/view-image/src/commonTest/kotlin/io/github/stream29/kodex/tool/viewimage/ViewImageToolClientTest.kt#L20-L42
[artifact-tests]: file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/tool/impl/image-generation/src/commonTest/kotlin/io/github/stream29/kodex/tool/imagegeneration/GeneratedImageArtifactsTest.kt#L13-L30

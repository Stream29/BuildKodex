# Codex / Kodex 模型调用差异审查

## 范围与证据

- 官方子模块：`343074d4207d572809bd8cea15f4be1d09d98e0b`（2026-08-22）→ `8d7cc24a87f4aa66aa434eb4f25f4f4bafc0e0a9`（2026-09-06 UTC），650 个提交，更新到审查时的 `origin/main`。
- Kodex 基线：`8d4cfba7512b5ba6bea470e211e7e9bb9a52826c`。工作区有并行进行的 UI 修改，本次未修改 Kodex 源码。
- 扫描上述提交记录，深入检查请求、历史投影、模型目录、工具/MCP、压缩和相关测试；不是对 650 个提交逐行完整审计。
- 明确排除用户否决的上下文换窗。不把官方的多 Agent、Guardian、固定回合编排或提示词策略当作必须跟随的设计。
- 以下优先级是审查建议，不是批准实施的任务或新的项目指导。
- 缓存影响主要是延迟、计算开销和费用；缓存未命中本身不等于模型推理能力下降。官方说明缓存不改变输出生成方式：[Prompt caching](https://developers.openai.com/api/docs/guides/prompt-caching)。

## 1. 已确认设计：简化压缩后的原文保留

- 当前 `buildCompactionPrefix` 遇到第一个超预算条目时，移除整个条目并停止；没有边界用户消息截断分支。
- 例：较新消息占 60K、边界用户消息占 8K、预算 64K，当前只保留 60K；旧实现还会保留边界消息约 4K 的截断内容。最新单条消息超过整个预算时，当前明文保留区甚至可以为空。
- 这影响的是压缩后的明文原文保留，不代表整个会话永久丢失：磁盘历史和服务端生成的 encrypted compaction 仍在，但摘要不保证保留原文细节。
- 追溯：`651c7b9c` 新增当前算法，`97efe9da`（2026-08-31）将 AgentState 接入新投影；其前一版 `agent-state/impl/.../RemoteCompactionV2.kt` 有 `truncateToTokenBudget`，仅用户消息可截断。
- 用户于 2026-09-07 明确确认：这是其设计结果，以不太影响效果为取舍，大幅降低实现复杂度。原审查将其称为“回归”并建议恢复边界截断的结论撤回；旧指导已同步修正，历史任务仅保留当时记录。
- 同次重构将文本估算从 UTF-8 字节数/4 改为 Kotlin `String.length`/4。对 BMP 汉字，后者可为旧估算的约三分之一；这是启发式预算变松，不是实测 tokenizer 的三倍误差。
- `InputImage` 仍按零 token 计费，纯图片消息只受到每条最少 1 token 的下限约束。官方最近加入图片预算并默认启用，防止图像密集的保留区超出预算。
- 保留现行原文保留设计，不恢复旧截断逻辑；本轮不为压缩、文本估算或图片预算新增任务。

证据：
- [当前保留算法，44–59、162–175 行](../../Kodex/agent-storage/contract-ext/src/commonMain/kotlin/io/github/stream29/kodex/agentstorage/contract/ext/RemoteCompactionRetention.kt#L44)。
- [AgentState 接入，172–194 行](../../Kodex/agent-state/impl/src/commonMain/kotlin/io/github/stream29/kodex/agentstate/impl/KodexAgentStateImpl.kt#L172)。
- [官方边界截断与图片计费，591–684 行](../codex/codex-rs/core/src/compact_remote_v2.rs#L591)；新增 `6677fd827d`，默认启用 `528fd7ace5`。
- [历史边界消息决策，已被后续简化取代](../../kanban/done/2026-08-27-retain-request-user-input-after-compaction.md#L15)；[当前保留规则](../../checklist/agent-state-and-runtime.md#L36)。
- [现有测试，113–125 行](../../Kodex/agent-storage/contract-ext/src/commonTest/kotlin/io/github/stream29/kodex/agentstorage/contract/ext/AgentStorageContractExtTest.kt#L113)检查预算耗尽后的提前停止读取。

## 2. 缓存：缺少显式亲和键和可测量数据

- 普通新 Session 没有设置 `promptCacheKey`，请求投影直接透传可空值；官方常规请求始终提供稳定键，通常基于 Session 身份。
- 这不等于关闭缓存：无键也可能命中；稳定键影响路由，不保证命中。当前不能量化 Kodex 的实际损失。
- Kodex `TokenUsage` 只解码 input/output/total，未保留 `input_tokens_details.cached_tokens`、cache-write 或 reasoning token 数；当前无法从这些持久化总量还原命中率。
- 官方解析 cache-read/cache-write/reasoning 用量；近期 `5f79a92e39` 还将 response usage 写入历史。值得借鉴的是观测能力，而不是复制其整套遥测。
- 建议先沿现有请求身份派生稳定缓存键，并保留必要的用量与首包时间；不要为此增加第二套 Session 身份或存储真源。

证据：
- [新 Session 设置，549–558 行](../../Kodex/app/viewmodel/application/src/commonMain/kotlin/io/github/stream29/kodex/cli/app/Application.kt#L549)。
- [请求透传，15–31 行](../../Kodex/agent-state/impl/src/commonMain/kotlin/io/github/stream29/kodex/agentstate/impl/KodexRequestProjection.kt#L15)。
- [Kodex usage DTO，512–519 行](../../Kodex/openai/models/src/commonMain/kotlin/io/github/stream29/kodex/openai/ResponsesApiModels.kt#L512)。
- [官方缓存键，504–515、976–991 行](../codex/codex-rs/core/src/client.rs#L504)；[官方 usage 解码，127–166 行](../codex/codex-rs/codex-api/src/sse/responses.rs#L127)。
- 路由语义参见 [OpenAI 缓存文档](https://developers.openai.com/api/docs/guides/prompt-caching#prompt-cache-keys)。

## 3. 缓存：动态前缀有可识别的失效触发点

- 每次请求的顺序是 planning → skills / AGENTS.md / environment → durable history。
- 环境前缀含当前日期和 Session 名称；改标题、跨日期，以及实际修改 AGENTS.md / skill metadata，会改变历史之前的前缀。
- 因而这些事件可能使后续长历史不能复用旧的完整前缀缓存；不是每次请求都必然失效。日期不是逐秒时间戳，skills 已排序，不能声称每轮有随机抖动。
- AGENTS.md / skills 逐请求热重载是已确认设计，不建议为缓存改回官方旧的缓存生命周期。
- 可在保留热重载语义的前提下，单独评估 Session 展示名称是否值得出现在长历史之前；涉及上下文内容取舍，未作设计变更。

证据：
- [请求拼接，172–194 行](../../Kodex/agent-state/impl/src/commonMain/kotlin/io/github/stream29/kodex/agentstate/impl/KodexAgentStateImpl.kt#L172)。
- [日期和 Session 名称，110–123 行](../../Kodex/agent-context/prefix/render/src/commonMain/kotlin/io/github/stream29/kodex/agentcontext/prefix/render/AgentContextPrefixRenderer.kt#L110)。
- [skills 排序，85–93 行](../../Kodex/agent-context/skill/filesystem/src/commonMain/kotlin/io/github/stream29/kodex/agentcontext/skill/filesystem/FileSystemSkillsResolver.kt#L85)。

## 4. MCP：纯文本包装开销与加密内容类型缺失

- Kodex 只有遇到 image 时才采用 typed content items；纯文本 MCP 结果回传为序列化后的整个 content JSON 数组，含包装及转义。
- 官方 `75cb7c903d`（2026-08-25）改为无 structuredContent 的结果统一使用 typed items，纯文本直接成为 `input_text`，减少无用包装。
- 更重要的条件性差异：官方识别 text item 的 `_meta["codex/encryptedContent"] == true` 并转换成 `encrypted_content`；Kodex text 分支未识别此标记。
- 因此遇到这种 MCP 结果时，Kodex 会把密文当普通文本，而不是保留加密内容的协议语义。当前未证明用户正在使用的 MCP 会返回这种内容。
- 普通图片已有 typed image 支持，structuredContent 优先返回序列化文本也与官方一致；不能概括为“MCP 输出全部丢失”。

证据：
- [Kodex 转换，61–78、111–150 行](../../Kodex/openai/models/src/commonMain/kotlin/io/github/stream29/kodex/openai/FunctionCallOutputModels.kt#L61)。
- [官方转换，2299–2352 行](../codex/codex-rs/protocol/src/models.rs#L2299)。

## 5. 推理设置：显式选择 Medium 与使用后端默认混在一起

- UI 新 Session 设置把所选 effort 传给 `Reasoning`，但 `Reasoning.effort` 将 `Medium` 定为默认值，并标记 `EncodeDefault.NEVER`；整个默认 Reasoning 也会被省略。
- 所以选择 Medium 不会显式发送 `"effort":"medium"`，而其他非默认档位会发送。官方会解析模型默认值/显式选择并发送 resolved effort。
- 这是确定的请求语义差异；是否实际降档取决于所用端点的默认值，不能把目录 `default_reasoning_level` 直接当作省略字段时的后端行为。
- 建议区分“用户明确选择 Medium”与“采用模型默认”，并记录响应报告的实际 reasoning 配置，避免 UI 与实际采样控制脱节。
- 当前会话使用 Low，不是本项 Medium 情形的实例。

证据：
- [Reasoning 序列化，391–398 行](../../Kodex/openai/models/src/commonMain/kotlin/io/github/stream29/kodex/openai/ResponsesApiModels.kt#L391)及 [request 默认省略，38–40 行](../../Kodex/openai/models/src/commonMain/kotlin/io/github/stream29/kodex/openai/ResponsesApiModels.kt#L38)。
- [官方 build_reasoning，869–890 行](../codex/codex-rs/core/src/client.rs#L869)。

## 6. 工具暴露：缺少不支持 Tool Search 时的降级

- Kodex 固定把 MCP、view_image、image generation 放到 deferred search；visibleToolSpecs 不读取模型能力，ModelInfo 也没有 `supports_search_tool`。
- 官方结合模型能力和 provider namespace 能力决定 search 是否可用，不可用时移除 deferred exposure、保留可用的 direct exposure。
- 这是选择不支持该协议的模型时的兼容性缺口；不能说当前 Astra 因此看不到工具。本会话 Tool Search 正常工作。
- 项目现有 `tool-search.md` 已要求不支持时直接暴露工具；补齐该路径不需要引入官方的工具编排架构。

证据：
- [Kodex visibleToolSpecs，43–74 行](../../Kodex/agent-state/tool/src/commonMain/kotlin/io/github/stream29/kodex/agentstate/tool/KodexVisibleToolSpecs.kt#L43)。
- [官方 exposure 选择，245–253 行](../codex/codex-rs/core/src/tools/spec_plan.rs#L245)及 [能力判断，629–631 行](../codex/codex-rs/core/src/tools/spec_plan.rs#L629)。
- [现有约束，第 13 行](../../checklist/tool-search.md#L13)。

## 7. 最近值得借鉴的独立改进

| 官方改动 | 结论 |
| --- | --- |
| 图片保留预算 `6677fd827d`、`528fd7ace5` | 保留为官方差异记录，不作为改写用户已确认的简化保留设计的理由；本轮不新增任务。 |
| typed MCP output `75cb7c903d` | 可减少包装并保留协议语义，见第 4 节。 |
| per-tool MCP output limit `f742dabc6f` | 对超大工具结果有价值；不能直接套用全局硬截断，否则反而丢上下文。 |
| skill 路径别名 `7c3747941a` | 保留 skill 数量和描述、仅缩短重复路径；当前目录不大，收益需先按体积判断。 |
| content kinds `afb797cae6`（8 月 25 日默认启用） | 官方用 internal metadata 标记内容来源；Kodex DTO 没有该字段。属于已确认协议差异，服务端质量收益没有证据，不列为确定降智原因。 |
| 模型专属 instructions template | Astra 的模板约 21K 字符，包含任务连续性、用户插入消息、skills 等指导。Kodex 不加载该目录字段；适合逐条比较，不适合整段导入官方交互/权限策略。 |
| PowerShell 版本上下文 `dc031d4bc7` | 可避免 PowerShell 5/7 语法误用；上游仍为实验开关，当前 Linux/Bash 会话不受益。 |
| MCP catalog 与 client 同生命周期 `32351a7b1a`、`2cfee7de25` | 值得借鉴刷新/失败保留/连接复用的测试案例。Kodex 已有 refresh → publishCatalog，不应直接断言缺少刷新能力。 |
| reasoning configuration history `56a8470aa0` | 上游默认关闭且仅作用于特定模型/协议；不因“较新”就引入。 |

- content kinds 证据：[官方开关，998–1003 行](../codex/codex-rs/features/src/lib.rs#L998)、[协议 metadata，944 行附近](../codex/codex-rs/protocol/src/models.rs#L944)；Kodex [Message DTO，72–78 行](../../Kodex/openai/models/src/commonMain/kotlin/io/github/stream29/kodex/openai/ResponseItemModels.kt#L72)。
- MCP refresh 证据：[McpClientOwner.refresh，155–181 行](../../Kodex/mcp/impl/src/commonMain/kotlin/io/github/stream29/kodex/mcp/impl/McpClientImpl.kt#L155)。
- 模型提示词证据：[Astra instructions_template，第 75 行](../codex/codex-rs/models-manager/models.json#L75)、[官方选择模型指令，717 行附近](../codex/codex-rs/core/src/session/mod.rs#L717)；Kodex [ModelInfo，第 39 行起](../../Kodex/openai/models/src/commonMain/kotlin/io/github/stream29/kodex/openai/ModelCatalogModels.kt#L39)未建模 `model_messages`。
- 换窗、history-notes 驱动的新上下文管理不在建议范围；Guardian、主动多 Agent、persistent 模式也不作为本次对齐建议。

## 8. 已排除的误判与应保留的差异

- **不能把 include 为空等同于丢失 encrypted reasoning。** 官方源码仍显式请求该 include，但最新 API 文档说明 `store=false` 已默认返回加密推理内容。
- 对当前 Session `236` 的只读复查确认：即使 `include=[]`，`work/22.json`、`77.json`、`84.json`、`91.json`、`98.json`、`105.json` 的 `item.encrypted_content` 都非空；Kodex `StableReasoning` 原样投影内层 item。最初只看外层 discriminator 的统计已作废。
- `reasoning.context=auto` 也不能直接认定少回传历史推理：公开 API 的 GPT-5.6 默认就是 `all_turns`，需要核对响应有效值，而不是照抄 Rust 在 Lite 分支中的显式设置。[官方 reasoning 说明](https://developers.openai.com/api/docs/guides/reasoning#persisted-reasoning)。
- Assistant `phase` 和已取得的 reasoning payload 有保留；最新 compaction payload 也随活动窗口回传，未发现这些路径普遍丢弃正文/推理的证据。
- 已完成的 Tool Search schema 进入历史；remote compaction 的输入由同一历史投影构造，没有发现压缩输入主动清空这些 schema。
- 项目文档与 skills 的逐请求加载、临时 prefix 不参与压缩、计划由事件保留、独立 Session 建议工具，均按现有设计审查，没有建议恢复已经删除的 Agent 树/官方协作提示。
- ModelInfo 没有复制官方全部提示词和功能开关，不足以证明模型变弱。Astra 没有顶层 `base_instructions` 字段，但有非空的 `model_messages.instructions_template`；不能误判为官方没有提示词，也不能把没复制整套提示词直接认定为缺陷。

证据：
- [StableReasoning 原样回传，12–23 行](../../Kodex/agent-storage/clean-models/src/commonMain/kotlin/io/github/stream29/kodex/agentstorage/cleanmodels/stable/work/StableProviderEvent.kt#L12)。
- [Assistant phase，30–51 行](../../Kodex/agent-storage/clean-models/src/commonMain/kotlin/io/github/stream29/kodex/agentstorage/cleanmodels/stable/index/StableMessageEvent.kt#L30)。
- [完整活动窗口，35–70 行](../../Kodex/agent-storage/contract-ext/src/commonMain/kotlin/io/github/stream29/kodex/agentstorage/contract/ext/AgentStorageProjection.kt#L35)。
- [Tool Search history，27–49 行](../../Kodex/agent-storage/clean-models/src/commonMain/kotlin/io/github/stream29/kodex/agentstorage/cleanmodels/stable/work/StableToolSearchEvent.kt#L27)。

## 验证边界

- 已验证子模块工作区干净、旧提交为新 HEAD 祖先、HEAD 与抓取的 `origin/main` 一致；没有创建 Git commit。
- 做了源码/历史差异核对、现有测试内容检查及当前会话落盘字段的只读验证；没有输出推理正文或凭据。
- 未修改应用代码，因此未运行编译或 Gradle 单元测试；没有声称现有测试在本次执行通过。
- 没有发起额外模型请求、端到端 A/B 或缓存基准；不能给出命中率、延迟改善或回答质量下降百分比。
- 用户指定的后续讨论仅为[稳定缓存键](../../kanban/done/2026-09-07-add-stable-prompt-cache-key.md)、[显式 Medium effort](../../kanban/done/2026-09-07-send-explicit-medium-reasoning-effort.md)、MCP 文本去包装；讨论本身不授权实现。缓存收益仍需实际测量。
- 2026-09-07 用户评估 MCP 文本去包装收益不大，明确丢弃该任务；对应规划文件已删除，未实施，保留现有 MCP 输出行为。本报告中的 MCP 差异仅作历史审查发现，不构成继续推进授权。

# Task Tree

- `Trace production search index consumers and runtime ownership`()
- `Distinguish shared search event data from backend indexing mechanisms`()
- `Confirm the ToolSearch contract backend split`()
- `Freeze the declaration and coverage preserving cutover manifest`()
- `Implement the authorized contract backend split with subagent`()
- `Run centralized dependency and platform validation`()

# Details

- 用户授权 subagent 实施，同时主 Session 继续讨论下一项；
  [Session1026](file:///home/stream/.kodex/sessions/1026) 已返回最终编辑回执，未运行构建；
  父 Session 已完成下述范围验收；没有将整个剩余迁移标为完成。
  子任务只编辑 ToolSearch 子树、注册树及必要直接消费者接线，不改共享任务/规范；
  禁止构建、提交推送及操作用户进程，父 Session 集中串行验收。
  两个新模块保留原 Host 六目标；utils/search-index 目录本轮不改。
  实施前 1516 份文件哈希及 9 份 ToolSearch 源码快照：
  `file:///home/stream/ACodeSpace/demo/kodex-detekt-998/toolsearch-before/`。
- 父复核：8 份原文件逐字节移动，混合模型按原声明块精确拆分，
  四个消费者脚本仅替换项目坐标。新增枚举、空成功及省略 limit 的协议回归。
  首次构建暴露嵌套 moduleTree 误用顶层 includeGroupingRoot 参数，导致解析根 tool-search 目录；
  已移除该参数以调用嵌套成员方法，不改变注册 DSL 或其他树，重新编译通过。
- 用户已确认同样采用 contract/backend 方法，具体为 `tool/tool-search/contract` 与
  `tool/tool-search/backend`；[规范职责](../../checklist/kmp-spec-impl-source-sets.md#toolsearch-contractbackend-boundary--2026-10-21)已同步。
  前者承载共享事件数据，后者承载引擎、索引输入/转换、工具描述构建和 search-index 依赖。
  父目录仅分组，无转发项目；不把 utils/search-index 再机械拆成一对模块。
- 平台范围不因角色拆分擅改：尚未批准新增 Wasm/iOS 创建能力或收窄 search-index 既有导出。
  撤回先前以复用 JS fallback 为理由优先补 Wasm/iOS 的推荐：技术可行不等于业务需求。
- 当前唯一直接生产消费者是 `tool/tool-search` 的 `ToolSearchEngine`，用于模型触发
  `tool_search` 时查找 deferred 工具；目录包含 MCP 工具和本地 ViewImage/ImageGeneration。
  Agent runtime 在 MCP 工具目录变化时重建搜索快照，工具 runtime 执行请求并生成结果事件。
  这是后端 Agent 工具发现，不是前端搜索框、会话目录或历史索引。
- 前端使用共享的 `SearchToolCallParams`、`ToolSearchResult` 和历史事件数据展示结果，
  不直接创建或执行该索引；核查时没有仅凭这一点宣称依赖隔离，最终解析结果见验收。
- 拆分前依赖边是 `agent-storage/clean-models -> tool/tool-search -> utils/search-index`；
  `tool/core -> agent-storage/clean-models`，MCP contract 也通过 Tool 接口进入这条链。
  上轮实际解析的 app/application、app/view、MCP contract JVM runtime 图均含
  `utils-search-index` 和 Lucene；核查时对应构建脚本未变，当前子任务正在切换。
  这是 ToolSearch 共享数据与执行机制混装造成的传递依赖，不是前端调用搜索能力的证据。
- 实施按逐声明分类，没有把混合的 ToolSearchModels 文件整体归到 contract：
  SearchToolCallParams、ToolSearchExecution、ToolSearchResult 及必要协议常量共享；
  ToolSearchDocument、文档输出/来源模型、转换函数及引擎属于 backend。
  clean-models 和前端切到 contract，Agent 执行消费者按需切 backend；backend 单向依赖 contract。
  保留类型身份、wire 形状、默认值及 specMain/actual 规则。
- 授权并派发不等于迁移或前端依赖隔离验收完成。utils/search-index 的工厂
  仍有 1 项源集缺口，不能因 ToolSearch 拆分就声称已消除，也不凭没有当前调用者收回导出。
- 代码依据（ToolSearch 源文件使用派发前快照，避免子任务搬迁造成链接失效）：
  - [直接创建索引](file:///home/stream/ACodeSpace/demo/kodex-detekt-998/toolsearch-before/tool/tool-search/src/commonMain/kotlin/io/github/stream29/kodex/tool/toolsearch/ToolSearchEngine.kt#L13)。
  - [目录快照与重建](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-runtime/composition/src/commonMain/kotlin/io/github/stream29/kodex/agentruntime/impl/KodexAgentTools.kt#L111)。
  - [MCP 与本地 deferred 工具目录](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-state/core/src/commonMain/kotlin/io/github/stream29/kodex/agentstate/tool/KodexVisibleToolSpecs.kt#L33)。
  - [实际执行搜索请求](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-runtime/decorator/tool/src/commonMain/kotlin/io/github/stream29/kodex/agentruntime/decorator/tool/KodexToolRuntimeImpl.kt#L65)。
  - [派发前共享事件依赖](file:///home/stream/ACodeSpace/demo/kodex-detekt-998/toolsearch-before/agent-storage/clean-models/build.gradle.kts#L18)与[参数/结果字段](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-storage/clean-models/src/specMain/kotlin/io/github/stream29/kodex/agentstorage/cleanmodels/stable/StableToolSearchEvent.kt#L24)。
- 解析图：`file:///home/stream/ACodeSpace/demo/kodex-detekt-998/mcp-resolved-model.json`。
  此图是拆分前 42 项检查点，不能用于宣称新闭包已通过；新图和最终结果见下节。

## 验收 — 2026-10-21 / Session998

- 新解析图 `parallel-resolved-model.json` 覆盖 12 项目、24 个 JVM 生产编译/运行配置；
  均无 ToolSearch backend、search-index、Lucene、MCP backend/SDK 或后端日志。
  原 ToolSearch 项目不存在，两新模块保留 Host 六目标。
- contract/backend metadata、JVM 3+7 项及 Node 3+7 项测试通过；
  backend Linux klib、真实 Agent/History/RPC/CLI 消费者编译通过。
  协议回归覆盖参数、枚举标识、空成功结果、错误结果及省略 limit。
- 合并批次 JVM 729 项中 726 通过，3 项失败与已复现干净基线精确一致；
  完整对照及源码哈希在 `parallel-baseline-comparison.json`。未运行真实模型集成测试。
- 最终 lint 34 项/10 模块，ToolSearch 自身无发现；
  search-index 的 1 项工厂缺口及其他未授权范围继续保留。
- 统一证据：`file:///home/stream/ACodeSpace/demo/kodex-detekt-998/parallel-validation-summary.json`。
  模型探针内容留存证据后删除，未提交推送。

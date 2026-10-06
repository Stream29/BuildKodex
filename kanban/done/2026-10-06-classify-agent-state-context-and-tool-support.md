# Task Tree

- `Classify budget extensions and concrete tool projections`()
- `Receive review of support consolidation and consumer closure`()
- `Move context-window and its tests to spec/context-window`()
- `Merge tool support and its tests into impl/state`()
- `Retarget actual production and test consumers`()
- `Remove old coordinates and validate the projected graph`()
- `Return evidence to the parent task and receive scoped acceptance`()

# Details

## Context-window

- [Parent task](2026-10-06-hard-migrate-agent-state.md)；执行完整 State 单根硬迁移。
- 原 `contextWindowTokenStatus` / `tokensUntilCompaction` 与 FQCN 保留；
  输入为真实 KodexAgentState 与 OpenAiModelCatalogStore，
  输出为既有 ModelContextWindowTokenStatus/nullable Long。
- 使用单个捕获的 storage index 读取 settings/tokenCount；null 表示没有可计算
  计数，compaction synthetic zero 有效。catalog 的解析/预算公式仍归 OpenAI。
- commonMain 仅依赖 State/storage/model-catalog/models 的 spec；
  原 tests 允许使用 State、storage、catalog 实现，不把测试边当生产依赖环。
- KDoc补实际 suspend storage read 失败/取消；不制造预算服务、复制计数状态
  或自动触发压缩。原三项测试整体保留。

## Tool support ablation

- [原可见定义装配](../../Kodex/agent-state/impl/state/src/commonMain/kotlin/io/github/stream29/kodex/agentstate/tool/KodexVisibleToolSpecs.kt)
  与 [原 pending projection](../../Kodex/agent-state/impl/state/src/commonMain/kotlin/io/github/stream29/kodex/agentstate/tool/PendingToolEventProjection.kt)
  连同两个测试文件并入 `impl/state`；原包名和函数名不变。
- 固定工具顺序、始终可见 Plan、AskUser 门控、当前 MCP 快照与 deferred search
  定义不变；Tool.handle/环境副作用仍归 tool 与 Runtime，不进入 State spec。
- malformed JSON/patch/search 输入仍投影 PendingInvalidToolCall；
  动态 function 与 MCP namespace 保持分类，保留 callId/itemId/原参数诊断。
- `ToolSearchDocument`/SourceInfo 已有真实 `tool/spec/tool-search` 所有者：
  合并后公开返回类型使用该 spec 的 api，转换/具体 search spec 构造用 impl
  implementation。不能再复制 DTO，或把 impl 依赖当 DTO 所有权。
- 合并两项目的依赖集合，删除旧内部 `:agent-state-tool` 边；
  `tool-impl-plan/get-context-remaining -> State spec` 保留，不能改指 State impl，
  否则 State impl 的工具定义装配会形成环。
- TestAgentContextSettings/TestMcpService 留在 `:agent-state-test`；
  固定 Shell 值仅需真实 shell spec，不以完整 shell impl 作为值模型依赖。
  测试假 MCP 的 no-op 是隔离外部服务，非生产未接入契约。

## Consumer closure

| 现有依赖 | 新依赖 / 关键消费者 |
| --- | --- |
| `:agent-state-contract` | `:agent-state-spec-contract`；Runtime spec/composition、RPC server/app RPC、History、History Index、plan/context tools。 |
| `:agent-state-context-window` | `:agent-state-spec-context-window`；compact Runtime 与 get-context-remaining。 |
| `:agent-state-impl` | `:agent-state-impl-state`；两种 Session、原 State/Runtime/tool/context tests、Integration。 |
| `:agent-state-tool` | `:agent-state-impl-state`；Runtime composition 复用 deferred projection 与 Runtime tool tests。 |
| `:agent-state-test` | 不改坐标；State/context/Runtime/tools/Session/Integration 测试装配。 |

- 验证真实 Session 工厂仍创建原 State、Runtime 委托同一实例、RPC 操作与
  History streaming 观测来源未改变。Koin module/API 编译也要随直接依赖闭包检查。
- Gradle settings 已递归 include 目录；新 build 文件与源移动直接产生新坐标。
  删除四个旧项目 build，不保留转发空项目；旧 generated build 仅在无使用者时清理。
- 当前文档中的 active 路径更新，原始迁移历史保留；对应 SOP 的旧 compact、
  turnId 和 plan-update 事务描述按主计划修正，不能把新计划写作已验收事实。

## Validation and handoff

- 先完整保留原 helper 行为断言；增加动态 MCP/AskUser 定义与实际请求
  组装一致、无效调用仍由原 Runtime 路径完成的真实装配检查。
- 主图无 `spec -> impl`；测试图可使用真实实现。合并后的 State→tool impl→
  State spec/context-window 是有效方向，不能为了“优化层级”删除实际契约依赖。
- 废除旧坐标、唯一声明、真实 factory+caller、原测试迁移、公开依赖暴露
  分别给证据；矩阵和平台限制见主计划。
- 若实际解析出现环或跨平台不可用，先报告真实依赖原因；不恢复旧项目或
  创建平行 State/tool 抽象凑编译。

## Implementation and validation evidence

- 原 context-window 与三项测试直接迁入 spec；tool 两个源码和两套测试
  逐文件字节对照一致，并入唯一 State 实现。旧 tool build 已删除，无空壳或别名。
- 全部实际消费者构建文件切到三个新生产坐标；Runtime tool 的重复测试边合并。
  `agent-state-test` 仅以实际 Shell spec 依赖值模型，不再依赖 Shell impl。
- 208 个当前构建文件的 project 引用均可解析，投影主源码图无环；
  两个 State spec 无直接 impl 边，四个旧 State 坐标无引用。
  静态图不包含测试 fixture 边，亦不替代 Gradle 实际解析。
- Xiaoxin Ubuntu 预算 JVM 三项和 State JVM 55 项通过；tool plan、
  get-context-remaining、Runtime decorator 与 Session/RPC/App 下游均已运行。
  主任务集中矩阵共 562 项，CLI/Integration 编译与四个 State Linux x64/JS 编译通过。
- 原 concrete helper 的合并只消融项目层，不改变工具顺序、MCP 投影、
  pending 解析或实际处理路由；当前 SOP 路径和基线语义已同步。
- 用户已验收并授权分批提交推送，本任务归档 done；
  其他平台限制与已有并发风险见主任务。

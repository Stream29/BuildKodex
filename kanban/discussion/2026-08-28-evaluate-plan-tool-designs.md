# Task Tree

- 研究 Plan 工具变体与长程任务评测
  - [done] 梳理 Kodex 现有 Plan 数据与调用链
  - [done] 对照 codex-rs 的协议与提示词
  - [done] 分析 Plan 作为上下文压缩状态的机制
  - [done] 设计可比较的 Plan 工具变体
  - [done] 设计长程任务、指标与实验矩阵
  - [done] 形成供用户审核的推荐方案
  - 等待用户确定首轮实验范围

# Details

- 当前阶段只进行研究与实验设计，不实现工具或 benchmark。
- IntelliJ IDEA 正在打开根项目。
- 根工作区已有未跟踪文件 `kanban/discussion/2026-08-28-plan-sqlite-storage-migration.md`，本任务不修改它。

## 现有机制

- `UpdatePlanArgs` 是全量替换快照，只包含可选说明和带三态状态的步骤列表，见 `Kodex/openai/models/src/commonMain/kotlin/io/github/stream29/kodex/openai/PlanToolModels.kt:17`。
- 工具 schema 声明“最多一个进行中步骤”，但 `strict = false`，handler 不校验步骤数量、状态组合、文本长度或状态转换，见 `Kodex/tool/plan/src/commonMain/kotlin/io/github/stream29/kodex/tool/plan/PlanTools.kt:8`。
- 每次普通 Responses 请求都重新注入约 500 词的固定 planning 指令，见 `Kodex/agent-state/impl/src/commonMain/kotlin/io/github/stream29/kodex/agentstate/impl/KodexAgentStateImpl.kt:168`。
- 成功调用会先把最新快照写入 `settings.plan`，再完成工具调用；两个写入不是同一个 AgentState 操作，见 `Kodex/agent-state/contract/src/commonMain/kotlin/io/github/stream29/kodex/agentstate/contract/KodexAgentStateExtensions.kt:24`。
- 完成事件会重新投影成原始 function call 和 `"Plan updated"` output，因而历次快照都进入后续模型输入，见 `Kodex/agent-storage/clean-models/src/commonMain/kotlin/io/github/stream29/kodex/agentstorage/cleanmodels/stable/StablePlanUpdate.kt:23`。
- `settings.plan` 当前没有投影回模型输入；除写入和测试外没有生产读取。模型看到的是历史中的 plan calls，不是单独注入的最新状态。
- Remote compaction 会在共享的 64K 预算内保留所有仍放得下的 Plan 更新，而不是只保留最新快照，见 `Kodex/agent-state/impl/src/commonMain/kotlin/io/github/stream29/kodex/agentstate/impl/RemoteCompactionV2.kt:14`。
- 相比 codex-rs，Kodex 已额外拥有最新 settings 快照和显式 compaction retention；目前没有利用前者消除旧快照。

## 机制判断

- Plan 的主要价值不是预测完整执行轨迹，而是诱导模型周期性生成一个低熵、结构化、持久化的工作状态摘要。
- 它同时提供任务分解、当前焦点、进度检查、用户可见协作和 compaction 锚点。
- 当前实现只是在语义上“压缩”；物理上下文会因为每次更新都追加全量快照而增长。
- completed/pending/in-progress 无法可靠保存约束、决策、发现、失败路径、产物位置、阻塞原因和验收证据。
- 旧快照并存会造成状态冲突；更新越频繁，Remote compaction 的保留预算消耗越大。
- 工具是否有效由四个可分离因素决定：状态表示、更新协议、模型输入投影、更新触发策略。实验不能把四者一次性全部改变。

## 候选条件

- `B0 NoPlan`：移除工具和专用 planning 指令，测量模型原生长程能力。
- `B1 PromptOnly`：保留等价规划要求，但不提供结构化持久工具，区分提示词收益与工具收益。
- `P1 AppendChecklist`：原样保留当前实现，作为生产基线。
- `P2 CanonicalChecklist`：保持 P1 的 schema 和调用方式，但模型输入只投影最新 Plan 快照；审计历史仍完整保存。
- `P3 HierarchicalPlan`：使用稳定节点 ID、父子关系、依赖、状态、验收条件和完成证据；模型输入只保留当前树和已折叠里程碑。
- `P4 TaskStateCapsule`：维护目标、约束、决策、发现、产物、当前焦点、下一步、阻塞和验证状态；它明确以“可恢复工作状态”而不是 TODO 列表为目标。
- 首轮只比较上述表示。胜出表示再做全量替换与 delta 更新、提示性规则与严格校验、模型自主更新与 harness checkpoint 更新三组消融。
- 所有结构设置确定性字段和节点预算，不使用额外 LLM 自动总结，避免引入第二个模型。
- 增加一组中性工具名复现实验，降低模型对 Codex `update_plan` 既有训练所造成的偏差。

## Benchmark 设计

- 用“必须经历的环境状态转换数”定义 horizon，不用输出 token 数替代任务长度。
- 每个任务提供机器可验证的最终测试、里程碑、约束和 oracle dependency graph。
- 核心采用离线、容器化、可生成的 terminal/repository 任务，避免网络变化和数据污染。
- 任务族覆盖跨模块迁移、分布式约束修改、带错误线索的故障定位、构建发布流水线、数据管线复现和多服务集成。
- 每个模板生成 `16/32/64/128` 个必要状态转换的配对实例；只改变 horizon，保持规则和难度结构一致。
- 压力条件覆盖固定次数强制 compaction、早期约束延迟验收、中途需求变化、无显式错误的路径阻断、干扰工具输出和 checkpoint 恢复。
- 在 25%、50%、75% 里程碑执行影子状态充分性探针，只向评估器提供目标和当前 Plan，检查约束、已完成工作、当前阻塞与合法下一步。
- checkpoint 恢复条件启动新模型上下文，只保留 workspace、原始任务和 canonical Plan，直接测量 Plan 是否足以承载长程状态。
- 外部效度轨道使用少量 SWE-bench Pro 和 Long-Horizon Terminal-Bench 任务；不把昂贵、异质的真实任务作为首轮工具筛选集。

## 指标

- 首要指标为确定性 `task success`、里程碑完成率、约束满足率和 checkpoint 恢复成功率。
- 诊断指标为遗忘约束率、重复工作率、过早结束率、故障后改计划延迟和 stale Plan 比例。
- 压缩指标为状态充分性探针召回率、canonical Plan token 数、被替代轨迹与 Plan 的 token 比和单位 Plan token 的里程碑收益。
- 工具指标为采用率、更新频率、无效调用率、拒绝率、状态转换违规和更新 churn。
- 效率指标为输入/输出 token、模型调用、工具调用、墙钟时间和估算成本。
- 结果以任务成功为主，Plan 质量评分只用于故障归因；不使用单一 LLM judge 代替隐藏测试。

## 实验方法

- 固定 Kodex scaffold、可见工具、Single Agent 模式、compaction 时点、token/时间预算和 provider 参数。
- 采用 task-instance 配对运行和多个重复样本；同一实例在所有工具条件下使用相同环境 seed。
- 模型按 Codex 原生模型、其他闭源 coding 模型、开放权重或较小模型分层，分析 `plan variant × model × horizon × compaction` 交互。
- 首轮建议 24 个生成实例、6 个条件、3 个代表模型、每格 2 次，共 864 次运行。
- 确认阶段仅保留 `NoPlan`、当前工具和首轮前两名；任务数、模型数和重复数依据首轮方差做 power analysis。
- 使用按任务分层的 bootstrap 置信区间和 mixed-effects 回归；同时报告成功率、成本与无效调用的 Pareto 前沿，不只报告总平均分。

## 推荐路线

- 先实现 benchmark instrumentation 和 `P2 CanonicalChecklist`，因为它改动最小，并直接检验“只保留最新状态是否就是主要收益来源”。
- 第二个原型实现 `P4 TaskStateCapsule`，它最直接检验用户提出的上下文压缩假设。
- `P3 HierarchicalPlan` 放在第三顺位；它更适合依赖密集任务，但 schema 成本和小模型调用失败风险更高。
- 首轮不要先做自动 planner、独立 planner 模型或多 Agent；这些会掩盖 Plan 工具本身的因果效果。
- 在确认 `P2` 或 `P4` 有稳定收益后，再决定是否引入 delta mutation、严格状态机和 harness 自动 checkpoint。

## 相关研究

- Plan-and-Act 把动态重规划视为携带长程相关上下文的机制，并报告计划可替代独立 memory module。
- UltraHorizon 说明标准设置也可超过 35K token 和 60 次工具调用，适合作为超长程设计参考。
- MemoryAgentBench 将准确检索、测试时学习、长程理解和冲突解决拆开，支持使用独立状态充分性诊断。
- SWE-bench Pro 提供真实的长程软件工程任务；Long-Horizon Terminal-Bench 提供 dense reward 和数百步 terminal 任务。

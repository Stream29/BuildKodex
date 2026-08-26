# Task Tree

- 固定 Subagent Fork 为最新 Compaction 活动窗口
  - [done] 收敛 `spawn_agent` 输入契约
  - [done] 保留 model/reasoning 继承与 override
  - [done] 隐藏并继承 service tier
  - [done] 固定 Fork 最新 Compaction 活动窗口
  - [done] 重定位六条 timeline 与 checkpoint
  - [done] 接入 `comp_hash` 兼容压缩
  - [done] 保持失败删除与运行调度语义
  - [done] 覆盖契约、Storage 与 Multi-Agent 回归
  - [done] 完成 JVM、Linux X64 与 IDEA 验证

# Details

- 状态：`done`。
- 本任务是[统一 Fork materialization 批次](2026-08-27-unify-fork-materialization-batch.md)的 Subagent 子任务。
- 本任务消费统一 Storage range、repository target materialization 与
  [filesystem fast path](2026-08-10-optimize-filesystem-session-fork.md)，不在 Tool handler 内建立第二套复制与恢复实现。

## 决策

- `spawn_agent` 不再向模型暴露历史继承选项。
- 每次 Spawn 固定继承父 Agent 在安全 boundary 前的最新活动 Compaction 窗口。
- 活动窗口包含：
  - 最新 `CleanCompactionCheckpoint`，包括 retained prefix 与 provider compaction payload。
  - 该 checkpoint 起至 boundary 前的六条 timeline change points。
- `model` 与 `reasoning_effort` 缺省时继承父 Agent，显式传入时 override。
- `service_tier` 不再向 Agent 暴露，child 固定继承父 Agent 的生效值。
- 用户已确认保留完整活动窗口，不采用会丢失 compaction payload 的清洁历史重建路线。
- 用户已确认 model override 跨越不兼容 `comp_hash` 时，先用 checkpoint 模型重压缩，再由 override 模型执行首轮任务。

## Agent 可见工具签名

- 当前：

  ```text
  spawn_agent({
    task_name: string,
    message: string,
    fork_turns?: string,
    model?: string,
    reasoning_effort?: string,
    service_tier?: string
  })
  ```

- 计划：

  ```text
  spawn_agent({
    task_name: string,
    message: string,
    model?: string,
    reasoning_effort?: string
  })
  ```

- 只有 `task_name` 与 `message` 必填；`additionalProperties=false` 和当前非 strict 工具声明保持不变。
- 成功输出保持 `{task_name: string, nickname: string | null}`。
- `model` 与 `reasoning_effort` 的说明明确“省略即继承父 Agent”；工具说明明确历史固定继承最新活动窗口。

## Storage 语义

- 继续复用
  `Kodex/tool/multi-agent/impl/src/commonMain/kotlin/io/github/stream29/kodex/tool/multiagent/MultiAgentToolFactory.kt:504-515`
  的 pending-call boundary：
  - 有 pending calls 时使用最早 pending index。
  - 否则使用 `latestIndex + 1`。
- 在 `boundary - 1` 上通过 compaction timeline 的 floor lookup 取得最新 checkpoint index `from`。
- 生成统一 Fork range `fromInclusive = from`、`untilExclusive = boundary`。
- 对 compaction、settings、timestamp、token-count、stable 和 unstable timeline：
  - 只复制 stored indexes `from <= index < boundary`。
  - 目标 index 为 `index - from`。
- 目标 checkpoint 位于 index `0`；统一 range copy 将每个 checkpoint 的 `historyBaseIndex` 同样减去 `from`。
- 保留 checkpoint 的 prefix、compaction payload、window number 和 window lineage。
- 保留活动窗口内的 settings、timestamps、token counts、stable events 和 unstable snapshots，维持窗口内 revert 与状态重建语义。
- Fork 后再追加 child settings，重置 `threadName`、`turnId`、`previousResponseId` 和 `promptCacheKey`；随后投递 `NEW_TASK`。
- child 生效 settings 由 boundary 前父 settings 派生：
  - `model = args.model ?: parent.model`。
  - `reasoning.effort = args.reasoningEffort ?: parent.reasoning.effort`。
  - `serviceTier = parent.serviceTier`，不读取 Agent 输入。
- 现有完整前缀 `KodexAgentStorage.forkTo` 继续服务 Session/History Fork，不改变其契约。
- `caller.subagents.createFork(sourceStorage, range)` 在 child runtime 打开前完成 raw target materialization；成功后才 open child、追加 settings 和调度。

## Model Override 与 Compaction

- provider compaction payload 是 opaque 数据，不能假定任意模型可复用。
- `/models` 的 `ModelInfo.comp_hash` 是 compaction-compatible 模型配置标识；当前 Kodex
  `ModelInfo` 尚未接收该字段：
  `Kodex/openai/models/src/commonMain/kotlin/io/github/stream29/kodex/openai/ModelCatalogModels.kt:16-43`。
- 上游只在前后两个非空 hash 不同时触发模型切换前压缩：
  `shared-context/codex/codex-rs/core/src/session/turn.rs:1043-1119`。
- Spawn 使用以下顺序：
  - 取活动 checkpoint index `from`；其 settings model 是 provider payload 的来源模型。
  - 计算 child 的最终生效 model，并从 model catalog 解析两侧 `compHash`。
  - checkpoint 没有 provider payload、hash 相同或任一 hash 缺失时，不增加过渡压缩。
  - 两个非空 hash 不同时，先以来源模型和独立 child identity 对 materialized 窗口执行
    `CompHashChanged`、`PreTurn` compaction。
  - 压缩成功后写入最终 model/reasoning settings，再投递 `NEW_TASK` 并首次 resume。
  - 过渡压缩失败时 Spawn 失败，由既有 target 删除补偿清理 child。
- `reasoning_effort` override 不单独触发该流程；兼容判断只使用 model catalog 的 `compHash`。

## 当前实现差异

- `fork_turns=all` 当前复制 boundary 前的全部六条 timeline：
  `Kodex/agent-storage/contract/src/commonMain/kotlin/io/github/stream29/kodex/agentstorage/contract/KodexAgentStorage.kt:177-200`。
- `Recent` 当前只把 checkpoint retained prefix 与 stable 后缀注入 child：
  `Kodex/tool/multi-agent/impl/src/commonMain/kotlin/io/github/stream29/kodex/tool/multiagent/MultiAgentToolFactory.kt:518-531`。
- `Recent` 没有转移 `CleanCompactionCheckpoint.compaction`，因此不能作为固定活动窗口实现；否则会丢失已 compact 的 assistant/tool 上下文。
- 新路线需要 Storage 层的 checkpoint-aware range Fork，而不是调用现有 `forkTo` 或把 `Recent` 设为固定大值。
- 当前 `create()` → `open()` → runtime 写入路线会提前建立空 target cache；统一 repository materialization 同时解决该问题与 filesystem JSON round-trip。

## 兼容性

- `OpenAiJsonCodec` 启用了 `ignoreUnknownKeys`：
  `Kodex/openai/json-codec/src/commonMain/kotlin/io/github/stream29/kodex/openai/jsoncodec/OpenAiJsonCodec.kt:7-10`。
- 删除字段后，既有 stable/unstable Session 记录中的 `fork_turns` 与 `service_tier`
  仍可读取，但解码后不再保留或重新输出。
- `model` 与 `reasoning_effort` 保留原字段名和 nullable 继承语义。
- 升级时已经 pending 的旧 Spawn 调用也按新的固定活动窗口语义执行；旧
  `none` 与 turn count 不再生效，旧 `service_tier` 被忽略，model/reasoning override
  继续生效。
- 不修改 filesystem 格式版本，也不迁移或重写已有 Session。
- History 详情不再显示旧 `Fork history` 或 `Service tier`；非空 model/reasoning
  override 与其他 Spawn 参数、结果保持可读。

## 空间效果

- 2026-08-27 对 Session `132` 根 Agent 的动态快照：
  - 当前全部六条 timeline 约 `1.996 GiB` allocated。
  - 最新 checkpoint 至当前 index 的同范围约 `96 MiB` allocated。
  - 单次根 Spawn 的该样本复制量下降约 `95%`。
- 同次扫描中，317 个 Agent 的 stable 全历史约 `36.99 GiB`，各自最新
  checkpoint 与 stable 后缀合计约 `0.519 GiB`，说明完整历史重复是主要放大项。
- 该方案不是绝对大小上限：
  - 尚未发生真实 compaction 时，初始 checkpoint 为 index `0`，仍会复制全部活动历史。
  - checkpoint 后出现超大工具结果时，当前活动窗口仍可能较大。
- 新行为只限制后续 Spawn，不自动回收既有 Subagent 数据。

## 预计修改范围

- 共享 Storage 与 Session materialization 由统一批次先完成：
  - `Kodex/agent-storage/contract/src/commonMain/kotlin/io/github/stream29/kodex/agentstorage/contract/IndexVersioning.kt`
  - `Kodex/agent-storage/contract/src/commonMain/kotlin/io/github/stream29/kodex/agentstorage/contract/KodexAgentStorage.kt`
  - `Kodex/agent-session/contract/src/commonMain/kotlin/io/github/stream29/kodex/agentsession/contract/KodexSession.kt`
  - in-memory/filesystem Session repository 与 filesystem storage 实现。
- Multi-Agent contract 与 schema：
  - `Kodex/tool/multi-agent/contract/src/commonMain/kotlin/io/github/stream29/kodex/tool/multiagent/MultiAgentModels.kt`
  - `Kodex/tool/multi-agent/impl/src/commonMain/kotlin/io/github/stream29/kodex/tool/multiagent/MultiAgentSchemas.kt`
  - `Kodex/tool/multi-agent/impl/src/commonMain/kotlin/io/github/stream29/kodex/tool/multiagent/MultiAgentTools.kt`
- Model catalog：
  - `Kodex/openai/models/src/commonMain/kotlin/io/github/stream29/kodex/openai/ModelCatalogModels.kt`
  - `Kodex/openai/model-catalog/impl/src/commonMain/kotlin/io/github/stream29/kodex/openai/modelcatalog/BuiltInModelCatalog.kt`
- Spawn 与 Storage：
  - `Kodex/tool/multi-agent/impl/src/commonMain/kotlin/io/github/stream29/kodex/tool/multiagent/MultiAgentToolFactory.kt`
  - `Kodex/agent-runtime/impl/src/commonMain/kotlin/io/github/stream29/kodex/agentruntime/impl/KodexAgentTools.kt`
- History：
  - `Kodex/app/view/history/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/history/CleanEventView.kt`
- 对应 contract、clean-model、in-memory/filesystem storage、AgentSession、History
  与 Multi-Agent 测试。
- 持久决策：
  - `checklist/agent-state-and-runtime.md`

## 验证

- 使用真实 in-memory 与 filesystem storage，不增加 mock。
- 检测当前 Gradle Daemon JVM，并向 Gradle 显式提供同一 JDK。
- 运行受影响模块的 `jvmTest` 与 `linuxX64Test`，至少覆盖：
  - `agent-storage-contract`
  - `agent-storage-in-memory`
  - `agent-storage-filesystem`
  - `agent-storage-clean-models`
  - `tool-multi-agent-contract`
  - `tool-multi-agent-impl`
  - `openai-models`
  - `openai-model-catalog-impl`
  - `agent-session-in-memory`
  - `agent-session-filesystem`
  - `app-view-history`
- 运行 IDEA build/inspection 和两层 `git diff --check`。
- 增加受控 filesystem fixture，比较完整前缀与活动窗口目标的文件数和字节数；
  测试结束删除 fixture。

## 排除项

- 不清理或迁移现有 Kodex Home。
- 不改变 Session Catalog/History 的完整 Fork。
- filesystem 优化只提供共享 materialization，不改变本任务的活动窗口产品语义。
- 不修改 context-limit compaction 阈值、checkpoint 内容或 storage 文件格式。
- `CompHashChanged` 过渡压缩只服务 model override 的 payload 兼容，不扩展为通用模型切换重构。
- 不实现共享前缀、hardlink、reflink、内容寻址或全局 GC。

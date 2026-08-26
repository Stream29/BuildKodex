# Task Tree

- 固定 Subagent Fork 为最新 Compaction 活动窗口
  - 收敛 `spawn_agent` 输入契约
    - 删除 `fork_turns` 字段、模式类型、serializer 和 schema
    - 删除 `model` 与 `reasoning_effort` override
    - 保留 `service_tier` override
    - 更新工具说明与 History 详情
  - 建立活动窗口 Storage Fork
    - 以当前 pending-call exclusive boundary 捕获父快照
    - 选择 boundary 前生效的最新 compaction checkpoint
    - 复制 checkpoint 至 boundary 前的六条 timeline 区间
    - 将源 index 区间按 checkpoint index 重定位到零
    - 同步重定位 checkpoint `historyBaseIndex`
    - 保留 checkpoint payload、window lineage 与区间状态
    - 保持完整历史 `forkTo` 行为不变
  - 接入固定 Spawn 流程
    - 删除 None、All 与 Recent 执行分支
    - 使用活动窗口 Fork 初始化 raw child
    - 追加独立 child settings 与 `NEW_TASK`
    - 保持失败删除与运行调度语义
  - 覆盖契约与持久化兼容
    - 验证旧 `fork_turns` JSON 可读但被忽略
    - 验证旧 model override JSON 可读但被忽略
    - 验证新 schema 与序列化不再输出删除字段
    - 验证旧 Session 无需迁移
  - 覆盖 Storage 与 Multi-Agent 回归
    - 验证初始 checkpoint 与真实 compaction checkpoint
    - 验证六条 timeline 的区间选择和 index 重定位
    - 验证 compaction payload 与活动模型历史等价
    - 验证当前 pending calls 不进入 child
    - 验证嵌套 Spawn 不重新复制 checkpoint 前历史
    - 验证 filesystem 目标不产生 checkpoint 前文件
    - 验证失败补偿、设置继承和任务投递
  - 更新 AgentState 与 Multi-Agent 维护决策
  - 运行受影响 JVM、Native 与 IDEA 验证
  - 用户确认完整计划后再进入 executable

# Details

- 状态：`planning`。本轮只完成调查与规划，不进入 `executable`，也不修改 `Kodex/`。

## 决策

- `spawn_agent` 不再向模型暴露历史继承选项。
- 每次 Spawn 固定继承父 Agent 在安全 boundary 前的最新活动 Compaction 窗口。
- 活动窗口包含：
  - 最新 `CleanCompactionCheckpoint`，包括 retained prefix 与 provider compaction payload。
  - 该 checkpoint 起至 boundary 前的六条 timeline change points。
- 子 Agent 固定继承父 Agent 的 model 与 reasoning effort；对应 override 一并从输入契约删除。
- `service_tier` override 保留。
- 用户已确认保留完整活动窗口，不采用会丢失 compaction payload 的清洁历史重建路线。

## Storage 语义

- 继续复用
  `Kodex/tool/multi-agent/impl/src/commonMain/kotlin/io/github/stream29/kodex/tool/multiagent/MultiAgentToolFactory.kt:504-515`
  的 pending-call boundary：
  - 有 pending calls 时使用最早 pending index。
  - 否则使用 `latestIndex + 1`。
- 在 `boundary - 1` 上通过 compaction timeline 的 floor lookup 取得最新 checkpoint index `from`。
- 对 compaction、settings、timestamp、token-count、stable 和 unstable timeline：
  - 只复制 stored indexes `from <= index < boundary`。
  - 目标 index 为 `index - from`。
- 目标 checkpoint 位于 index `0`；其 `historyBaseIndex` 同样减去 `from`。
- 保留 checkpoint 的 prefix、compaction payload、window number 和 window lineage。
- 保留活动窗口内的 settings、timestamps、token counts、stable events 和 unstable snapshots，维持窗口内 revert 与状态重建语义。
- Fork 后再追加 child settings，重置 `threadName`、`turnId`、`previousResponseId` 和 `promptCacheKey`；随后投递 `NEW_TASK`。
- 现有完整前缀 `KodexAgentStorage.forkTo` 继续服务 Session/History Fork，不改变其契约。

## 当前实现差异

- `fork_turns=all` 当前复制 boundary 前的全部六条 timeline：
  `Kodex/agent-storage/contract/src/commonMain/kotlin/io/github/stream29/kodex/agentstorage/contract/KodexAgentStorage.kt:177-200`。
- `Recent` 当前只把 checkpoint retained prefix 与 stable 后缀注入 child：
  `Kodex/tool/multi-agent/impl/src/commonMain/kotlin/io/github/stream29/kodex/tool/multiagent/MultiAgentToolFactory.kt:518-531`。
- `Recent` 没有转移 `CleanCompactionCheckpoint.compaction`，因此不能作为固定活动窗口实现；否则会丢失已 compact 的 assistant/tool 上下文。
- 新路线需要 Storage 层的 checkpoint-aware range Fork，而不是调用现有 `forkTo` 或把 `Recent` 设为固定大值。

## 兼容性

- `OpenAiJsonCodec` 启用了 `ignoreUnknownKeys`：
  `Kodex/openai/json-codec/src/commonMain/kotlin/io/github/stream29/kodex/openai/jsoncodec/OpenAiJsonCodec.kt:7-10`。
- 删除字段后，既有 stable/unstable Session 记录中的 `fork_turns`、`model` 和
  `reasoning_effort` 仍可读取，但解码后不再保留或重新输出。
- 升级时已经 pending 的旧 Spawn 调用也按新的固定活动窗口语义执行；旧
  `none`、turn count 和 model override 不再生效。
- 不修改 filesystem 格式版本，也不迁移或重写已有 Session。
- History 详情不再显示旧 `Fork history`、model 或 reasoning override；其他 Spawn
  参数与结果保持可读。

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

- Multi-Agent contract 与 schema：
  - `Kodex/tool/multi-agent/contract/src/commonMain/kotlin/io/github/stream29/kodex/tool/multiagent/MultiAgentModels.kt`
  - `Kodex/tool/multi-agent/impl/src/commonMain/kotlin/io/github/stream29/kodex/tool/multiagent/MultiAgentSchemas.kt`
  - `Kodex/tool/multi-agent/impl/src/commonMain/kotlin/io/github/stream29/kodex/tool/multiagent/MultiAgentTools.kt`
- Spawn 与 Storage：
  - `Kodex/tool/multi-agent/impl/src/commonMain/kotlin/io/github/stream29/kodex/tool/multiagent/MultiAgentToolFactory.kt`
  - `Kodex/agent-storage/contract/src/commonMain/kotlin/io/github/stream29/kodex/agentstorage/contract/IndexVersioning.kt`
  - `Kodex/agent-storage/contract/src/commonMain/kotlin/io/github/stream29/kodex/agentstorage/contract/KodexAgentStorage.kt`
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
  - `agent-session-in-memory`
  - `agent-session-filesystem`
  - `app-view-history`
- 运行 IDEA build/inspection 和两层 `git diff --check`。
- 增加受控 filesystem fixture，比较完整前缀与活动窗口目标的文件数和字节数；
  测试结束删除 fixture。

## 排除项

- 不清理或迁移现有 Kodex Home。
- 不改变 Session Catalog/History 的完整 Fork。
- 不推进
  [`filesystem session fork` 优化](../discussion/2026-08-10-optimize-filesystem-session-fork.md)。
- 不修改 compaction 触发阈值、checkpoint 内容或 storage 文件格式。
- 不实现共享前缀、hardlink、reflink、内容寻址或全局 GC。

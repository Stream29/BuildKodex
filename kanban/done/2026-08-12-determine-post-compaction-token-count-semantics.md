# Task Tree

- [done] 将 compaction 后的 token count 强制重置为 `0`
  - [done] 固化 compaction checkpoint 写入不变量
    - [done] 移除 checkpoint helper 的可选 token count 输入
    - [done] 在 checkpoint 事务的同一 index 写入 `0L`
    - [done] 停止读取 compaction response usage
  - [done] 对齐 token count 契约与维护决策
    - [done] 保留普通 response 的 provider-reported 语义
    - [done] 记录 compaction synthetic zero 的例外
    - [done] 记录无需迁移或回填既有 timeline
  - [done] 覆盖持久化与行为回归
    - [done] 验证有无 usage 均写入 `0`
    - [done] 验证强制 compact 后 resume 不会再次 compact
    - [done] 验证预算消费者将该值作为数值 `0`
  - [done] 运行定向 JVM 与 LinuxX64 验证
  - [done] 用户确认并授权进入 executable

# Details

- 状态：`done`。

## Problem

- 普通 Responses 仅在 `response.completed` 携带 usage 时写入新计数：
  `Kodex/agent-state/impl/src/commonMain/kotlin/io/github/stream29/kodex/agentstate/impl/KodexAgentStateImpl.kt:212-218`。
- Compaction 将 `completedResponse?.usage?.totalTokens` 传给 checkpoint helper：
  `Kodex/agent-state/impl/src/commonMain/kotlin/io/github/stream29/kodex/agentstate/impl/KodexAgentStateImpl.kt:293-300`。
- 该值为 `null` 时，helper 不写 token-count timeline：
  `Kodex/agent-storage/contract/src/commonMain/kotlin/io/github/stream29/kodex/agentstorage/contract/CompactionStorage.kt:22-49`。
- 稀疏 timeline 随后继续返回 compact 前的高计数。自动压缩和
  `get_context_remaining` 都读取这份状态：
  `Kodex/agent-state/context-window/src/commonMain/kotlin/io/github/stream29/kodex/agentstate/contextwindow/ContextWindowTokenBudget.kt:15-40`。
- 因此强制 compact 后立即 resume 仍可能再次触发自动 compact。

## Decisions

- 每个成功提交的 compaction checkpoint 都在同一 index 写入
  `tokenCount = 0L`。
- 该规则覆盖 manual、auto、pre-turn、mid-turn 和 standalone compaction。
- Compaction response 是否携带 usage、以及 usage 的具体值，均不改变结果。
- 在 `appendCompactionCheckpoint` 持久化边界强制该规则，并删除
  `tokenCount` 参数及 nullable 分支。
- 普通 Responses 的 token-count 写入保持不变。
- `0` 是可被预算、工具和 UI 直接消费的数值，不表示 unknown。
- 这是有意写入的 synthetic low watermark，不是服务端报告的 post-compaction
  active-context 大小。
- 接受的副作用：
  - UI 会暂时显示 `0t`。
  - `get_context_remaining` 会暂时返回完整阈值预算。
  - 下一次普通 response 报告 usage 前，自动压缩可能低估真实上下文。
- 下一次普通 response 报告 usage 后，以新的 timeline change point 恢复实际计数。
- 不回填既有 session。既有异常 session 在下一次成功 compact 后获得新值。
- Compaction 失败或取消时不提交 checkpoint，也不写入 `0`。

## Implementation

- `Kodex/agent-storage/contract/src/commonMain/kotlin/io/github/stream29/kodex/agentstorage/contract/CompactionStorage.kt`
  - 删除 `tokenCount` 参数。
  - 在 checkpoint compound write 中无条件嵌套
    `tokenCount.setWithTransaction(index, 0L)`。
  - 更新 KDoc，明确 synthetic reset 语义。
- `Kodex/agent-state/impl/src/commonMain/kotlin/io/github/stream29/kodex/agentstate/impl/KodexAgentStateImpl.kt`
  - 删除 compaction usage 到 storage helper 的投影。
  - 不改变普通 Responses 的 usage 写入。
- 对齐受影响的契约说明：
  - `Kodex/agent-storage/contract/src/commonMain/kotlin/io/github/stream29/kodex/agentstorage/contract/KodexAgentStorage.kt`
  - `Kodex/agent-state/contract/src/commonMain/kotlin/io/github/stream29/kodex/agentstate/contract/KodexAgentState.kt`
  - `Kodex/agent-state/context-window/src/commonMain/kotlin/io/github/stream29/kodex/agentstate/contextwindow/ContextWindowTokenBudget.kt`
- 更新持久决策：
  - `checklist/model-catalog.md`
  - `checklist/agent-state-and-runtime.md`
  - `shared-context/findings/agent-storage-compensation-semantics.md`

## Tests

- 更新 `InMemoryKodexAgentStorageTest`，证明 checkpoint helper 总是在同一
  index 写入 `0L`。
- 更新 `KodexAgentStateImplTest`：
  - compaction response 携带非零 usage 时仍写入 `0L`。
  - compaction response 缺少 usage 时也写入 `0L`。
- 在 `KodexAgentCompactionRuntimeTest` 增加回归：
  - 先以达到阈值的旧计数执行 manual compact。
  - 随后调用 `resume()`。
  - 断言只发生首次 manual compact，不会在 pre-turn 再 compact。
- 在 `ContextWindowTokenBudgetTest` 断言 timeline 中的 `0L` 产生完整剩余预算，
  而不是 unknown。

## Validation

- 先检测当前 Gradle Daemon JVM，并将同一 JDK 显式提供给 Gradle。
- 运行：
  - `:agent-storage-in-memory:jvmTest`
  - `:agent-state-impl:jvmTest`
  - `:agent-state-context-window:jvmTest`
  - `:agent-runtime-decorator-compact:jvmTest`
  - 对应四个模块的 `linuxX64Test`
- 运行 `git diff --check`。
- 无需真实 OpenAI 请求；本任务有意忽略 compaction usage，确定性测试足以验收。

## Result

- `appendCompactionCheckpoint` 已移除可选 token count 参数，并在每个成功
  checkpoint 的同一 index 写入 synthetic `0L`。
- Compaction response 的 usage 不再参与 token-count timeline；普通 Responses
  的 provider-reported 写入保持不变。
- 契约、维护决策和 storage compensation finding 已同步。
- 使用 Gradle daemon 的 `/home/stream/.jdks/openjdk-26.0.2` 验证：
  - 四个目标模块的 `jvmTest`：passed。
  - 四个目标模块的 `linuxX64Test`：passed。
  - root 与 `Kodex/` 的 `git diff --check`：passed。
- 未执行真实 OpenAI 请求；该验证对本任务不适用。

## Excluded

- 计算真实 post-compaction token count。
- 为 timeline 增加 unknown 或 invalidation 数据类型。
- 本地 tokenization 或上游 `recompute_token_usage()` 对齐。
- 修改自动压缩阈值、UI 展示或 remote compaction 传输终态。
- 迁移或批量修复既有 session 数据。

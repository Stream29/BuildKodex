# Task Tree

- 确定 compaction 后的 token count 语义
  - [done] 核对 compaction、token timeline、预算与 UI 的现有路径
  - 确定 post-compaction token count 的权威来源
  - 确定服务端未报告新计数时的 unknown 语义
  - 确定自动压缩与 `get_context_remaining` 的消费行为
  - 确定持久化表示、兼容处理与验证范围

# Details

- 状态：`await discussion`。
- 当前不进入规划或实现。
- 普通 Responses 请求在 `response.completed` 携带 usage 时，将 `usage.totalTokens` 写入 token timeline：`Kodex/agent-state/impl/src/commonMain/kotlin/io/github/stream29/kodex/agentstate/impl/KodexAgentStateImpl.kt:212-218`。
- Compaction 只尝试写入 `completedResponse?.usage?.totalTokens`：`Kodex/agent-state/impl/src/commonMain/kotlin/io/github/stream29/kodex/agentstate/impl/KodexAgentStateImpl.kt:293-298`。
- Remote compaction 当前允许只收到 compaction output、没有 `response.completed`；此时 `completedResponse` 为 `null`：`Kodex/openai/client/src/commonMain/kotlin/io/github/stream29/kodex/openai/client/OpenAiClient.kt:305-352`。
- `tokenCount == null` 时，compaction checkpoint 不写 token timeline：`Kodex/agent-storage/contract/src/commonMain/kotlin/io/github/stream29/kodex/agentstorage/contract/CompactionStorage.kt:43-49`。
- Token timeline 是稀疏状态；没有新 change point 时，新 snapshot 继续读取 compact 前的旧值：`Kodex/agent-storage/contract/src/commonMain/kotlin/io/github/stream29/kodex/agentstorage/contract/IndexVersioning.kt:13-17`。
- CLI、自动压缩和 `get_context_remaining` 因而都会继续消费旧计数，直到后续普通 response 报告新 usage。
- Compaction response 的 usage 描述压缩请求本身，不足以直接证明安装 compacted history 后的 active context 大小。当前本地 Codex 上游在替换 history 后另行调用 `recompute_token_usage()`：`shared-context/codex/codex-rs/core/src/compact_remote_v2.rs:288-333`。
- 现有项目决策要求只根据 OpenAI 实际报告的 token count 计算剩余上下文，缺失值保持 unknown，不做本地估算：`checklist/model-catalog.md:6`。
- 当前 `IndexVersioned<Long>` 没有在 compaction 边界发布 unknown/invalidation 的表示，因此实际行为与“缺失值保持 unknown”存在缺口。

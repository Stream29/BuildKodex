# Ultrafast 契约与 Kodex 缺口 — 2026-10-10

## 调研覆盖

- 调研日期：2026-10-10 UTC，02:04 UTC 前后完成资料核对。本文是选择性摘要，不是官方文档副本。
- Kodex 基线：`5b41be52d2f64a12ea3671bcecb9151fd2d1fa60`；只读审查，没有运行构建或真实模型请求。
- 本地 Codex 参考：`604061ce51d194a3aa6aad3b3170240d096e1725`。另读取上游 `rust-v0.162.0` 的 client、模型协议和 bundled catalog，未更新参考子模块。
- [Ultrafast 任务](../../kanban/executable/2026-10-09-support-openai-ultrafast.md)记录方案与分阶段 spec/impl 审阅门槛；本文件只保存调查时的事实和推断，不代表分支变更后的状态。

## 官方资料：订阅与 API 必须分开

- [Work and Codex](https://help.openai.com/en/articles/20001275-chatgpt-work-and-codex)列出 GPT-6 Astra / GPT-6.1 Sol 的 Ultrafast；订阅访问与套餐、workspace 权限有关，API-key 计费另算。模型名称本身不证明账号权限。
- [Codex Speed](https://learn.chatgpt.com/docs/agent-configuration/speed)同样区分订阅与 API 计费，并提示 Ultrafast 使用量显著增加。实现应提示 higher usage，不把套餐价格、倍率或速度宣传值固化成协议判定。
- [Ultrafast API guide](https://developers.openai.com/api/docs/guides/ultrafast-mode)当前描述两种模型和 `service_tier="ultrafast"`；建议 WebSocket，但没有把 WebSocket 写成接入必需条件。
- [Fast mode FAQ](https://help.openai.com/en/articles/11647665-fast-mode-faq)仍描述 alpha API 要求额外 `OpenAI-Service-Tier: ultrafast`，其他档位不能使用此 header。它与 API guide 的覆盖不完全一致；不能据此认定该 header 对 ChatGPT Codex backend 也必需。
- 结论：API 文档只能说明 API 路径；本次订阅实现以 Codex backend 源码路径为依据。未来新增 API-key 路径时，需要重新解决上述文档差异。

## 上游固定版本源码

- [client.rs](https://github.com/openai/codex/blob/rust-v0.162.0/codex-rs/core/src/client.rs)的 Codex backend SSE 路径把已解析档位写入 request，同时生成 `x-codex-routing-hint`：有档位时为 `model=<model>;tier=<tier>`。所审 client 文件没有 `OpenAI-Service-Tier` 的字面使用；这不是所有外围配置的穷尽证明。
- [openai_models.rs](https://github.com/openai/codex/blob/rust-v0.162.0/codex-rs/protocol/src/openai_models.rs)按 `service_tiers` 判断非标准档位支持；Flex 有特别规则，Default 被过滤。上游可能省略不支持的请求档位，不能直接复制为 Kodex 的静默降级策略。
- [bundled models.json](https://github.com/openai/codex/blob/rust-v0.162.0/codex-rs/models-manager/models.json)中 Astra / Sol 6.1 的静态 `service_tiers` 只有 `priority`。因此静态目录不等于账号实时能力，不能声称上游静态文件已提供 Ultrafast entitlement。
- 推断：现有 Kodex 订阅 SSE 可以增加 Ultrafast body 值与 Codex routing hint，无需同时引入 WebSocket。这个推断尚未通过真实有权限账号验证。

下载的三个源码文件 SHA-256：

| 上游路径 | SHA-256 |
| --- | --- |
| `core/src/client.rs` | `6ca16eb5fb185192738d321c22eacf74664e0ae2c107176c1c8a7f23fde33089` |
| `protocol/src/openai_models.rs` | `88d0f8df840078761b5a480ac21011ea097a55ffad6b0b8ad3742af215417d2d` |
| `models-manager/models.json` | `943ca7fe1d19ed019054158303f0dbbd80b3bef43b3aa425a71aa7cb3fb52c2b` |

## Kodex 当前实际链路

- 认证只接受 ChatGPT / ChatGPTAuthTokens，API-key 模式返回 UnsupportedAuthMode：[SubscriptionAuth.kt:56–78](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/server/src/commonMain/kotlin/io/github/stream29/kodex/cli/auth/SubscriptionAuth.kt#L56-L78)。
- 默认 endpoint 是 ChatGPT Codex backend，兼容 client version 为 `0.159.2`：[OpenAiClientConfig.kt:3–39](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/impl/client/src/commonMain/kotlin/io/github/stream29/kodex/openai/client/OpenAiClientConfig.kt#L3-L39)。可配置 baseUrl 不代表已实现 API-key 产品支持。
- ServiceTier 只有 Default/Fast/Flex，Fast 序列化及请求值仍是 `priority`：[OpenAiCommonModels.kt:20–38](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/spec/models/src/commonMain/kotlin/io/github/stream29/kodex/openai/OpenAiCommonModels.kt#L20-L38)。
- Backend YAML 另有显式 tier decoder，接受 `fast` / `priority` 别名；仅添加枚举不足以支持 YAML：[SplitSettingsFiles.kt:127–131](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/settings/impl/filesystem/src/commonMain/kotlin/io/github/stream29/kodex/cli/settings/SplitSettingsFiles.kt#L127-L131)。
- 普通生成与 remote compaction 都 POST `/responses`，两者已有 serviceTier body，但都未生成 routing hint：[OpenAiClient.kt:218–279](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/impl/client/src/commonMain/kotlin/io/github/stream29/kodex/openai/client/OpenAiClient.kt#L218-L279)。
- runtime 菜单按 `ModelInfo.availableServiceTiers()` 展示；Current session / New session defaults 页面直接枚举所有 tier。存在四处 exhaustive tier 标签分支，含 Suggest subagent task，不能只改状态栏。
- 内置目录只提供 Fast；远端成功时完整替换当前模型目录，失败保留旧快照：[OpenAiModelCatalogStoreImpl.kt:18–55](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/impl/model-catalog/src/commonMain/kotlin/io/github/stream29/kodex/openai/modelcatalog/OpenAiModelCatalogStoreImpl.kt#L18-L55)。现有 ModelServiceTier 已有 id/name/description，无需复制能力 DTO。
- 新 Session 和建议子 Session 沿用实际配置中的 tier；自动标题生成不传 tier，使用 client 的 Default。不得因本次接入把自动标题也改成 Ultrafast。

## 错误与实际档位的边界

- HTTP retry 只将 408/429/5xx 等现有条件视为可重试；400/403 等不重试。`response.failed` 在 AgentState 中统一产生 Retryable，后续有界重试耗尽才失败，不能声称它已具有 entitlement 错误的专用分类。
- 请求值与实际响应值已经分别进入 TokenCountDiagnostics：[KodexAgentStateImpl.kt:581–609](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-state/impl/state/src/commonMain/kotlin/io/github/stream29/kodex/agentstate/impl/KodexAgentStateImpl.kt#L581-L609)；实际 tier 是可空字符串，可保留未知值。
- diagnostics 只随可记录 usage 采集。缺 usage、请求失败或取消不保证有记录；不能伪造 token count 来补服务档位日志。
- 前端 binding 已接收完整 TokenCountSnapshot，但 AgentViewModel 当前只暴露 totalTokens：[RpcAgentViewModel.kt:84–85](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/rpc/src/commonMain/kotlin/io/github/stream29/kodex/cli/rpc/RpcAgentViewModel.kt#L84-L85)。
- 活跃输出的 RPC flow 不是每请求完成日志，Started 不可订阅，中间状态可 conflation：[AgentRuntimeRpc.kt:54–79](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/spec/contract/src/commonMain/kotlin/io/github/stream29/kodex/rpc/contract/AgentRuntimeRpc.kt#L54-L79)。因此不能用旁订阅承诺所有响应都能展示实际档位。
- 可复用现有本地 AgentNotification 的 Warning 展示已采集结果，不增加远端 Notification 分支。若要求所有请求都可靠展示实际 tier，需要另行设计结果契约。

## 实施前必须处理的门槛

- 新增 `ServiceTier.Ultrafast` 会扩展复用领域类型的跨线取值集合，属于 [RPC 契约冻结](../../checklist/rpc-architecture.md#rpc-契约冻结)范围，需要用户明确批准此具体扩展。
- 当前静态审查足以规划模型、YAML、UI、request 和 fixture 改动；不证明真实账号有权限、SSE 实际速度或 remote compaction 接受 Ultrafast。
- 不为本任务刷新参考子模块、升级 client version、改 Gradle 或修改历史 Home migrations。

# Task Tree

- `核对账户统计页与 Responses 元数据差异`()
- **`等待用户继续讨论字段语义与补齐范围`()**
- `确认普通任务、标题请求及会话关联的协议方案`()
- `等待用户授权进入 planning`()

# Details

## 已确认范围

- 用户要求调查 [ChatGPT Analytics](https://chatgpt.com/settings/usage?tab=analytics) 中未正确识别的 Kodex 使用数据，随后明确只关注 Responses API 补齐，不做独立事件遥测上传。
- 当前仅创建 discussion 任务；未授权进入 planning、实现或修改 RPC、存储契约。
- 补齐范围限于 `/responses` 请求及 `client_metadata["x-codex-turn-metadata"]` 的真实身份、用途归因；不新增 `analytics-events` 客户端或会话、轮次、技能、插件、工具事件上传。
- 不以 Messages、Skills、Plugins 活动计数完整为验收目标，也不因这些计数仍然缺失而扩大范围。

## 已核实证据

- 2026-10-13 查看账户页，所选范围为 10 月 7–13 日；Usage history 显示 Unknown 99.4%。对应只读接口中，大量记录已有 `surface=cli` 和模型信息，但 `thread_source`、`turn_trigger` 为 `unknown`。不能把用途 Unknown 等同于客户端或模型未识别。
- Kodex 普通请求已传 thread、turn、window 等身份，但元数据模型没有 `thread_source`、`turn_trigger`；官方将这两个字段写入 turn metadata。现有 `request_kind=turn/compaction` 表示请求类型，不替代用途归因。
- 自动标题请求没有传入会话身份或标题来源标记；其消费无法可靠归入 Task titles。页面 Top chats 的 Untitled／Model、Effort、Speed 不可用已观察到，但尚未证明这些条目对应 Kodex 会话。
- Plan usage history 接口明确报告历史覆盖与账目不完整；不能承诺客户端元数据补齐会修复该页面的 Not available。

## 待讨论

- 普通用户任务的 `thread_source`、`turn_trigger` 取值与写入时机；按实际来源区分用户输入、自动流程等，不统一伪装为用户轮次。
- 标题请求如何标记 `thread_title` 用途并关联原 Session，同时保持其独立请求语义。
- installation／session／thread／turn／window 身份的实际传值与生命周期；保留现有稳定身份，不为归因重建或批量改写历史。
- `parent_thread_id`、`parent_turn_id`、`root_turn_id` 等关系字段是否适用；只对真实关系填写，不把独立 Session 自动认定为官方子 agent。
- 其他官方元数据按必要性单独判断，不默认增加工作目录、Git remote、commit 或工具调用清单的上传；不伪报 `analytics_enabled=true`，不把 `history_ingest_requested` 当成通用标题同步开关。
- 候选检查：JSON 序列化与缺失值语义、普通／压缩／标题请求的身份和用途、HTTP 实际载荷，以及上线后新增消费的用途分类。服务端聚合规则和历史回填效果尚未验证。

## 参考

- [Kodex 元数据模型：26–70 行](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/openai/spec/models/src/specMain/kotlin/io/github/stream29/kodex/openai/CodexResponsesMetadataModels.kt#L26-L70)。
- [Kodex 普通请求传值：194–217 行](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/agent-state/impl/state/src/commonMain/kotlin/io/github/stream29/kodex/agentstate/impl/KodexAgentStateImpl.kt#L194-L217)。
- [Kodex 标题请求：31–54 行](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/rpc/impl/server/src/commonMain/kotlin/io/github/stream29/kodex/cli/sessiontitle/OpenAiSessionTitleGenerator.kt#L31-L54)。
- 官方本地参考基线为 `604061ce51`：[turn metadata 投影：417–448 行](file:///home/stream/ACodeSpace/push/BuildKodex/shared-context/codex/codex-rs/core/src/responses_metadata.rs#L417-L448)、[analytics／history 标记来源：739–749 行](file:///home/stream/ACodeSpace/push/BuildKodex/shared-context/codex/codex-rs/core/src/session/session.rs#L739-L749)。
- 与[现有 Codex 对齐讨论](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/discussion/2026-09-30-align-kodex-with-codex-rs.md#L1)相关，本任务单独限定 Responses 用途归因范围。

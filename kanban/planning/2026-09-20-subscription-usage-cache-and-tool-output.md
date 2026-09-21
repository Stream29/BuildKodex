# Task Tree

- 调查订阅额度快速消耗，修复缓存路由并补齐用量观测
  - [done] 对照本地 Codex Rust 的请求与工具输出链路
  - [done] 使用本地凭据实测 turn-state 与缓存明细
  - [done] 记录实测结果、适用范围和未确认事项
  - [done] 使用真实 Session 历史重放更长上下文，核查 turn-state 对照
  - [done] 记录长上下文实测并清理临时文件
  - [done] 确认主分支及缓存修复、明细与诊断范围
  - [done] 对照 Rust 说明建模并比较两条时间线的归属
  - [done] 确认 turn-state 持久化在 Session settings
  - [done] 核查迁移入口与实现阻塞点
  - [done] 确认现有轮次推断、历史恢复及迁移保证
  - [done] 制定数据结构、改动顺序与验收方案
  - [done] 按用户反馈移除额外身份校验及强制逐请求记录
  - [done] 审查 fork 标识并确认只修改当前快照
  - [done] 实现并验证持久路由状态与响应头链路
  - [done] 实现并验证用量快照和各消费端投影
  - [done] 增加版本冻结迁移、fixtures 与升级测试
  - [done] 完成隔离环境回归及发布准入核对

# Details

- 用户报告：2026-09-20 凌晨约一小时消耗超过 20% 额度。
- 用户最初要求：使用本地凭据验证 `x-codex-turn-state`，将调查保留在 discussion；最新实施范围与待确认设计见下节。
- 调查对照版本：Codex Rust `5ecb3afd1b`（2026-09-07）；实施前 Kodex `92b572d4` / 0.4.4，当前源码版本升至 0.4.5。
- 源码差异：
  - Rust 接收 turn-state，并在同一轮后续请求回传、新轮次清空；Kodex 尚无对应链路。这是路由协议差异，不等于已证实缓存失效。
  - Rust 有 MCP 输出限长、shell 模型预算约束和历史层统一截断；Kodex 不完整。
  - Rust 持久化初始上下文并追加变化；Kodex 每次重建上下文前缀。尚未证明本次发生频繁前缀变化。
  - Kodex 未保留缓存命中和缓存写入 token 明细，无法从历史 total_tokens 还原命中率。
- 既有离线核查：
  - 01:30—02:22（UTC+8），调查开始前，本机 7 个会话共 393 次模型请求、2 次响应重试。
  - Session 366 两次大快照含约 67.3 / 70.0 KB 原始 UTF-8 文本，超过 Rust 默认 MCP 约 48 KB 预算；节点 UID 归一化后第二份约 95.7% 文本重复。
  - 38 次 web.run 累计 877,242 字符，但单次最大 38,525 字节，均低于 Rust 默认历史兜底预算；不能将其累计量直接认定为相对 Rust 的异常放大。
- 实测约束：首轮为合成输入，追加轮为用户授权的真实历史重放；凭据和原始 turn-state 不写入输出或文档；不自动重试、不购买额度或重置限额。
- 证据路径（相对仓库根）：
  - `shared-context/codex/codex-rs/core/src/client.rs:270–292`：turn-state 生命周期。
  - `shared-context/codex/codex-rs/core/tests/suite/turn_state.rs:47–67`：HTTP 回传与新轮次清空测试。
  - `shared-context/codex/codex-rs/core/src/context_manager/history.rs:335–359`：历史统一截断。
  - `Kodex/openai/client/src/commonMain/kotlin/io/github/stream29/kodex/openai/client/OpenAiClient.kt:202–218`：当前请求头构造。
  - `Kodex/openai/models/src/commonMain/kotlin/io/github/stream29/kodex/openai/ResponsesApiModels.kt:510–518`：当前用量模型。

## 2026-09-20 实测

- 时间：03:18:14—03:22:00（UTC+8）。
- 使用本机 `~/.kodex/auth.yml` 中的现有凭据；未刷新或改写认证文件。
- 路径：`POST https://chatgpt.com/backend-api/codex/responses`，HTTP SSE；使用 `uv run --no-project python` 和 urllib，不是直接运行 Rust/Kodex 客户端。
- 模型 `gpt-6-astra`；为控制消耗，推理强度设为 `low`；请求 `service_tier=priority`，所有成功响应均报告 `service_tier=default`。不能把请求档位直接当实际 Fast 档。
- 每组独立 thread/turn/window/cache key，组内稳定；输入是 256 行合成表格，约 4.3K 输入 token。每组顺序执行两次无外部动作的 `cache_probe` 工具调用，再输出 `OK`。
- A/B 使用 Kodex-style 常规 Responses：顶层 tools、`include=[]`、不指定 reasoning.context、初始消息不带 ID。
- C/D 使用 Rust-style 对照：`x-openai-internal-codex-responses-lite=true`、稳定消息及工具结果 ID、`additional_tools`、`reasoning.context=all_turns`、`include=["reasoning.encrypted_content"]`；不是完整 Rust 运行时等价测试。
- B/C 从首响应取 turn-state，后续原样回传；A/D 始终省略。另检查 SSE `response.metadata`，没有观察到通过该事件下发状态。

| 组 | 路径 / turn-state | 三次 input_tokens | 三次 cached_tokens |
| --- | --- | --- | --- |
| A | Kodex-style / 不回传 | 4319 / 4353 / 4387 | 0 / 0 / 0 |
| B | Kodex-style / 回传 | 4328 / 4362 / 4396 | 0 / 0 / 0 |
| C | Rust-style lite / 回传 | 4332 / 4366 / 4400 | 0 / 4096 / 0 |
| D | Rust-style lite / 不回传 | 4324 / 4358 / 4392 | 0 / 0 / 0 |

- A/D 每次响应均带 turn-state；B/C 仅首响应携带，后续回传后不再重复下发。已确认该头在当前账号/端点实际使用，但未直接观察到后端机器或路由。
- 所有成功请求的 `cache_write_tokens=0`、`reasoning_tokens=0`；不能因前者为零就推断从未发生缓存写入，因为 C2 确有命中。
- C2 命中率 `4096 / 4366 ≈ 93.8%`；C3 为零，未形成稳定命中。
- 共 12 次成功请求：输入 52,317、输出 164、合计 52,481 token；其中缓存命中 4,096。total_tokens 含缓存部分，不代表未缓存计费量或额度百分比。
- 初次 lite 对照漏传 `reasoning.context=all_turns`，收到 HTTP 400 `unsupported_value`，未完成模型响应；核对 Rust `client.rs:870–887` 后修正并新建 C 组。合计请求尝试 13 次，无自动重试。
- 账户整数额度快照：A/B 前后 39%→40%，参数失败组 41%→41%，C/D 各自 42%→42%。期间未隔离其他会话、调查自身及其他账号活动，不能据此计算本测试扣额。
- 可供后续核对的 C 组 `x-oai-request-id`（非凭据）：
  - C1：`ea977aff-f4ce-44b6-bf3c-55a353b6dff3`
  - C2：`7111155d-e69d-4c9b-8743-3be3cb7285a0`
  - C3：`587fc7a8-899c-44f1-914f-741b33c33136`

## 判断与待讨论事项

- 已证实：服务端实际下发 turn-state，Kodex 缺少 Rust 所实现的回传链路。
- 未证实：仅补 turn-state 就能稳定改善 Kodex-style 请求缓存。合成 A/B 均零命中；下述真实历史长样本回传组第三次出现单次命中。
- C2 说明该账号/端点在本测试中能报告非零缓存命中；C/D 样本太少，且组间前缀、时间不同，不能据一次命中证明 turn-state 的因果效果。
- Rust-style 对照同时改变 lite、ID、工具布局、reasoning.context、include，不能把 C2 命中单独归因于其中任何一项。
- 本次使用 low、约 4.3K 合成输入、HTTP SSE，且实际返回 default；不等价于今晚 high/xhigh、10万级上下文或 Rust WebSocket 场景。
- 暂不把“缺少 turn-state 导致今晚超过 20% 扣额”记录为根因，也不把正常 total_tokens 增长当作缓存失效证据。
- 用户已决定推进缓存路由修复与用量观测，并确认 turn-state 放在 Session settings；尚未修改实现。工具输出保护不在本轮实施范围。

## 主分支修复范围与待确认设计

- 两个仓库均已在 main，代码工作区干净；本轮开始时仅本调查文档未跟踪。
- 用户明确要求修复 header，并确认 token-count 记录“明细＋诊断”：input/output/cached/reasoning、响应标识、实际模型/档位及路由诊断。
- 用户已明确 turn-state 必须跨内存持久化，排除仅内存方案，并确认放在 Session settings timeline。
- 本地 Rust `5ecb3afd1b`：`ModelClientSession.turn_state: Arc<OnceLock<String>>`，每次 new_session 新建空值；收到首个值后固定，同一 turn 回传，新 turn 不复用。HTTP 使用响应头/请求头；WebSocket 也可经 response.metadata 接收、client_metadata 回传。
- 已确认分工：原值以会话 settings 时间线为唯一持久真源；token-count 尽力保存可解读的用量与收发诊断，不用它反向恢复有效路由状态。不是全局 Home settings，也不是用户可编辑/继承的新会话默认项。规范见 [Codex Turn State](../../checklist/codex-turn-state.md)。
- token-count 已选择显式迁移：旧整数转换为 legacy 对象，运行时只读新结构；缺失明细不伪装为零，预算仍读取上下文快照。规范见 [Token-count Timeline](../../checklist/token-count-timeline.md)。
- 路线已确定并已在主分支实施；本轮未运行新增真实账号请求，也未迁移真实 Home。

### 本轮已确认的边界

- 保留当前 `startsNewTurnBefore` / `startsNewTurn` 推断，不引入 `markNewTurn()`；中断后提交不一定产生新 turn 的既有行为不在本任务修复范围。已纠正 checklist 与 main 的冲突。
- 同 Session revert 忠实恢复历史 settings 快照，不追加身份校验；fork 发布新当前 settings，生成新 turnId 并清空 turn-state，threadId 由新 storage URI 派生。不能以“进程重建”本身为理由丢弃持久值。
- 用户明确选择 fork 只修改当前快照：不批量重写继承的 settings、不限制 revert；以后 revert 到 fork 新快照之前，可以恢复源历史 turnId/turn-state。这是接受的快照语义，不再作为待补的身份校验漏洞。
- 用户明确不要新增账号/端点/thread 绑定校验：作为 settings 字段直接回传。token-count 一直是尽力记录，不要求每次请求都有一条完整记录；已删除逐调用强制观测与强制收尾方案。
- 原地迁移沿用既有 SOP：不新增 staging、backup、journal 或整库事务；可识别目标记录可跳过，损坏/歧义数据 fail closed。不承诺任意中断后可自动恢复。

### 具体改动计划

#### A. Settings 路由状态

- `KodexAgentSettings.turnState` 增加可选不透明字符串，旧字段缺失默认为 null，表示尚无值；不迁移旧 settings。请求直接投影当前快照，不增加 scope envelope 或发送时校验。
- 原值不进入模型 input、日志、错误文本或 token-count；检查 settings 整体日志/调试输出，避免新增字段随 data-class toString 泄漏。
- 当前轮次首次值固定；后续响应缺头不清空，返回不同值不覆盖，诊断能记录则记录。不把所有 Session 共用的 OpenAiClient 变成全局状态表。
- header 到达时通过 AgentState 串行化写入最新 settings 并推进索引；持久化完成后再继续消费流。不能等 completed/usage，也不能用请求开始时的旧快照覆盖期间的模型、标题、plan 等更新。
- 同时约束 `updateSettings` 的整份替换：普通用户设置修改必须保留内部路由字段；收到路由状态的内部操作只 patch 路由。覆盖测试必须模拟旧快照与响应头交错。

| 操作 | 计划中的状态处理 |
| --- | --- |
| 初始化 | 即使初始模板携带状态，也不继承；字段为空 |
| 重启/重建同 Session | 从 settings 恢复并直接回传 |
| 同 turn 工具续接、模型响应重试 | 保留首次值 |
| 当前历史推断产生新 turnId | 在同一 settings 快照清空状态；覆盖 appendUserMessage 与 injectHistory 两条路径 |
| 模型/effort/tier/cwd/标题设置变化 | 不以普通设置修改为理由覆盖或清空状态；记录实际请求参数 |
| 同 Session revert | 恢复选中历史记录中的值，不进行额外校验 |
| fork | 完整 fork 与历史位置 fork 均发布 fresh turnId + null turnState 的新当前 settings；threadId 自然变化。旧快照不改，revert 可以恢复其中的旧 turnId/turnState |
| 普通 turn 内 compaction | 上下文归零不等于 turn 结束；保留普通 turn 状态，压缩专用请求不得用其响应头污染普通状态 |
| 账号或端点变化 | 本任务不增加绑定或自动失效规则，仍按 settings 投影 |

#### Fork 标识专项审查

- 当前 main 的两条业务入口已在同一次新快照提交中生成 fresh turnId 并清空 turn-state：`Kodex/app/viewmodel/session/src/commonMain/kotlin/io/github/stream29/kodex/cli/session/SessionViewModels.kt:339–350,381–392`。复制历史仍不改写。
- `threadId` 不存 settings，由目标 storage URI 派生（`KodexRequestProjection.kt:59–63`）；fork 后首次请求即应使用目标 thread，不依赖用户再次输入来更换 turn。
- 实际请求 windowId 是 `threadId:windowNumber`（`CompactionModels.kt:83–84`），因此随新 thread 改变；不为本修复重置 windowNumber 或改写复制历史的窗口链。
- 默认 promptCacheKey 跟随新 thread；显式 override 仍保留（`KodexRequestProjection.kt:29`）。不能把它误当作必须清空的 turn-state，也不凭字段名批量重置 installationId/sessionId 等其他 settings。
- 底层 `forkRawTo` 继续只复制原始 timeline；两个业务入口在复制/选定边界后复用相同的当前 settings 转换，避免完整 fork 与历史 fork 行为分叉。不改源会话和历史用量诊断。
- 验收：两种 fork 的 threadId/当前 turnId 都与源不同、turnState 为空；无追加用户消息时直接 resume 也成立；目标重开保持新值；原设置历史不变；revert 到继承快照明确恢复旧 turnId/turnState；无显式 cache override 时 cache key 随目标变化，有 override 时原样保留。

#### B. 传输链路

- `utils/ktor-client-ext/SseRequests.kt` 增加可等待的响应头交付能力；`openai/client-contract` 用具名参数承载 turn-state，并交付必要的响应头 metadata，禁止以任意 extraHeaders 暴露所有头。
- `openai/client` 从真正的 HTTP 响应头提取状态和 `x-oai-request-id`；若支持 response.metadata，同样采用首次值语义。未取得头与“取得头但无该字段”分别可辨。
- 保持 streaming-only 接口，不伪造 Responses API JSON 事件表达本地传输状态；同步更新 mock-client DSL 和所有调用点。
- **阻断本地 I/O 被网络重试吞掉**：持久化回调位于 transport catch 之外，或使用独立、明确不 retry 的异常边界。磁盘故障必须终止并报告，不能归为 `IOException` 网络重试。取消仍传播为取消。
- 普通 Responses 是本轮 header 修复对象；不切换 lite/WebSocket、不改变提示词布局、不扩展工具输出截断。压缩专用传输的完整计量不纳入本轮普通响应 timeline。

#### C. Token-count 新结构与消费语义

- 每条记录仍是 context-count snapshot，具有 `kind` 与 `total_tokens`，可附加 `usage` 和 `diagnostics`：
  - `legacy`：仅旧 total；旧 0 仍是 legacy。
  - `initialization` / `compaction`：明确的 synthetic 0，不计入请求消费。
  - `response`：可解读的普通 response 用量快照；不是每请求完整账本。
- `usage` 保留能可靠解读的服务端 input/output/total、input cached/cache-write、output reasoning；缺字段使用未知，不默认填 0。新增可选诊断解读失败不应丢掉已经可用的基础 token count。
- 诊断字段能取则存：turn/window 身份、请求模型/effort/tier、服务端实际模型/tier、response ID、HTTP request ID、turn-state 是否发送/收到/采用。不要求这些字段齐全，不保存完整错误正文或原始 header。
- 延续普通 completed 中可解读 usage 的现有记录入口；没有可用计数/usage 就允许没有 change point，不为请求开始、失败、断流、取消或缺 usage 强制造记录，不新增“每次调用必须收尾落盘”的生命周期。
- 本轮不额外建立 failed/incomplete/HTTP 重试账本，也不要求恢复进程被强杀时的缺失记录。若可选细节缺失或无法解读，降级保留可靠的基础字段；不伪造成功、零命中或零消费。
- 初始化/压缩使用显式 reset；预算、自动压缩与 UI 只投影上下文 total，不把累计消费当作上下文大小。尽力采集不代表修改已有 storage 写入故障协议或静默忽略损坏文件。
- API DTO 与持久 snapshot 分层，快照放 storage contract/models 边界，避免迁移依赖当前业务 DTO。
- 改动消费端：filesystem/in-memory storage、初始化/压缩、AgentState、context-window、AgentRuntimeViewModel 及对应测试。fork/revert 继续遵循 timeline 索引语义，不另增第七条 timeline。

#### D. 显式迁移

- 复用 `prepareKodexHome` 的 Home 写租约和 `KodexHomeMigrations`；版本已核对并升至 0.4.5。应用版本激活与新 schema 一起验证，旧版本号不会让新 runtime 打开旧整数。
- 新目标路径保存冻结的 migration、最小私有 codec 与 fixtures；已发布的 0.3.3/0.3.5/0.4.3 不改动，迁移不得依赖当前业务模型。
- 范围仅当前 root Session 的 `token-count/<n>.json`，包括 archived Session；无记录的空 Session 合法。枚举复用 filesystem-layout；保留 settings、timestamp、latest pointer、未知路径和 legacy subagents。
- 合法旧 Long 转为 `legacy` 对象；合法目标结构跳过；非法数值类型、溢出或损坏对象报路径并中止。不得凭 “是 JSON object” 就认为已迁移。
- 转换保持原 total 和 index，不从相邻事件推算缺失明细；在既有接口下原地写入。正常重跑跳过完整目标，部分写入损坏则 fail closed。
- 只有 action 全部成功后才由 Home coordinator 推进版本。升级失败时旧版本号不代表数据仍为旧格式；操作上必须持续停机，不能启动旧客户端接管半迁移 Home。
- 真实 Home 的停机/备份/升级是后续发布操作，不在本轮规划或隔离测试中执行；不新增自动备份框架，也不主动停止用户进程。

## 2026-09-20 实施结果

- `KodexAgentSettings.turnState` 已加入 Session settings timeline；响应头中的首个非空 `x-codex-turn-state` 在 SSE body 消费前写入最新 settings，已有值不覆盖，取消继续传播，持久化异常不会伪装成网络重试。
- 普通 Responses 请求现在按 settings 回传 `x-codex-turn-state`；SSE transport 仅向上层交付 `x-codex-turn-state` 与 request ID，不暴露原始 header 集合。压缩专用请求不接入普通 turn-state 回写。
- token-count 已改为结构化 snapshot：`kind`、`total_tokens`、可选完整 usage（input/output/total、cached/cache-write、reasoning）和 best-effort diagnostics（response/request ID、实际 model/tier、状态收发标志）。上下文预算仍只读取 `totalTokens`。
- Home 版本升至 `0.4.5`；新增原地 token-count migration：旧整数转换为 `legacy` 对象，目标对象跳过，损坏或歧义记录 fail closed；settings 不需要迁移。
- 两个 fork 入口均只转换 fork 后当前 settings：生成新的 `turnId`、清空 `turnState`；复制历史不重写，之后 revert 到旧历史仍按用户确认的快照语义恢复旧值。
- 新增/更新 HTTP、migration、storage、context-window、AgentState 测试。已通过：
  - `./gradlew :openai-client:jvmTest :app-migration-impl:jvmTest --no-configuration-cache`
  - `./gradlew :agent-state-impl:jvmTest :agent-state-context-window:jvmTest :agent-storage-in-memory:jvmTest :agent-storage-contract-ext:jvmTest --no-configuration-cache`
  - 受影响 Linux Native test source compile，以及 `git diff --check`
- 未执行真实 Home migration、真实账号请求或提交；发布前仍需安排停机/备份并在隔离 Home 上走一次实际 coordinator upgrade。

#### E. 验证顺序与准入

- 先模型/序列化：旧 settings 缺字段；新记录各 kind；usage 缺失与零；大 Long；不存在的字段不得伪造。
- 再本地 HTTP fixture：首请求不带头、同轮回传、不同值不覆盖、缺头保留、断流后重试、未携带账号标识仍直接投影 settings、落盘失败不网络重试。
- AgentState + filesystem 重开：只收到头即落盘，销毁再重建能恢复；最新 settings 合并；两条新 turn 推断路径清空；fork 新当前身份与继承历史 revert 按上述专项验收；compaction 不丢失普通 turn 状态。
- 用量测试：completed 完整/部分 usage、缺 usage、可选诊断解读失败、failed/incomplete/断流/取消允许无记录；上下文预算和 UI 投影不回归，不宣称完整请求覆盖率。
- 迁移 tests：纯旧、纯目标、混合、稀疏、空 Session、archived、多版本直接升级、损坏和中断边界、旧二进制版本拒绝、未知文件与历史 migration 保持不变、大 timeline 性能。
- 单元/集成测试仅用 synthetic fixture 和隔离临时 Home；Gradle 复用运行中的 Daemon JVM。真实缓存命中率改善须另做受控实测，不以 mock 测试宣称修复额度根因。

### 阻塞审查结论

| 项目 | 状态与处置 |
| --- | --- |
| settings vs token-count 归属 | 已决：settings 原值，token-count 用量/诊断 |
| 轮次 API / 历史恢复 / 迁移保证 | 已决：沿用推断；fork 当前快照 fresh turnId + 清空状态，旧历史不改且允许 revert；既有原地迁移 SOP |
| fork 当前 turnId 没有轮换 | 确认是计划遗漏；两条业务入口必须同时补齐，底层 raw copy 不改语义 |
| 迁移入口及租约 | 已有可复用实现，不需新框架 |
| scope 身份及 unknown-account 策略 | 已撤回：用户明确不添加额外校验，直接回传 settings 字段 |
| 请求记录与上下文快照 | 已纠正：尽力记录可解读 usage/诊断，不保证每请求记录，不增加失败账本 |
| settings 并发覆盖 / 本地 I/O 重试分类 | 已定位的实现硬要求，以交错与故障注入测试为验收门槛 |
| 目标版本及发布激活 | 实施前核对 0.4.5 未占用；新模型不可在未迁移 Home 上运行 |
| 真实 Home 升级 | 发布门槛：用户安排停机；本轮不执行、不抢占进程 |
| 服务端 token 有效期和缓存因果效果 | 仍未知；不阻塞持久化与观测实现，不能据此承诺缓存命中 |

### 持久化归属调研

- settings 已包含 turnId、windowNumber/windowId、previousResponseId 等请求恢复字段，而非仅用户偏好，见 `Kodex/openai/models/src/commonMain/kotlin/io/github/stream29/kodex/openai/CompactionModels.kt:44–78`。
- token-count 当前仅在 completed 且有 usage 时新增记录，见 `Kodex/agent-state/impl/src/commonMain/kotlin/io/github/stream29/kodex/agentstate/impl/KodexAgentStateImpl.kt:215–220`；turn-state 在响应头阶段就可收到。若只在最终用量记录中保存，会漏掉收到头后断流/取消、没有 usage 的情况。把 token-count 改为允许独立路由事件虽可实现，但会将用量时间线扩大为请求状态日志。
- 压缩会更新 settings 的窗口身份并将 token-count 写为 0，见 `Kodex/agent-storage/contract-ext/src/commonMain/kotlin/io/github/stream29/kodex/agentstorage/contract/ext/CompactionStorage.kt:19–27`。路由状态不能因上下文计数归零而被隐式丢弃。
- Rust 的可对齐约束是首次值固定、同 turn 续接回传、跨 turn 不复用；内存容器不是服务端要求不得落盘的证据。其实现也未提供跨进程恢复和有效期承诺。已在线核对官方固定版本源码：https://github.com/openai/codex/blob/5ecb3afd1b/codex-rs/core/src/client.rs#L270-L292 及 HTTP 测试 https://github.com/openai/codex/blob/5ecb3afd1b/codex-rs/core/tests/suite/turn_state.rs#L47-L86 。
- settings 实施需处理：首次收到值走 AgentState 串行化写入，合并最新 settings、当前轮次尚无值时写入；不得等 usage 到达。当前 `updateSettings` 整份替换快照（`KodexAgentStateImpl.kt:398–406`），需要防止旧 UI/标题/模型设置快照抹掉新收到的值，亦不能用请求开始时的旧 settings 覆盖期间的用户设置。
- 两个位置都会被 raw fork 复制、被 revert 回退（`FileSystemAgentStorage.kt:132–146`、`KodexAgentStorage.kt:69–78`）。恢复与清空已按上表确定；此前建议的身份绑定校验已被用户明确否决，不再作为实现要求。
- 持久化可保证客户端重启后仍有原值，不保证服务端长时间后仍接受或仍有缓存；本轮未新增付费端点验证。诊断可记录持久记录索引及收发/采用状态，避免再维护一份可恢复的原值。

## 真实 Session 长上下文重放

- 用户追加授权：使用本地真实 Session，测试更长场景。
- 时间：2026-09-20 03:35:53—03:36:39（UTC+8）；12 次请求全部 HTTP 200 / response.completed，无重试。
- 只读 Session 367（Android OSS 音乐同步）的两个历史截面；合并 index/work 稳定事件，校验历史工具调用与结果一一配对。不执行原工具、不修改源 Session。
  - state 64：原 `token-count/64.json` 为 55,436；投影 43 个输入 item、194,710 UTF-8 字节。
  - state 391：原 `token-count/391.json` 为 151,401；投影 278 个输入 item、645,483 UTF-8 字节。
  - 投影保留用户/助手消息、加密 reasoning、工具调用和已存结果；web 只发送客户端使用的明文 output，不发送存储专用 encrypted_output。
  - 长样本 `work/153.json` 含 OSS 明文凭据；在内存投影中遮蔽 access key / secret 的两处值。原文件未修改，reasoning 密文保持不透明。
- 每组新建独立 thread/turn/window/cache key，组内固定；组首开发者测量指令含独立 nonce，避免四组直接共用相同前缀。顺序为短不回传→长回传→短回传→长不回传；组间并非逐字相同前缀。
- 每组连续 3 次请求：模型调用两次无外部动作的 `cache_probe`，脚本分别附加 `{"ok":true}`，最后模型简短答复；不触发原工具。
- 使用常规 Responses HTTP SSE，顶层 tools、`include=[]`、未指定 reasoning.context；不启用 lite。模型 `gpt-6-astra`、effort `high`、请求 priority；所有响应实际报告 default，reasoning_tokens 均为 0。
- 原工具声明和瞬态系统前缀替换为固定测量指令/单个测试工具，不称作原始网络请求逐字重放；下表实际 input_tokens 比源 token-count 小，不能互换。

| 原截面 / turn-state | 三次 input_tokens | 三次 cached_tokens |
| --- | --- | --- |
| 64 / 不回传 | 50,223 / 50,257 / 50,291 | 0 / 0 / 0 |
| 64 / 回传 | 50,222 / 50,256 / 50,290 | 0 / 0 / 0 |
| 391 / 不回传 | 146,505 / 146,539 / 146,573 | 0 / 0 / 0 |
| 391 / 回传 | 146,505 / 146,539 / 146,573 | 0 / 0 / 146,304 |

- 回传组保存首响应 header，后两次原样回传；仅首响应带 turn-state。不回传组每次响应都带 turn-state。没有观察到 SSE metadata 下发状态。
- 长回传组第三次命中 `146,304 / 146,573 = 99.816%`，剩余 269 input token 未标为 cached；该组第二次仍为零。对应第三次延时 3.068 秒，不回传为 5.083 秒；单次延时不构成性能结论。
- 每组 output_tokens 为 18 / 18 / 5；全部 cache_write_tokens 为 0，包括有缓存命中的请求，不能用该字段零值证明未写缓存。
- 总 input_tokens 1,180,773，其中 cached 146,304、未标缓存 1,034,469；output 164，total 1,180,937。不是按额度加权后的消费量。
- 账户整数周用量快照 48%→50%。期间存在其他会话和本调查自身活动，不能将这 2 个百分点全部归于实验。
- 本次证明：真实 14.65 万 token 历史在常规 Responses 路径也能出现近全前缀缓存命中，并非只有 lite 才能命中。
- 本次未证明：turn-state 单独造成命中或保证稳定命中。每个条件只有一组、两次续接，独立前缀和执行时间不同；后端缓存写入/路由不可见，不能排除随机命中及缓存准备时机差异。
- high 是请求参数，但本次受控回复没有产生 reasoning token；不能等同于真实任务连续推理、长工具循环或跨用户轮次的命中表现。
- 结论仍是：缺失回传是确定的协议差异，根因归属尚未证实；不能把今晚超过 20% 扣额归因于该头。按约定停止于 12 次，不追加耗额测试。
- 逐次用量、请求 ID 和样本摘要已记录；临时脚本与结果 JSON 已删除。该实验阶段未改认证文件、源 Session 或客户端实现。

### 长样本证据定位

- 源文件集合 SHA-256（按 state index 排序，散列 timeline/index 换行及原始 JSON 字节）：
  - state 64：`aa2fb4455dc0eab1405396d7b7aaa35d050741e8da956fded34a0398624fa902`。
  - state 391：`0af22756b075e32d2600dca93da94cf35aef0a967d3a010f85ab8287fb0fcced`。
- 遮蔽后的历史投影 SHA-256（ensure_ascii=false、紧凑 JSON，不含测量前后缀）：
  - state 64：`72a79277b95926478ed18388b13d25800c10d8dd30cd138954609103a91f6431`。
  - state 391：`1ec9e2603de44f14b194966343ffe5df255e43ad7e565008a438a3ff27748232`。
- `x-oai-request-id` 按各组三次请求顺序：
  - 64 不回传：`86541524-62fc-4a55-b596-cee70374c4d7`、`1604175a-eafb-44fc-b0ab-9e7a9f2dce9e`、`c68af827-5201-4d93-b949-4af51d7bd9c8`。
  - 64 回传：`7fd87a2b-c37d-480a-b0d8-3bc99c4345c9`、`d969a553-234f-446a-880c-e8ded448843c`、`6c14a6a5-22dd-42a4-b5ff-3a302887acf6`。
  - 391 不回传：`30f25811-af68-42c5-a4b5-c8c80402ed93`、`7bd9d8ff-e908-4688-a0e2-457473b832e6`、`fa7d4100-63ff-4d3f-87fb-79b6ea1ca1f9`。
  - 391 回传：`75579bb3-a0ed-48da-814c-793baa087f6a`、`672eb5f0-3ff8-4bbe-a2d5-98b167456f84`、`3a88ab55-71b1-4acb-b227-d3ba69d6bfe7`。

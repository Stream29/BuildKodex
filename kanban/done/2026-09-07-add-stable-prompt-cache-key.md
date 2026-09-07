# Task Tree

- [done] 为模型请求补充稳定缓存键
  - [done] 确认普通新 Session 缺少显式缓存键
  - [done] 核实官方及 Kodex 身份、请求和恢复路径
  - [done] 与用户确定 threadId 默认键，不随 window 变化
  - [done] 制定最小修改范围与分层验证方案
  - [done] 实现统一请求投影的默认键
    - [done] 仅在 settings.promptCacheKey 为 null 时使用 clientMetadata.threadId
    - [done] 在 settings KDoc 说明 null 表示未存 override、AgentState 请求使用 threadId
  - [done] 补齐请求与生命周期回归测试
    - [done] 投影测试覆盖默认键、override 原样透传及 JSON 字段
    - [done] AgentState 测试覆盖续跑、重试、压缩及压缩后普通请求
    - [done] 文件系统 Session 测试覆盖重开、默认 fork 和 override 继承
  - [done] 执行相关模块检查并记录验证层级与限制

# Details

## 执行结果（已完成）

- 默认缓存键在统一投影中使用 `promptCacheKey ?: clientMetadata.threadId`，
  不回写 settings；非 null override（包括空串、空白）保持原样。
- 投影 JSON 矩阵、AgentState 请求/Retryable 重试/压缩/续跑测试通过；
  filesystem 定向用例验证同目录重开、默认 fork 新键和 override fork 继承。
- AgentState JVM 与 linuxX64 测试、定向 filesystem 生命周期测试通过；
  全量 filesystem 测试两次停滞后终止，未声称全量通过。定向用例通过，
  未为排查其他测试扩展生产修改范围。
- 已更新 `checklist/agent-state-and-runtime.md`；CLI Linux release 链接通过。
- 未做真实模型缓存采样，不证明缓存命中率或收益；未提交或发布。
- 以下保留推进前的决策和验证计划；原“暂不执行”限制已由后续授权解除。

- 执行授权更新：用户随后明确要求由当前会话执行全部四项任务；下述旧的
  “暂不执行”记录仅为历史，当前按统筹任务顺序实施，不创建提交。

- 授权：用户于 2026-09-07 确认使用 threadId；具体计划已完成，用户随后授权移至 executable，但明确要求不要执行，未来另行安排执行人。当前仅完成状态迁移，实施与测试节点保持未完成；未经后续明确执行授权，不修改 Kodex 实现、不运行实施测试、不创建 commit。
- 已确认决策：默认键复用现有 threadId，不包含 turnId、windowNumber 或 requestKind；普通请求、续跑、重试、压缩及压缩后的请求保持默认同键，不新增持久化身份。
- 保持现有行为：非 null override 原样优先，不 trim、不把空串转换为默认值、不新增校验或禁用缓存选项；fork 继续复制 settings，因此显式 override 也继承。默认 fork 因新 storage URI 得到不同 threadId，不主动跨 Session 聚合。
- 稳定性边界：threadId 由 storage URI 确定性派生；原目录重开稳定，搬迁目录可能变键，删除后复用同路径可能复用键。不是跨目录永久唯一身份，也不是安全隔离边界；本任务不改身份算法或目录分配。
- 阻塞检查：普通与压缩请求共用投影，clientMetadata.threadId 为非空 String；默认值可在投影时计算，旧 settings 无须迁移。未发现进入 planning 的技术阻塞；未来构建环境与真实缓存收益尚未验证。
- 范围排除：usage 持久化、遥测、缓存策略新字段、前缀重排、上下文热重载、上下文换窗及压缩保留算法。保留用户认可的压缩原文整条保留，不恢复边界消息截断。

实施落点（待后续授权）：
- [KodexRequestProjection.kt:15–33](../../Kodex/agent-state/impl/src/commonMain/kotlin/io/github/stream29/kodex/agentstate/impl/KodexRequestProjection.kt#L15)：将赋值收敛为 `promptCacheKey = promptCacheKey ?: clientMetadata.threadId`；不新增 helper、状态、构造参数或持久化字段。
- [CompactionModels.kt:40–41](../../Kodex/openai/models/src/commonMain/kotlin/io/github/stream29/kodex/openai/CompactionModels.kt#L40)：仅澄清 KodexAgentSettings 的 null 语义；通用 ResponsesApiRequest DTO 继续允许 null，不给所有 OpenAI 客户端请求强加 thread 默认值。
- 不改 Application 新 Session 设置、AgentState 两处调用入口、Session repository、fork/revert、HTTP 传输或 window metadata。
- 测试优先扩展现有 `KodexRequestProjectionTest.kt`、`KodexAgentStateImplTest.kt` 和 `FileSystemKodexSessionRepositoryTest.kt`；复用现有假客户端及文件系统 fixture，不新增测试框架或跨模块抽象。
- 实施前识别项目已打开的 IDE 并复用相关能力；重新检查并行差异，仅改本任务所需代码/测试，不整理其他任务。

验证方案：
- 投影单元测试：null override 时 key 等于 metadata.threadId；改变 turn/window/requestKind 不改变默认键，改变 threadId 则改变默认键；非空、空串及空白 override 均原样透传；请求生成不回写 settings。
- JSON 正确性：用 OpenAiJsonCodec 序列化实际投影结果，断言 `prompt_cache_key` 存在且为预期字符串，不只检查 Kotlin 属性；保持通用 DTO 的可空行为。
- AgentState 路径：捕获普通请求、Retryable 后再次请求、remote compaction v2 和压缩后请求；确认默认 key 相同，即使 windowId 已改变。另覆盖 override 在普通/压缩两条路径均生效，不修改输入保留算法。
- 文件系统生命周期：创建 Session 并请求，关闭并重开同目录后再次请求，断言 key 相同；fork 后 key 等于新请求的 threadId 且不同于源；显式 override 的 fork 请求仍发送原值。null 默认不落入 settings，避免 fork 继承源的派生键。
- 编译/测试：先按 Gradle skill 检测并显式复用现有 Daemon JVM、确认本机可用测试任务，再执行 agent-state-impl、agent-session-filesystem 和 openai-client 的相关测试。分别报告编译、单元/假客户端行为测试及实际运行结果；未执行不得标通过。
- 实际运行与收益：假客户端及 JSON 测试只证明请求正确，不证明服务端命中。真实端点运行须另获请求/费用授权；如测收益，固定模型、配置、输入序列和时间间隔，对比无键与稳定键，分别观察续跑和压缩序列的原始响应 cached_tokens 与延迟，多次重复并记录冷/热条件。不为测量新增 usage 持久化或遥测；若端点不提供用量细节，明确收益无法判定，不以延迟单独推断命中。收益测试不作为本任务请求正确性的完成门槛。

源码依据：
- Kodex：[默认新 Session 设置:549–558](../../Kodex/app/viewmodel/application/src/commonMain/kotlin/io/github/stream29/kodex/cli/app/Application.kt#L549)、[普通请求:180–198](../../Kodex/agent-state/impl/src/commonMain/kotlin/io/github/stream29/kodex/agentstate/impl/KodexAgentStateImpl.kt#L180)、[压缩请求:261–290](../../Kodex/agent-state/impl/src/commonMain/kotlin/io/github/stream29/kodex/agentstate/impl/KodexAgentStateImpl.kt#L261)、[thread 派生:58–66](../../Kodex/agent-state/impl/src/commonMain/kotlin/io/github/stream29/kodex/agentstate/impl/KodexRequestProjection.kt#L58)、[fork 与重开:132–190](../../Kodex/agent-session/filesystem/src/commonMain/kotlin/io/github/stream29/kodex/agentsession/filesystem/FileSystemKodexSessionRepository.kt#L132)。
- 官方本地子模块 `8d7cc24a87`：[键选择:504–515](../../shared-context/codex/codex-rs/core/src/client.rs#L504)默认 session_id，不读取 window；[根 Session 身份及 resume/fork:759–795](../../shared-context/codex/codex-rs/core/src/session/session.rs#L759)；[压缩 v2 共用 stream:371–397](../../shared-context/codex/codex-rs/core/src/compact_remote_v2.rs#L371)。不复制官方多 Agent/内部 Session 特例。
- 无键不等于无缓存，同键也不保证命中；当前未实测实际损失或收益。此次仅完成源码核查和规划，未运行编译、应用测试或真实模型请求。

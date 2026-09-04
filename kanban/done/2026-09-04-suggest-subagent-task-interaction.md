# Task Tree

- 实现重制版 Multi-agent 并行工作系统
  - [done] 明确用户授权边界
    - [done] 主 Agent 只建议任务
    - [done] 用户接受后才创建 Session
    - [done] 不恢复模型直控调度工具
  - [done] 定义模型可见协议
    - [done] 定义任务参数
    - [done] 定义接受与拒绝结果
    - [done] 对齐 Session meta URI
  - [done] 定义确认交互
    - [done] 任务内容只读
    - [done] 配置由用户统一选择
    - [done] 接受和拒绝均返回可选反馈
  - [done] 定义执行器 Session 语义
    - [done] 使用普通 root Session
    - [done] 创建并打开全部 Session Tab
    - [done] 无界并发启动首个 turn
    - [done] 不自动回流执行结果
  - [done] 对齐既有 pending-tool 生命周期
    - [done] pending call 使用 unstable timeline
    - [done] ViewModel 草稿保持进程内状态
    - [done] 完成后写 stable event 并恢复 runtime
  - [done] 排除实现阻塞点
    - [done] 确定 Application 批量创建命令边界
    - [done] 确定跨 Session 创建的重复执行边界
    - [done] 对齐 Stop hook 的宿主交互语义
    - [done] 确定 Session meta DTO 模块所有权
    - [done] 保证显式任务名称不被自动标题覆盖
    - [done] 对齐 URI 关联任务的交付顺序
    - [done] 确定同一响应内多个 host-owned pending call 的处理
    - [done] 确定 PreToolUse/PostToolUse hook 边界
    - [done] 确定 remote compaction retention
    - [done] 确定首条 prompt 的持久化边界
  - [done] 形成下一阶段实现路线与验收条件
  - [done] 实现新版 Multi-agent
    - [done] 增加 suggest tool contract 与 fixed tool spec
    - [done] 增加 typed pending 与 stable history event
    - [done] 扩展 AgentState tool projection 与 compaction retention
    - [done] 扩展 host-owned pending interaction arbitration
    - [done] 增加 suggest interaction ViewModel 与确认界面
    - [done] 增加 Application 批量 Session 创建命令
    - [done] 接入 Tab 安装、prompt 启动与自动标题 suppress
    - [done] 补齐 hooks、serialization、history、token 和集成测试

# Details

## 目标

- Agent 只提出并行任务建议，不直接创建、控制或等待其他 Agent。
- 用户确认是创建执行器 Session 的唯一授权点。
- 用户可以一次接受多个任务，并在确认时统一选择 Session 配置。
- 执行器是普通 root Session，不恢复旧版递归 Agent Tree。

## 模型可见工具

`suggest_subagent_task` 仅接收任务内容：

```kotlin
@Serializable
public data class SuggestSubagentTaskArgs(
    public val tasks: List<SuggestedSubagentTask>,
)

@Serializable
public data class SuggestedSubagentTask(
    public val name: String,
    public val prompt: String,
)
```

```json
{
  "tasks": [
    {
      "name": "Inspect storage",
      "prompt": "Review the storage design and report risks."
    }
  ]
}
```

- `name` 是实际的新 Session name，不是临时 UI 标题。
- `prompt` 按新 Session 的初始 user message 调度，但不构成 accepted 前的持久化
  guarantee。
- 不复制主 Session 历史，不注入隐藏 developer context。
- 模型不提供 model、reasoning、service tier、cwd 或 Ask User 配置。
- 不提供 `task_id`；输入和结果依靠数组顺序对应。
- 不限制任务数量。
- 不限制名称重复，包括同批重名和与已有 Session 重名。
- 不做内容级校验或规范化，不 trim，不检查空名称、空 prompt 或空任务数组。
- JSON schema 只约束 `tasks`、`name` 和 `prompt` 的结构与类型。
- tool description 必须说明每个 prompt 进入没有源历史的新 Session，因此任务说明应当
  自包含。
- tool description 必须说明 accepted 不代表任务完成，也不会自动返回执行结果。

## 工具结果

正常业务结果：

```kotlin
@Serializable
public sealed interface SuggestSubagentTaskResponse {
    public val feedback: String?

    @Serializable
    @SerialName("accepted")
    public data class Accepted(
        override val feedback: String?,
        public val sessions: List<SuggestedSessionMeta>,
    ) : SuggestSubagentTaskResponse {
        @EncodeDefault(EncodeDefault.Mode.ALWAYS)
        public val decision: SuggestSubagentTaskDecision =
            SuggestSubagentTaskDecision.Accepted
    }

    @Serializable
    @SerialName("rejected")
    public data class Rejected(
        override val feedback: String?,
    ) : SuggestSubagentTaskResponse {
        @EncodeDefault(EncodeDefault.Mode.ALWAYS)
        public val decision: SuggestSubagentTaskDecision =
            SuggestSubagentTaskDecision.Rejected
    }
}

@Serializable
public enum class SuggestSubagentTaskDecision {
    @SerialName("accepted")
    Accepted,

    @SerialName("rejected")
    Rejected,
}

@Serializable
public data class SuggestedSessionMeta(
    public val uri: String,
    public val name: String,
)
```

模型可见 JSON：

```json
{
  "decision": "accepted",
  "feedback": null,
  "sessions": [
    {
      "uri": "file:///home/user/.kodex/sessions/42",
      "name": "Inspect storage"
    }
  ]
}
```

```json
{
  "decision": "rejected",
  "feedback": "Please split this into smaller tasks."
}
```

- `feedback` 在两个分支中都存在，未填写或纯空白时为 `null`。
- 同意时的反馈只返回给主 Agent，不修改只读任务。
- 拒绝是正常业务结果，使用成功的 function output。
- 拒绝不创建 Session。
- 接受结果不返回最终 model、cwd 或 Ask User 配置。
- `sessions` 与 `tasks` 保持相同顺序。
- `SuggestedSessionMeta` 与 Recall 的 `AgentSessionMeta` 使用相同的 `uri + name`
  wire 语义，但由 tool contract 自己定义。
- tool contract 不直接依赖 `agent-context-prefix-contract`。
- function output 按 `Accepted` 或 `Rejected` 的具体 serializer 编码，不直接用 sealed
  interface serializer，避免把内部 polymorphic `type` discriminator 暴露给模型。
- `decision` 是显式 wire 字段，不依赖 Kotlin serialization 的 class discriminator。
- `StableToolJson = Json` 默认不编码等于默认值的属性，因此两个 `decision` 属性必须使用
  `@EncodeDefault(ALWAYS)`；否则示例中的 `decision` 会被静默省略。
- clean-model 的 `StableToolJson = Json` 保留显式 `null`，因此空反馈输出为
  `"feedback": null`。
- 不单独返回 Session index、activity time 或配置。

稳定 clean-history 结果仍需要普通工具失败分支：

```kotlin
@Serializable
public sealed interface StableSuggestSubagentTaskResult {
    @Serializable
    @SerialName("completed")
    public data class Completed(
        public val response: SuggestSubagentTaskResponse,
    ) : StableSuggestSubagentTaskResult

    @Serializable
    @SerialName("failure")
    public data class Failure(
        public val message: String,
    ) : StableSuggestSubagentTaskResult
}
```

- `Completed(Accepted)` 和 `Completed(Rejected)` 投影为 `success=true`。
- `Failure` 只服务普通 tool failure、Clear pending 和宿主异常，投影为
  `success=false`。
- 不为 Session 部分创建失败设计业务结果。

## Ask User 可见性

- 保留 `RequestUserInputMode` 类型和 settings 字段名。
- `AskUser` 同时暴露 `request_user_input` 和 `suggest_subagent_task`。
- `NoQuestion` 同时隐藏两个工具。
- 不增加第二个开关。
- `suggest_subagent_task` 与 `request_user_input` 一样加入 fixed visible tool specs，
  不进入 deferred tool search。
- 普通 Responses 请求和 remote compaction 继续使用同一份 settings-based visible tool
  projection，不为本工具新增 request-kind 特例。
- 切换为 `NoQuestion` 不移除已经发出的 pending call。
- 执行器是普通 Session；使用 `AskUser` 时也可以提出新的任务建议，但仍必须由用户接受。

## 用户确认界面

- 展示所有任务的 Session name 和完整 prompt。
- 任务内容只读，不能修改、删除或重新排序。
- 同意或拒绝作用于整个调用，不支持只接受部分任务。
- 提供一份由所有新 Session 共用的批量配置：
  - model、reasoning effort 和 service tier 的有效目录组合。
  - cwd。
  - `RequestUserInputMode`。
- 初始配置来自工具调用时的源 Session settings snapshot。
- 用户可以在接受前修改批量配置。
- 不复制 `plan`、instructions、window、response、compaction 或身份状态。
- 其余字段使用普通新 Session 初始化语义。
- 提供始终可编辑的反馈输入框。
- 提供“同意”和“拒绝”两个明确动作。
- 同意和拒绝都会返回反馈；同意不会把反馈应用到任务。

界面草稿沿用 `request_user_input`：

- 工具参数和 pending call 持久化。
- 配置选择、反馈文本、revision 和 submission 状态由 ViewModel 持有。
- Tab 关闭后 pending call 保留，未提交草稿不持久化。
- 重开后从同一个 typed pending event 重建交互。
- 工具调用时的默认配置应从该 pending call 首次出现位置对应的 settings snapshot
  重建，不新增草稿持久化。

## 接受后的 Session 行为

- 每个任务创建一个独立普通 root Session。
- Session 创建完成时已持久化：
  - 独立 Session identity 和 URI。
  - 任务指定的 name。
  - 用户最终选择的批量配置。
- 每个 Session 分别经过普通 New Session settings 初始化，再只覆盖 name 和用户选择的
  model 配置、cwd、`RequestUserInputMode`；各 storage 继续由 `initialize` 生成自己的
  turn/window identity。
- 所有 Session 都作为普通 Session Tab 打开。
- 不增加无 Tab 的后台 Session 生命周期。
- 新 Tab 按任务数组顺序加入。
- 批量创建后仍保持主 Session Tab 为当前选中项。
- 工具完成不等待 prompt 写入、模型首包或执行完成。
- 每个首条 prompt 由对应 Tab 的既有 runtime 生命周期异步提交。
- 首条 prompt 仍触发普通 Session 的 `UserPromptSubmit` hook；hook 可以增加 context
  或停止该子 Session 的首个 turn。
- 所有 prompt 无界并发启动，不增加队列、信号量或并发限制。
- prompt 提交或模型 turn 失败只在对应普通 Session Tab 中展示。
- `accepted` 只表示用户接受且 Session 已创建、配置已持久化。
- 不增加跨 Session outbox；进程在 accepted 后、异步 prompt 提交前退出时，可能留下尚未
  收到首条 prompt 的已创建 Session。
- 执行完成后不通知主 Session，不自动回传结果。
- 不新增 wait、status、message、collect 或批次管理工具。

## Pending-tool 生命周期

本工具必须复用当前 `request_user_input` 的宿主交互模式：

- `PendingSuggestSubagentTaskToolEvent` 是 typed `PendingToolEvent`。
- pending event 保存 `callId`、`itemId`、模型参数和发起该 Responses 请求的
  `settingsIndex`，并写入 `unstable` timeline。
- `settingsIndex` 是 host-only 恢复信息，不进入模型可见 function-call arguments。
- 确认界面从源 storage 的 `settings[settingsIndex]` 重建初始批量配置，不读取重开时的
  latest settings，也不复制完整 settings snapshot。
- Agent loop 到达 `ToolPending` 后停止。
- Generic `KodexToolRuntime` 不执行该 host-owned call。
- 从当前 `ToolPending` 按输出顺序选择第一个 host-owned call，并只显示其对应的专用
  交互面板。
- 关闭 Tab 不完成、不拒绝也不清除调用。
- 用户提交同意或拒绝后，ViewModel 调用 `completeToolCall`。
- 完成操作原子写入 stable event、从 unstable 移除该 call，并恢复同一 runtime。
- 显式 Clear pending 使用普通失败投影，不伪造 `Rejected`。
- 不增加独立 pending batch、恢复协议或第三种业务结果。

## Session 身份与自我认知

关联任务：

- [Bundle Kodex Home recall](../done/2026-09-04-bundle-kodex-home-recall.md)

采用其最新 URI 决策：

- `KodexAgentStorage.uri` 是非空绝对 URI。
- Filesystem Session 使用标准 `file:` URI。
- In-memory Session 使用 `memory:<token>` URI。
- `AgentSessionMeta` 包含 `uri` 和 `name`。
- 每个执行器在自己的 environment context 中看到自己的 Session meta。
- 执行器不自动看到源 Session meta 或历史。
- 主 Agent只通过 accepted 结果得到新 Session meta。
- 不持久化 master/executor 身份或父子关系。
- 不写入旧版 `subagents/` 目录。

## 明确排除

- 不恢复旧版六个模型可见调度工具。
- 不恢复 `AgentMode`、`AgentPathResolver` 或递归 `KodexAgentSession`。
- 不恢复 Agent Tree UI。
- 不恢复 `SubagentParentNotificationRuntime`。
- 不提供模型直接创建、发送消息、等待、中断或关闭其他 Session 的能力。
- 不提供自动结果汇总。
- 不为该功能新增并发调度基础设施。

历史参考：

- [旧版 Multi-agent V2](../done/2026-07-21-implement-multi-agent-v2.md)
- [旧版活动窗口 Fork](../done/2026-08-27-hardcode-compaction-based-subagent-fork.md)
- [AgentState 与 AgentRuntime](../../checklist/agent-state-and-runtime.md)
- [CLI Session 与 Agent ViewModel 边界](../../checklist/cli-session-view-models.md)

## 实现阻塞点审查

### Application 批量创建命令

- `request_user_input` 只需要当前 Agent runtime，因此 Agent ViewModel 可以独立完成。
- 本工具需要创建 root Session、打开多个 Tab、保持当前 Tab，并启动多个 Agent。
- 这些能力当前由 `ApplicationViewModel`、`PersistedSessionViewModelRegistry` 和
  `AgentViewModel` 分层持有。
- 不能让 Agent ViewModel 直接拥有 Application 或 Session registry。
- 使用 Application-owned 批量命令；frontend 从 Application screen 把该命令传给当前
  Session 的建议面板，不让 Agent ViewModel 反向持有 Application。
- 命令通过 `PersistedSessionViewModelRegistry.create` 初始化每个普通 Session。
- 命令在一次 application command serialization 中追加全部 Tab，并保留原
  `selectedIndex`；不循环调用当前会逐次切换焦点的 `openSession`。
- Tab 安装后，由 Application owner scope 为每个 `rootAgent.submit(prompt)` 启动独立
  coroutine；这些提交不属于 accepted 的等待条件。
- 批量命令返回按任务顺序排列的 `SuggestedSessionMeta`，确认 ViewModel 随后完成源
  Session 的 pending tool call。

### 重复执行窗口

- UI revision 可以阻止同一 ViewModel 内的双击和 stale submit。
- Session 创建与源 Session 的 `completeToolCall` 分属不同 storage，无法使用现有单
  timeline 原子提交。
- 如果进程在“已创建 Session、尚未完成源工具调用”之间退出，重开后源调用仍是
  pending，再次接受可能重复创建 Session。
- 已确定接受该极窄重复窗口。
- 不增加以源 URI 和 callId 为键的全局 dispatch ledger、恢复协议或清理机制。
- 正常进程内提交继续使用 callId、revision 和 submission state 防止双击与 stale
  frontend event 重复创建。

### Stop hook

- 当前 `TurnHookRuntime` 将 pending 中的 host-owned `request_user_input` 和
  `suggest_subagent_task` 视为可进入 Stop hook 的宿主交互。
- Stop hook 的 Stop/Continue 会把该 pending call 完成为失败，再停止或注入 continuation。
- `suggest_subagent_task` 使用完全相同的 Stop hook 语义。
- `Finish` 保留 pending 建议并显示确认面板。
- `Stop` 把建议完成为普通工具 failure，不创建 Session。
- `Continue` 先把建议完成为普通工具 failure，再注入 hook fragments 让 Agent 继续。
- 没有普通 assistant text 时，`lastAssistantMessage` 使用任务名称和 prompt 的只读摘要。

### Session meta DTO 所有权

- Recall 任务计划把 `AgentSessionMeta(uri, name)` 放在
  `agent-context-prefix-contract`。
- tool contract 定义自己的 `SuggestedSessionMeta(uri, name)`。
- 两个 DTO 使用相同的 URI、name 事实来源和 wire shape，但不直接共享 Kotlin 类型。
- 不让 `tool-suggest-subagent-task-contract` 反向依赖 prompt-context contract。
- 不为两个字段新增低层公共 contract 模块。

### 自动标题

- 当前自动标题只通过 `Session <number>` 模式判断名称是否仍为默认值。
- 工具任务名称是显式最终 Session name，即使恰好为 `Session 42` 也不能被首条 prompt
  的自动标题覆盖。
- 在当前进程中，批量创建路径可以在异步提交 prompt 前调用
  `AgentTitleGeneration.suppress()`，且不需要伪造 rename、trim 或改写任务名称。
- 但 `AgentTitleGeneration.consumed` 只存在于 ViewModel 内存。
- Session 关闭并重开后，该状态重置；如果显式任务名称恰好匹配 `Session <number>`，
  后续 user message 仍可能触发自动标题并覆盖它。
- 因为任务名称允许任意内容且不做校验，不能通过禁止默认名称模式规避该问题。
- 已决定接受重开后的该边界，不持久化自动标题 eligibility/consumed 状态。
- 创建进程内仍必须在提交初始 prompt 前调用 `suppress()`，保证初始任务不会立即改写
  用户确认的名称。

### URI 交付顺序

- 本工具结果依赖 `KodexAgentStorage.uri` 和 `AgentSessionMeta(uri, name)`。
- 关联 Recall 任务已完成 Storage URI 迁移，并已移入 done。
- 实现顺序固定为先完成 Recall 的 Storage URI 迁移，再实现本工具。
- 本工具不增加临时 URI encoder，也不复制 Session identity 迁移。
- Recall 提供 URI 事实来源；本工具自己的 `SuggestedSessionMeta` 只负责 function
  output contract。

### 多个 host-owned pending call

- 当前 tool runtime 会先完成同一响应中的普通工具，使单个 host-owned call 最终成为唯一
  pending call。
- 当前专用交互和 Stop hook 都只识别“恰好一个”`request_user_input` pending call。
- 模型仍可能在同一响应中发出：
  - 多个 `suggest_subagent_task`。
  - 一个 `request_user_input` 和一个 `suggest_subagent_task`。
  - 多个 `request_user_input`。
- 这些组合经过普通工具处理后仍会留下多个 host-owned call；现有专用面板不会显示，
  runtime 也不会自行恢复，因此会形成无法由正常交互完成的卡死状态。
- 仅在 tool description 中要求模型不要这样调用，不能构成宿主侧状态保证。
- 已决定按模型输出顺序逐个处理 host-owned call。
- Agent 层从 `pending.events` 中选择第一个 host-owned call，并且只激活其对应的专用
  ViewModel；完成后恢复 runtime，再暴露下一个。
- 普通工具仍由 tool runtime 先行完成；顺序约束只适用于剩余的 host-owned calls。
- Stop hook 的 `Finish` 保留整组 pending calls，并让 UI 从第一项开始处理。
- Stop hook 的 `Stop` 或 `Continue` 必须把当时剩余的全部 host-owned calls 完成为普通
  failure；不能只移除第一项并留下仍可交互的同 turn 调用。
- `Continue` 清空这些 pending calls 后再注入 hook fragments。

### Tool hooks

- 当前 `request_user_input` 没有 runtime `Tool` 实现；它由 Agent ViewModel 直接调用
  `completeToolCall`，因此不经过 `KodexToolRuntime.handleToolCall`。
- 只有进入 `handleToolCall` 的普通工具才执行 `PreToolUse` 和 `PostToolUse`。
- 如果 `suggest_subagent_task` 完全复用该 host-owned 路径，它也不会触发 Tool hooks。
- 与 `request_user_input` 不同，接受建议会创建多个 Session，存在明确的宿主副作用。
- 已决定不触发 `PreToolUse` 或 `PostToolUse`。
- 用户确认本身是创建 Session 的完整授权边界。
- Stop hook 仍按前述规则运行；不增加专用于 host-owned 工具的 Tool hook 路径。

### Remote compaction retention

- 当前 remote compaction 的 64,000-token retained window 只保留：
  - `StableUserMessage`。
  - 完成的 `StablePlanUpdate`。
  - 完成的 `StableRequestUserInputToolEvent`。
- `suggest_subagent_task` 如果不加入 retained items，compaction 后主 Agent 会失去建议内容、
  用户决定、反馈和新 Session URI。
- 如果完整保留，事件必须像其他 completed tool event 一样不可拆分，并同时保留 function
  call arguments 与 output。
- 本工具不限制任务数量和 prompt 长度，因此一个批次可能占用 retained window 的大量
  token，甚至因单项超过预算而完全无法保留。
- 已决定完整保留 completed event。
- `StableSuggestSubagentTaskToolEvent` 实现 `CompactionRetainedItem`。
- retention 同时保留原始 tasks、用户决定、反馈和 accepted Session meta，并作为一个
  不可拆分项参与既有 64,000-token 预算。
- 如果单个事件自身超过剩余或总预算，沿用现有不可拆分项的排除语义，不增加截断或摘要
  协议。

### 首条 prompt 的持久化竞态

- 当前方案先创建并公开全部 Tab，再在 Application owner scope 异步调用各
  `rootAgent.submit(prompt)`。
- `AgentViewModel.submit` 进入 Agent command mutex 后才把 user message 写入 storage。
- Application command 释放后，用户可以立即切换到新 Tab 并提交输入；该输入与后台
  task prompt 竞争 Agent command mutex。
- 如果用户输入先获得锁，它会成为 Session 的第一条 user message，违反
  “task prompt 是首条 user message”。
- 如果进程在 source tool 已完成、后台 submit 尚未落盘时退出，该 Session 会永久缺少
  task prompt；现有 pending 恢复和重复创建边界都无法补偿。
- 已决定接受该竞态与丢失窗口。
- 不把 prompt 持久化加入 Session 创建或 accepted 完成条件。
- 不增加 startup gate、预留消息或提交队列。
- 正常调度仍在公开 Tab 后立即无界并发调用 `rootAgent.submit(prompt)`；该 prompt
  通常是第一条 user message，但协议不保证它一定先于用户在新 Tab 中手工提交的输入。

## 下一阶段实现路线

1. 先完成 Recall 任务中的 Storage URI 迁移。
2. 增加 suggest tool contract、fixed spec 和 settings-based visibility。
3. 增加 typed pending event、stable completed event 和 history projection。
4. 扩展 Stop hook，使两种 host-owned pending tool 使用相同流程。
5. 增加建议交互 ViewModel 和 Mosaic 确认面板。
6. 增加 Application-owned 批量 Session 创建与 Tab 安装命令。
7. 在 Application owner scope 无界并发提交首条 prompt。
8. 补齐 serialization、token accounting、history、ViewModel 和交互测试。

## 具体实现切片

- `tool/multi-agent/contract`：
  - `SuggestSubagentTaskArgs`、`SuggestedSubagentTask`。
  - `SuggestSubagentTaskResponse`、`SuggestSubagentTaskDecision` 和
    `SuggestedSessionMeta`。
  - `PendingSuggestSubagentTaskToolEvent` 与 stable completed result 所需的共享模型。
- `tool/multi-agent/impl`：
  - fixed `ToolSpec` 与 JSON schema。
  - 不提供可执行 `Tool`，保持 host-owned pending 路径。
- `agent-state/tool` 与 `agent-storage/clean-models`：
  - AskUser/NoQuestion 的 visible spec 投影。
  - pending/stable event projection、failure projection 和 function output serialization。
  - `CompactionRetainedItem` 与 token accounting。
- `agent-runtime/decorator/turn-hook`：
  - 将多个 host-owned pending call 按顺序处理。
  - Stop/Continue 对剩余 host-owned call 的 failure 处理。
- `app/contract/agent`、`app/viewmodel/agent` 与 `app/view`：
  - 建议 pending state、revision/submission state、批量配置草稿和确认面板。
  - 只激活当前 pending 队列中的第一个 host-owned call。
- `app/contract/application`、`app/viewmodel/application` 与 Session registry：
  - Application-owned 批量创建、Tab 批量安装、原 selected index 保留。
  - 每个 Session 使用普通初始化，返回按任务顺序排列的
    `SuggestedSessionMeta`。
  - Application owner scope 无界并发提交 prompt。
- `AgentTitleGeneration` integration：
  - 创建进程内 suppress 初始 prompt 的自动标题。
  - 接受已确认的重开后默认名称匹配边界。
- 测试：
  - contract serialization/schema。
  - pending close/reopen、revision、多个 host call 和 Stop hook。
  - batch tab ordering/config/meta、prompt launch 和 child failure。
  - compaction retention、token count、normal session behavior。

## 验收条件

- `AskUser` 同时暴露两个 host-owned 工具；`NoQuestion` 同时隐藏。
- pending 建议关闭并重开 Tab 后仍存在，未提交 UI 草稿不持久化。
- 同一响应中的多个 host-owned calls 按模型输出顺序逐个展示和完成。
- 拒绝不创建 Session，并以 `success=true` 返回显式 `decision` 和可空反馈。
- 接受返回与任务同序的 `uri + name`，且显式输出 `feedback: null`。
- 每个新 Session 使用用户最终选择的共享 model 配置、cwd 和
  `RequestUserInputMode`。
- 所有新 Session Tab 按任务顺序打开，源 Session 保持选中。
- 工具结果不等待首条 prompt 持久化、模型首包或执行完成。
- 首条 prompt 通过普通 Agent runtime 无界并发提交。
- 子 Session 失败只使用其普通 Tab 通知，不向源 Session 自动回流。
- Stop hook 的 Finish、Stop 和 Continue 与 `request_user_input` 一致。
- 本工具不触发 `PreToolUse` 或 `PostToolUse`。
- completed event 作为不可拆分项参与 remote compaction retention。
- 显式任务名称不会被初始 prompt 的自动标题覆盖。
- 不出现旧版 Agent Tree、父子身份或跨 Session 控制工具。

## Implementation record

- 已实现 `suggest_subagent_task` contract、fixed tool spec、typed pending event 和
  stable history event。
- 已接入 AskUser/NoQuestion visibility、多个 host-owned pending call 顺序仲裁、
  Stop hook failure、history projection 和 compaction retention。
- 已接入建议交互 ViewModel 与确认面板；模型配置、cwd 和
  `RequestUserInputMode` 由用户统一选择。
- 已接入 Application 批量创建普通 root Session、按顺序打开 Tab、保持源 Tab 选中，
  并无界并发提交首条 prompt。
- 已按已确认边界保留：不等待异步 prompt、不提供幂等 ledger、不自动回流结果、不触发
  Pre/Post Tool hooks。
- 验证通过：
  - `:tool-multi-agent-impl:jvmTest`
  - `:agent-storage-clean-models:jvmTest`
  - `:agent-state-tool:jvmTest`
  - `:agent-runtime-decorator-turn-hook:jvmTest`
  - `:app-viewmodel-agent:jvmTest`
  - `:app-viewmodel-application:jvmTest`

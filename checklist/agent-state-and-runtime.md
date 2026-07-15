# AgentState 与 AgentRuntime

修改 `agent-state` 或引入 `agent-runtime` 时遵守以下决策。

- AgentStorage只保存数据并维护存储后端，不承载agent编排。
- 只有AgentStorage区分只读与可变接口；AgentState保留完整原子操作，AgentRuntime只暴露`state`、`latestIndex`、只读`storage`和`resume`。
- AgentState只提供可校验的原子会话操作，不执行环境副作用。
- 一次AgentState.requestResponseApi()只发起一次Responses API请求。
- 自动上下文压缩和`end_turn == false`续跑由基础AgentRuntime处理。
- 基础CodexAgentRuntime私有持有完整AgentState，并在`resume`中处理自动上下文压缩和续跑。
- CodexAgentLoopImpl不执行工具；ToolPending由更外层runtime接手。
- 工具、hook、skill、AGENTS.md和外部交互通过AgentRuntime装饰器编排。
- AgentState以sealed的CodexAgentStateValue和热StateFlow发布状态；除ToolPending(calls)外均为data object。瞬态状态通过CAS抢占，冲突直接抛异常。
- CodexAgentSettings持有非空UUIDv7 turnId；AgentState只在开始新逻辑轮次时轮换它，所有上下文压缩走remote compaction v2。
- 普通Codex请求通过OpenAiClient.createResponse(CodexResponsesRequest)扩展投影；传输原语显式要求installationId、turnMetadata和windowId，不使用extraHeaders。remote compaction v2的beta header由client内部固定。
- ToolPending携带当前未完成调用的有序快照，供原子校验和路由使用；storage仍是持久化真源，重建状态时从活动history尾部推导。
- AgentState不公开通用的ResponseItem追加操作；用户消息和完整工具调用批次分别通过语义原语写入。
- 工具调用按单个结果完成；每个结果必须匹配当前待处理调用，未完成调用仍保持ToolPending。update_plan由外层显式走appendPlanUpdate，并在该操作中与plan timeline同一事务提交。
- 用户强制压缩是AgentState扩展；上下文上限自动压缩是Runtime内部行为，CodexAgentRuntime不暴露任何压缩操作。
- storage提交完成后才能发布新的稳定状态；已发布历史不因取消回滚。
- AgentContextInjection是创建CodexAgentState时必传的静态结构化上下文，不读取AgentState、storage或history。它始终携带EnvironmentContext；开发者指令可缺省，空skill或AGENTS.md列表分别表示不注入对应内容。
- `agent-context:render`负责将AgentContextInjection投影为临时请求前缀；环境、日期、时区和shell始终渲染。`agent-context:prompt-dsl`提供覆盖全部host target的XML-shaped prompt DSL，不依赖Koog。
- CodexAgentState在每次正常Responses请求开始时投影同一份AgentContextInjection；这些数据不写入history，也不参与compaction。需要持久化的上下文由对应的AgentState原子操作显式写入。
- Runtime装饰器只围绕`resume`和工具边界编排，不要求调用者处理压缩。
- 运行中干预、pending input、mailbox和stop hook属于Runtime协议；在实现前单独定义其admission与delivery规则。

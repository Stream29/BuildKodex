# AgentState 与 AgentRuntime

修改 `agent-state` 或引入 `agent-runtime` 时遵守以下决策。

- AgentStorage只保存数据并维护存储后端，不承载agent编排。
- 只有AgentStorage区分只读与可变接口；AgentRuntime继承完整的AgentState原子操作，并以`resume`增加多步编排。
- AgentState只提供可校验的原子会话操作，不执行环境副作用。
- 一次AgentState.requestResponseApi()只发起一次Responses API请求。
- 自动上下文压缩和`end_turn == false`续跑由基础AgentRuntime处理。
- 基础CodexAgentRuntime通过Kotlin委托复用同一份AgentState，并在`resume`中处理自动上下文压缩和续跑。
- CodexAgentCompactionRuntime不执行工具；ToolPending由更外层runtime接手。
- CodexAgentState.compactionRuntime()创建基础CodexAgentCompactionRuntime；调用方通过CodexAgentSettings.tools声明模型可见工具，runtime不注册ToolSpec。普通本地工具统一由CodexToolRuntime根据传入Tool的spec路由并完成匹配调用，未匹配调用保留；PlanRuntime保持专用路径，因为update_plan需要与plan timeline同一事务写入。
- 工具、hook、skill、AGENTS.md和外部交互通过AgentRuntime装饰器编排。
- AgentState以sealed的CodexAgentStateValue和热StateFlow发布状态；除ToolPending(calls)外均为data object。瞬态状态通过CAS抢占，冲突直接抛异常。
- CodexAgentSettings持有非空UUIDv7 turnId；AgentState只在开始新逻辑轮次时轮换它，所有上下文压缩走remote compaction v2。
- 普通Codex请求通过OpenAiClient.createResponse(CodexResponsesRequest)扩展投影；传输原语显式要求installationId、turnMetadata和windowId，不使用extraHeaders。remote compaction v2的beta header由client内部固定。
- ToolPending携带当前未完成调用的有序快照，供原子校验和路由使用；storage仍是持久化真源，重建状态时从活动history尾部推导。
- AgentState不公开通用的ResponseItem追加操作；用户消息和完整工具调用批次分别通过语义原语写入。
- 工具调用按单个结果完成；每个结果必须匹配当前待处理调用，未完成调用仍保持ToolPending。update_plan由外层显式走appendPlanUpdate，并在该操作中与plan timeline同一事务提交。
- 用户强制压缩是AgentState原子操作；上下文上限自动压缩是Runtime内部行为，调用`resume`不要求调用者预先处理压缩。
- storage提交完成后才能发布新的稳定状态；已发布历史不因取消回滚。
- AgentContextPrefixProvider是创建CodexAgentState时必传的动态结构化前缀来源。它以无默认只读`val`暴露环境、AGENTS.md和skill catalog；getter可返回当前值，但不读取AgentState、storage或history。
- `agent-context:collaboration:render`渲染内置`ModeKind`的固定 developer instructions；`Default`注入 Rust 对齐的`update_plan`使用指引，`Plan`使用其专属 developer instructions 并禁止调用`update_plan`。`UpdatePlanArgs`和`ThreadGoal`只保留为 settings 状态，不参与提示词投影。
- CodexAgentState在每次正常Responses请求开始时先投影当前`ModeKind`的 developer message，再读取并投影AgentContextPrefixProvider的当前值；二者均不写入history，也不参与compaction。需要持久化的上下文由对应的AgentState原子操作显式写入。
- `agent-context`只规定结构化 context contract 与通用prompt DSL；provider的具体数据加载、注入时机、消息角色/顺序和持久化交付均属于未来AgentRuntime。不要在`agent-context`中实现依赖runtime能力的上下文注入。
- Runtime装饰器通过Kotlin委托围绕`resume`、工具边界和需要增强的AgentState原子操作编排，不要求调用者处理自动压缩。
- 不引入仿Rust的固定`TurnRunner`。一次`AgentRuntime.resume()`是runtime自行编排的turn单元；各runtime可定义该次运行的中止、继续和流转条件，不将这些条件固化为全局turn runner。
- 运行中干预、pending input、mailbox和stop hook属于Runtime协议；在实现前单独定义其admission与delivery规则。

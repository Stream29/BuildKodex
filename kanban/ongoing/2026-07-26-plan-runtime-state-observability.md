# Task Tree

- 规划Agent与Runtime状态上报及观测系统
  - 与用户确定StateProvider契约、作用域、组合方式和生命周期
  - 确定StateFlow当前快照与操作生命周期事件的边界
  - 定义AgentState原子操作与Responses请求的细分状态
  - 盘点并定义各AgentRuntime装饰层的局部状态
  - 定义Shell进程、Unified Exec session、MCP连接与其他长生命周期资源状态
  - 确定状态聚合、关联标识、并发一致性、隐私与运行时重建语义
  - 确定CLI消费方式及现有推断式activity/running状态的迁移边界
  - 形成经用户审核的实施顺序、验证方案和后续任务拆分

# Details

- 状态：`await planning`。
- 等待用户未来明确启动；当前只保留已收集的上下文，不继续设计或实现。
- 本任务不预设StateProvider的具体API、注册表或provider tree方案，也不将候选状态视为已确认设计。

## 当前观测基线

- `CodexAgentStateValue`同时包含稳定会话状态和`ExternalWrite`、`RequestResponse`、`Compacting`三个粗粒度瞬态状态。
- 一次Responses请求内部实际包含快照读取、context prefix解析、动态tool spec解析、HTTP/SSE传输、output item提交和usage提交，目前全部折叠为`RequestResponse`。
- `CodexToolRuntime`在路由、PreToolUse、handler、PostToolUse和output持久化期间，AgentState一直保持同一个`ToolPending`。
- 当前持久runtime链包含compaction、session hook、plan、skill selection、MCP catalog、request user input、multi-agent tool、turn hook和coordinator层；MCP层每次`resume`还会创建临时`CodexToolRuntime`。
- `SteerRuntime`已维护私有FIFO，但尚未安装进当前CLI runtime链。
- `RequestUserInputRuntime.pendingRequests`已是runtime自有StateFlow的先例，但只发布已完成预处理的等待项。
- `MultiAgentCoordinator`已同时提供agent快照StateFlow和turn事件SharedFlow，但`Running`尚未区分排队、等待permit、正在运行和`wait_agent`等待。
- `McpClientManager.catalog`已发布catalog generation以及`Available`、`Degraded`、`Unavailable`，但不发布连接、刷新、lease、retire和close过程。
- Hook当前只公开业务窄端口；Hook command process尚无运行状态出口，未来由统一资源层观测覆盖。
- `ToolHookCoordinator`私有保存长时间`exec_command`到后续`write_stdin`的deferred PostToolUse关联。
- `AgentTurnContext`已有私有`Unresolved`/`Resolved`状态；AGENTS.md和skill的发现警告目前没有统一观测出口。
- `ToolSearchCatalog`已内部使用StateFlow保存documents，并延迟重建BM25 index，但catalog generation、index stale/building/ready和search活动不可见。
- `UnifiedExecToolClient`已维护私有session registry；`ShellClient`拥有并关闭所有`ProcessSession`，两者都未发布active session、进程终态或正在进行的stdin/output交互。
- CLI目前直接将Responses SSE事件投影成英文`activity`字符串，并从coordinator status、response Job和settled deferred三处推导`running`。

## 候选上报面

- AgentState：稳定会话状态、类型化external write、Responses请求阶段和compaction core阶段。
- AgentRuntime：围绕`resume()`的自动compaction、请求续跑、工具路由与执行、plan、request user input、hooks、skill/context、MCP catalog lease、multi-agent调度与steer队列。
- 资源层：ShellClient生命周期、ProcessSession、Unified Exec session映射、MCP connection/lease/call、Hook command process和tool search index。
- 应用层：OpenAI transport/retry、model catalog refresh、runtime generation替换和CLI组合观测。

## 规划约束

- AgentState必须继续表达原子会话不变量，不把环境副作用或所有runtime状态塞入一个总枚举。
- `CodexAgentState.state`已占用`StateFlow<CodexAgentStateValue>`；新的通用provider契约需要避免与这个属性发生不同类型的同名冲突。
- 当前Kotlin委托链只保留最外层runtime接口，多数中间层在组装后不可访问；规划需要显式解决provider发现与组合。
- StateFlow只承载当前快照；不能依赖它保留每一个快速状态过渡或终态结果。
- 需要区分application、session/agent、runtime generation、turn、operation和resource作用域，并使用agentId、turnId、operationId/requestId、callId、process/sessionId等稳定关联标识。
- runtime重建时需原子替换provider generation，关闭旧资源并避免残留幽灵状态。
- 默认上报不应包含access token、完整prompt、完整工具参数、stdin/stdout或命令原文；详细数据需要独立的受控调试通道。
- 纯计算组件、内部mutex/buffer和无独立生命周期的短工具不需要单独provider，可由上层工具执行状态覆盖。

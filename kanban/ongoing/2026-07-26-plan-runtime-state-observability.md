# Task Tree

- 规划Agent与Runtime状态上报及观测系统
  - [done] 定位当前前端渲染与Agent层能力的错位
    - [done] 确认`SessionSnapshot`混合持久会话、完整history、stream文本、英文activity和running推断
    - [done] 确认CLI从Responses SSE构造展示字符串，并从Job、deferred和AgentState推断运行状态
    - [done] 确认`KodexAgentStateValue`只表达原子操作合法性，不能承担完整运行观测
  - 确定Agent可渲染能力契约
    - [done] 定义只含类型化事实的Agent execution snapshot，不在Agent层输出终端文案或布局数据
    - [done] 将一次Responses request建模为按output item切换的Request子状态；每个子状态有自己的无限replay流
    - 定义当前快照、可丢失的生命周期事件和已提交history/stream tail三种不同数据面
    - 定义AgentState、Runtime decorator、资源和CLI各自可发布的能力边界
  - 确定运行操作模型
    - 为Responses、compaction、工具、request-user-input、steer、multi-agent调度和title generation定义类型化阶段与稳定关联ID
    - 明确并发操作、取消、失败、恢复、runtime generation替换和资源关闭的快照语义
    - 为Shell、Unified Exec、MCP、hook与tool-search界定是否需要独立资源状态及其隐私边界
  - 确定组合与消费方式
    - 设计可发现的provider组合方式，避免Kotlin runtime委托链组装后丢失中间层状态
    - 将SessionManager改为转发类型化Agent capability，而非维护平行Job/deferred/activity真源
    - 让Mosaic按状态类型渲染本地化文案、控件可用性、stream和阻塞交互，不再解释SSE或拼装activity字符串
  - 确定迁移与验证顺序
    - 先建立最小端到端Responses、tool pending与request-user-input能力，再逐层接入runtime和资源状态
    - 覆盖多Agent并发、取消、失败、恢复、runtime替换和快速事件丢失场景
    - 为每个已迁移renderer删除旧推断路径，并验证没有重复状态真源

# Details

- 状态：规划已启动，尚未进入实现；后续节点不构成自动实施授权。
- 本次规划的目标是移除前端对Agent执行过程的推断，不是把当前`SessionSnapshot`换一个名字或把展示字符串下推到Agent层。
- Unified Exec已获用户确认：直接通过`AgentRuntime.unifiedExecToolClient`暴露当前runtime持有的client；其余provider的公开形态和首批覆盖范围仍待确认。
- 本阶段保留`AgentState.requestResponseApi(): Flow<ResponsesStreamEvent>`和`AgentRuntime.resume()`的现有事件流签名，供runtime编排和既有调用方使用；前端迁移不依赖也不消费这条返回流。

## 已确认的问题

- `SessionSnapshot`同时携带会话配置、完整`conversationHistory`、流式文本、`activity: String?`和`running: Boolean`；任何一个流式事件都迫使同一个展示快照变化。
- `SessionManager`在启动响应、取消和SSE消费时直接写入`"Requesting response..."`、`"Cancelling response..."`等英文展示文本；`HistoryView`再把它包装成history item。
- `ManagedAgent.isRunning`同时读取response Job、settled deferred和`KodexAgentStateValue`。这使CLI拥有一套与runtime并行的执行状态真源。
- `KodexAgentStateValue`的职责是保护原子会话转换：`RequestResponse`、`Compacting`和`ToolPending`不足以说明请求的具体阶段、工具执行、外部等待或资源交互。
- `requestResponseApi()`当前返回的原始事件流仍由runtime用于读取`Completed.response.endTurn`并续跑；它不属于前端的观测契约，且本阶段不改变该返回设计。
- 因此renderer不得继续从SSE、协程生命周期或多个布尔值重建语义；这些事实应由其所属的Agent、Runtime或资源层发布。

## 拟定目标模型

- AgentState继续发布会话原子状态和持久化边界，不把所有执行细节塞进`KodexAgentStateValue`。
- 当前Responses请求以`KodexAgentStateValue.RequestResponse`子类型公开。Started、reasoning、message、agent-message和tool-call阶段各有自己的类型化状态；不同工具调用类型统一聚合为`ToolCall`，只有未建模协议项进入`Unknown`。每个活跃output子状态持有自己的无限replay流，使任何在该阶段仍活跃时订阅的frontend都能从阶段开头重放并自行投影。
- 一个request只持有当前output子状态；前一output在`OutputItemDone`落盘后立即回到Started并释放其flow，前端从`latestIndex`界定的storage history读取已提交内容。
- 其他需要顺序消费的开始、阶段变化、完成和失败通过独立事件流发布；UI不得仅靠短窗口或可丢失事件流恢复当前状态。
- Runtime decorator只发布自己拥有的操作；组合根负责以稳定Agent identity和runtime generation汇总，而非扫描或猜测内部委托链。
- 资源状态归资源owner发布。没有独立生命周期的纯计算工具不建provider；状态中默认不包含prompt、token、命令、stdin/stdout或完整工具参数。
- SessionManager仅维护Session/Agent地址、选择和生命周期，并将Agent capability投影给CLI。Mosaic负责终端文案、排序和布局，不再维护协议到activity字符串的映射。

## 建议的首批垂直切片

- Responses：请求已排队、准备上下文、传输/流式输出、提交item、完成/取消/失败；每个output item以operation ID、output index/item ID和自己的无限replay类型化流公开，完成后以storage为准。
- Local tools：从`ToolPending`派生待处理调用，并由Tool Runtime公布路由、执行、等待外部输入和输出提交阶段；保留call ID。
- `request_user_input`：由其Runtime发布结构化问题、答案草稿/自动解决倒计时和完成状态，前端不再识别特定函数调用来构造dialog。
- Multi-agent：在既有拓扑快照之上增加排队、等待permit、执行和`wait_agent`等待的类型化状态，不改变Coordinator的执行所有权。
- 其它资源和hook不进入首批切片；先完成其状态owner和ID/隐私契约，再独立接入。

## 待确认决策

- 除已确认的`AgentRuntime.unifiedExecToolClient`外，provider是否以一个显式的组合根和稳定`AgentExecutionCapability`公开，还是允许每个runtime decorator直接暴露独立provider并由CLI聚合。
- 首批是否严格限于Responses、local tools、`request_user_input`和multi-agent，排除MCP、Shell、hook和tool-search资源观测。
- CLI状态是否仅保留展示本地状态（选择、展开、草稿、焦点、overlay），并在本任务中同步移除`SessionSnapshot.activity`、本地`running`和SSE展示映射。

## 当前观测基线

- `KodexAgentStateValue`同时包含稳定会话状态和`ExternalWrite`、`RequestResponse`、`Compacting`三个粗粒度瞬态状态。
- 一次Responses请求内部实际包含快照读取、context prefix解析、动态tool spec解析、HTTP/SSE传输、output item提交和usage提交，目前全部折叠为`RequestResponse`。
- `KodexToolRuntime`在路由、PreToolUse、handler、PostToolUse和output持久化期间，AgentState一直保持同一个`ToolPending`。
- 当前持久runtime链包含compaction、session hook、plan、skill selection、MCP catalog、multi-agent tool、turn hook和coordinator层；MCP层每次`resume`还会创建临时`KodexToolRuntime`。
- `SteerRuntime`已维护私有FIFO，但尚未安装进当前CLI runtime链。
- `request_user_input`不建立runtime状态；UI从AgentState的单个`ToolPending`调用直接投影表单并完成调用。
- `MultiAgentCoordinator`已同时提供agent快照StateFlow和turn事件SharedFlow，但`Running`尚未区分排队、等待permit、正在运行和`wait_agent`等待。
- `McpClientManager.catalog`已发布catalog generation以及`Available`、`Degraded`、`Unavailable`，但不发布连接、刷新、lease、retire和close过程。
- Hook当前只公开业务窄端口；Hook command process尚无运行状态出口，未来由统一资源层观测覆盖。
- `KodexToolRuntime`直接执行Tool Hooks，并保存长时间`exec_command`到后续`write_stdin`终态的必要关联。
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
- `KodexAgentState.state`已占用`StateFlow<KodexAgentStateValue>`；新的通用provider契约需要避免与这个属性发生不同类型的同名冲突。
- 当前Kotlin委托链只保留最外层runtime接口，多数中间层在组装后不可访问；规划需要显式解决provider发现与组合。
- StateFlow只承载当前快照；不能依赖它保留每一个快速状态过渡或终态结果。
- 需要区分application、session/agent、runtime generation、turn、operation和resource作用域，并使用agentId、turnId、operationId/requestId、callId、process/sessionId等稳定关联标识。
- runtime重建时需原子替换provider generation，关闭旧资源并避免残留幽灵状态。
- 默认上报不应包含access token、完整prompt、完整工具参数、stdin/stdout或命令原文；详细数据需要独立的受控调试通道。
- 纯计算组件、内部mutex/buffer和无独立生命周期的短工具不需要单独provider，可由上层工具执行状态覆盖。

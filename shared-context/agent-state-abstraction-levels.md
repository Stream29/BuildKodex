# Agent State Abstraction Levels

## AgentStorage

- AgentStorage只负责保存对话相关数据，并由具体实现负责与存储后端交互。
- AgentStorage默认是只读接口；MutableAgentStorage才暴露原子写入操作。
- AgentStorage不持有agent编排、工具执行、自动上下文压缩或注入策略。
- 已发布的数据不可变，并以从0开始的稀疏整数下标进行版本管理。
- 所有写入必须具备强异常安全：操作抛出异常时不得留下部分可见的修改。
- 通过copy和forkUntilInclusive派生历史版本；不提供破坏性truncate。

### 原始数据和干净数据

- 原始数据对齐LLM厂商协议；干净数据面向UI和跨provider语义。
- 两者使用独立的下标空间，并通过显式映射关联。
- 一个原始数据可派生零个、一个或多个连续的干净数据；干净数据不要求无损还原原始数据。
- 原始数据可保存完整上下文、上下文压缩信息和厂商专有字段。

## AgentState

- AgentState是对AgentStorage的行为包装器，提供不会破坏会话状态的原子操作。
- AgentState不区分只读和可变版本；它本身就是完整的原子操作接口。
- AgentState可以暴露底层AgentStorage的只读视图，绝不暴露MutableAgentStorage。
- AgentState负责单步LLM请求、单步远程压缩、追加输入、追加工具结果等原子状态转换。
- 单步LLM请求只发起一次Responses API调用；流式收到的完成item按item原子写入storage。
- AgentState不决定是否自动压缩，不执行`end_turn == false`后的续跑，不执行工具，也不处理skill、AGENTS.md或hook。
- 取消或失败不会回滚已发布历史；每个已提交item都必须保持storage合法。
- 调用方不得绕过AgentState直接修改storage，否则会破坏状态机和缓存语义。

### 状态机

- AgentState对外以sealed的CodexAgentStateValue和热StateFlow发布状态，状态反映当前允许的原子操作，而不只是UI标记。
- 稳定状态至少区分Empty、UserMessage、AssistantMessage、ToolPending和ToolCompleted。
- LLM请求中的短暂状态区分RequestResponse与Compacting。
- 只有稳定状态可以开始新的原子操作；请求开始前发布in-flight状态，storage提交成功后才发布下一个稳定状态。
- 除ToolPending(calls)外，状态值均为data object。ToolPending携带当前未配对调用的有序快照，用于拒绝不匹配或重复的工具结果；storage仍是持久化真源，状态重建时从活动history尾部推导。
- 状态转换应拒绝非法操作，例如在ToolPending时追加新用户消息，或在请求进行中再次请求模型。

## AgentRuntime

- AgentRuntime是对agent run和环境副作用的编排抽象，不将其限制为单纯的loop。
- AgentRuntime公开面只有`resume`以及底层AgentState的`state`、`latestIndex`和只读`storage`；不暴露完整AgentState或压缩操作。
- 基础CodexAgentLoopImpl私有持有完整AgentState，并在`resume`中处理自动上下文压缩和`end_turn == false`续跑。
- 基础runtime不执行工具；遇到ToolPending时将控制权交给更上层runtime。
- 工具执行、审批、skill、AGENTS.md、hook和外部交互都属于更外层的AgentRuntime能力。
- Runtime的能力优先通过装饰器叠加；每层在下一层的resume或工具边界前后织入自己的逻辑。
- 只有无法由runtime装饰器实现的状态内不变量，才在AgentState上增加窄的原子操作。
- 自动压缩是Runtime内部行为；调用`resume`的代码和更外层Runtime均不需要处理它。

## Context Injection

- 对话上下文按来源区分为generated context和injected context。
- generated context来自user、assistant和tool，通常持久化在AgentStorage中。
- injected context由运行时提供；默认每次请求临时拼接，不写入AgentStorage。
- 若确实需要跨请求延续的注入，必须显式作为durable injection写入，不能把所有注入都当作历史基线。
- ContextInjectionProvider属于ContextRuntime：它产生注入计划，不直接修改AgentStorage，也不是AgentState的通用依赖。

## LlmProvider

- LlmProvider或OpenAiClient只对原始API调用、鉴权和连接资源建模。
- AgentState在其上提供类型化的会话原子操作；AgentRuntime在AgentState之上提供编排。

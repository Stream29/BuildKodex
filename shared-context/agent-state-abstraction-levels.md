# Agent State Abstraction Levels

## AgentStorage

- AgentStorage不持有自己的SafeRw；它是append-only storage。
- AgentStorage默认指不可变只读接口。
- 明确分区：AgentStorage里的数据，一部分是已经累积下来的稳定数据，另一部分是尚未commit的热数据。
- MutableAgentStorage才暴露产生更新的原子追加操作。
- AgentStorage必须提供copy和基于下标的只读forkUntilInclusive操作。
- AgentStorage不提供破坏性的truncate操作；历史截断必须通过forkUntilInclusive产生新的storage。
- AgentStorage必须提供原子性的append操作，使每次追加后storage都保持合法状态。
- AgentStorage所有已发布数据不可变。
- AgentStorage所有数据都有和下标绑定的版本管理，在forkUntilInclusive产生历史版本的时候行为正确。
- AgentStorage的所有操作必须保证强异常安全，如果抛出异常必须保证不产生数据修改等副作用。

### 原始数据和干净数据

- AgentStorage分为两部分：内部使用的、高度对齐LLM厂商的原始数据和对外暴露的、简单整齐的干净数据。
- 原始数据和干净数据各自都有基于从0开始的连续整数下标的列表。
- 原始数据下标和干净数据下标不是同一个index空间。
- 每个干净数据必须由一个原始数据派生。
- 一个原始数据可以派生零个、一个或多个干净数据。
- 同一个原始数据派生出的多个干净数据必须在干净数据列表中连续。
- 原始数据可能包含上下文压缩信息、厂商传来的推理签名等特殊信息，当然要包含完整上下文窗口。
- 原始数据的类型应当根据模型提供商不同而不同，只实现一个基本的公共接口，提供forkUntilInclusive这种基于index的基本操作。
- 原始数据和干净数据通过显式映射关系关联，干净数据不能被要求无损还原原始数据。
- 因为每种原始数据都有自己的类型，比如OpenAI的对话状态是特殊的，所以对应的storage也是特殊的。
- 干净数据包含的是所有provider都有的共性，比如user/assistant的message、工具调用、token计数。

## AgentState

- AgentState向外提供基本原子操作接口，维护每个操作后状态正确。
- AgentStorage不可能拒绝所有非法操作，但是AgentState必须拒绝所有非法操作。
- AgentState应当是lazy的，即默认不把全部数据加载到内存且释放不再使用的引用，需要的时候从AgentStorage加载。
- 可以手动预热AgentState以确保第一次`resume()`时不再需要额外从AgentStorage加载。
- AgentState持有conversation state并负责AgentStorage追加顺序。
- AgentState默认指不可变只读接口。
- MutableAgentState才暴露触发LLM调用、追加输入、forkUntilInclusive等会改变conversation state的操作。
- AgentState可以向外暴露底层AgentStorage的不可变只读视图。
- AgentState不向外暴露MutableAgentStorage。
- 允许在某些情况下从将来自另一个provider的干净数据导入新建的AgentState，根据干净数据尝试重建，但为了实现正确的跨provider上下文压缩，可能需要产生网络调用。
- AgentState不执行工具。
- AgentState负责LLM协议中的tool call和tool result建模。
- 我们可以在MutableAgentState上调用`resume()`，这是最重要的触发LLM调用的入口，且它的返回结果必然是一个Flow，里面装着干净数据，且遵循下标规则。
- `resume()`运行期间的turn处于pending状态，不属于已发布的AgentStorage历史。
- `resume()`不持有覆盖整个Flow的SafeRw写锁。
- `resume()`正常结束时通过AgentStorage.appendTurn原子提交本次turn。
- `resume()`失败或取消时不回滚已发布历史；失败或取消的turn由Agent负责通过AgentState原子写入AgentStorage。
- `resume()`发出的Flow数据在写入AgentStorage前是pending event，在提交后成为已持久化状态。
- 我们不可以绕过AgentState直接编辑底层AgentStorage，那样会导致缓存错乱。正确的方式是通过AgentState暴露出的append方法。
- AgentState将上下文压缩这层复杂度包装起来，使得调用方不需要知道有上下文压缩这回事，可以安全地执行LLM的上下文操作。

## Agent

- Agent可以看作是在AgentState外包了一层harness编排和工具调用的壳。
- Agent可以持有自己的SafeRw保护工具执行、审批、subagent编排、pending turn等harness状态。
- Agent不直接保护conversation state，也不直接编辑AgentStorage。
- Agent负责将失败或取消的pending turn通过AgentState原子提交为conversation state的一部分。
- 每个provider可能都会有一些特殊的工作，比如subagent，比如特定的工具集，这个也是Agent在做。
- 因为已经做好了工具调用，所以Agent应该是个状态机，但是不同Agent的状态机可能不太一样。

## LlmProvider

- LlmProvider对原始的API调用进行建模，是实现AgentState的底层依赖。
- 它应该是个承载了鉴权/登录状态的有状态内存对象。遵守RAII。
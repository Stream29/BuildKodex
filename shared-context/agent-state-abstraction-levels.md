# Agent State Abstraction Levels

## AgentStorage

- AgentStorage不持有自己的SafeRw；它由Agent保护并发访问。
- AgentStorage默认指不可变只读接口。
- MutableAgentStorage才暴露产生更新的append和truncate操作。
- AgentStorage必须提供copy和基于下标的只读fork操作。
- AgentStorage所有数据都有和下标绑定的版本管理，在fork被截断为历史版本的时候行为正确。
- AgentStorage的所有操作必须保证强异常安全，如果抛出异常必须保证不产生数据修改等副作用。

### 原始数据和干净数据

- AgentStorage分为两部分：内部使用的、高度对齐LLM厂商的原始数据和对外暴露的、简单整齐的干净数据。
- 原始数据和干净数据都有一个基于从0开始的整数下标的message列表。可能还包含其他属性，比如审计数据和元数据，但是也以某种方式和index产生关联。
- 原始数据可能包含上下文压缩信息、厂商传来的推理签名等特殊信息，当然要包含完整上下文窗口。
- 原始数据的类型应当根据模型提供商不同而不同，只实现一个基本的公共接口，提供fork这种基于index的基本操作。
- 原始数据通过下标关联和干净数据双向映射。
- 因为每种原始数据都有自己的类型，比如OpenAI的对话状态是特殊的，所以对应的storage也是特殊的。
- 干净数据包含的是所有provider都有的共性，比如user/assistant的message、工具调用、token计数。

## AgentLoop

- AgentLoop应当是lazy的，即默认不把全部数据加载到内存且释放不再使用的引用，需要的时候从AgentStorage加载。
- 可以手动预热AgentLoop以确保第一次`resume()`时不再需要额外从AgentStorage加载。
- AgentLoop不持有自己的SafeRw；它只能由Agent在受控流程中调用。
- AgentLoop默认指不可变只读接口。
- MutableAgentLoop才暴露触发LLM调用、追加输入、截断历史等会改变状态的操作。
- 允许在某些情况下从将来自另一个provider的干净数据导入新建的AgentLoop，根据干净数据尝试重建，但为了实现正确的跨provider上下文压缩，可能需要产生网络调用。
- AgentLoop只碰LLM API的调用，不碰工具调用。
- 我们可以在MutableAgentLoop上调用`resume()`，这是唯一的触发LLM调用的入口，且它的返回结果必然是一个Flow，里面装着干净数据，且遵循下标规则。
- 我们不可以绕过Agent直接编辑AgentLoop或底层AgentStorage，那样会导致缓存错乱。正确的方式是通过Agent暴露出的append方法。
- AgentLoop将上下文压缩这层复杂度包装起来，使得调用方不需要知道有上下文压缩这回事，可以安全地执行LLM的上下文操作。

## Agent

- Agent可以看作是在AgentLoop外包了一层harness编排和工具调用的壳。
- Agent是唯一并发入口，必须持有SafeRw保护整体可变状态。
- 每个provider可能都会有一些特殊的工作，比如subagent，比如特定的工具集，这个也是Agent在做。
- 因为已经做好了工具调用，所以Agent应该是个状态机，但是不同Agent的状态机可能不太一样。

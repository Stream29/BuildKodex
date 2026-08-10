# Task Tree

- [done] 为AgentState全部写入建立串行队列
  - [done] 确认`completeToolCall`与标题写入在`ExternalWrite` CAS准入处竞争
  - [done] 固化全部写入共用队列、状态校验时机与标题patch不变量
  - [done] 为每个AgentState实例加入覆盖全部写入的公平Mutex
  - [done] 将`requestResponseApi()`与compaction纳入同一写入临界区，移除其CAS竞争准入
  - [done] 让标题写入在队列内基于最新settings进行条件`threadName` patch
  - [done] 覆盖请求、compaction、工具完成、标题写入、settings/plan保留、取消等待者与非法状态的串行测试
  - [done] 运行受影响的AgentState和session-title Gradle测试

# Details

- 状态：已完成。
- 当前竞争位于`Kodex/agent-state/impl/src/commonMain/kotlin/io/github/stream29/kodex/agentstate/impl/KodexAgentStateImpl.kt:149`与`:567`：Responses请求直接CAS抢占，其他原子操作也会在短暂state下失败。
- 所有AgentState写入使用同一个Mutex：包括完整Responses流、remote compaction和所有短事务。写入在获锁后重新校验当前状态；不再把并发竞争本身当作错误。
- 标题写入不能继续把队列外读取的完整`KodexAgentSettings`提交给`updateSettings`；它必须在AgentState临界区读取当前值并只改变`threadName`，以保留先完成的`update_plan`等settings更新。
- 主要改动点：`Kodex/agent-state/contract/src/commonMain/kotlin/io/github/stream29/kodex/agentstate/contract/KodexAgentState.kt`、`Kodex/agent-state/impl/src/commonMain/kotlin/io/github/stream29/kodex/agentstate/impl/KodexAgentStateImpl.kt`、`Kodex/cli/session-title/src/commonMain/kotlin/io/github/stream29/kodex/cli/sessiontitle/AgentTitleGeneration.kt`及各自测试。
- 设计准则见[AgentState写入串行化](../../checklist/agent-state-mutation-serialization.md)。

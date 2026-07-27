# Task Tree

- [done] 复核AgentStorage组合回滚的原子性
  - [done] 明确跨timeline回滚遇到取消或后端失败时的语义
  - [done] 评估保留extension决策时可行的事务边界
  - [done] 确定持久化实现必须提供的底层能力
  - [done] 记录经用户确认的实施方案

# Details

本任务只完成方案复核，不在其他pending计划确认前修改实现。

最终决定采用单writer和操作级补偿，不提供compound write期间的跨timeline snapshot isolation。删除timeline与storage级`transaction`，改用可嵌套的`setWithTransaction`和`revertWithTransaction`，依靠Kotlin调用栈逆序补偿。

进程崩溃允许保留结构合法的持久化前缀，不引入Saga或durable transaction journal。完整决策记录在`shared-context/findings/agent-storage-compensation-semantics.md`。

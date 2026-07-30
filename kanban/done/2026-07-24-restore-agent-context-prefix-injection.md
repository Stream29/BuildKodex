# Task Tree

- [done] 恢复AgentContextPrefix负责请求前缀注入
  - [done] 删除KodexAgentState.requestResponseApi的transientInput参数
  - [done] 将结构化AgentContextPrefixProvider绑定到AgentState构造
  - [done] 让AgentTurnContext提供每轮冻结的完整prefix
  - [done] 移除CompactionRuntime与Multi-agent的原始HistoryItem前缀管道
  - [done] 更新CLI构造和runtime替换生命周期
  - [done] 更新测试与上下文设计文档
  - [done] 运行相关JVM与Linux Native验证

# Details

- Prefix只进入普通Responses请求，不写入history，也不进入remote compaction输入。
- 用户消息、显式skill正文和Hook持久化结果继续通过AgentState原子操作写入storage。
- AgentState只消费结构化provider并负责最终请求拼装，不向调用方开放任意HistoryItem注入口。
- Prefix、AgentState、CompactionRuntime、Multi-agent与相关Runtime测试已通过JVM和Linux Native验证。

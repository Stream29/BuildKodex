# Task Tree

- 决定剩余上下文组成成分的归属
  - [done] 盘点 Rust 请求组装中尚未对应到 Kotlin 的上下文来源
  - [done] 对照现有 `AgentContextInjection`、AgentState 与 runtime 边界
  - [done] 提出各组成成分的归属与注入时机
  - [done] 撰写上下文组成成分归属 finding
  - 经讨论后记录已确认的决策

# Details

本任务先讨论归属，不在未确认边界前扩展 API 或实现。

建议以请求配置、临时快照、持久化宿主事件、宿主 runtime 四类边界取代 Rust 的统一 world-state history；具体映射待用户确认。

# Task Tree

- [done] 暴露每个Agent runtime持有的Unified Exec client
  - [done] 确认前端需要从`AgentRuntime`取得同一份client
  - [done] 将client贯穿runtime composition并加入公开contract
  - [done] 验证每个Agent runtime拥有独立且可取得的client

# Details

- 用户已明确要求通过`AgentRuntime`暴露`UnifiedExecToolClient`，以便前端取得资源。
- 不在本任务中添加进程状态或stdout的观测模型。

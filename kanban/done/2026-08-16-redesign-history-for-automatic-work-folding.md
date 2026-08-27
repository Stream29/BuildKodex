# Task Tree

- [done] 重构 History 并自动折叠消息间工作记录
  - [done] 完成现状、性能风险与参考实现调研
  - [done] 确认既有 History 基础设施与独立 payload 边界
    - [done] [修复既有 History 的大历史扩展性](../done/2026-08-16-stabilize-existing-history-scalability.md)
    - [done] 将大型 MCP payload 渲染保留为独立任务
  - [done] [确定消息间工作记录的自动折叠语义](../done/2026-08-16-define-history-work-folding-semantics.md)
  - [done] [增加增量 History WorkGroup 投影](../done/2026-08-16-add-incremental-history-work-groups.md)
  - [done] [完成端到端大历史与折叠行为验证](../done/2026-08-18-end-to-end-verify-history-work-groups.md)

# Details

- 状态：`done`。自动折叠语义、增量投影、渲染交互和 release 端到端验证均已完成。
- 既有 History 扩展性修复先保持“一条 stable event 对应一行”，随后由独立的
  [`WorkGroup` 实现任务](../done/2026-08-18-implement-history-work-groups.md)引入自动折叠。
- 大型 MCP payload 冻结问题已由
  [`防止大型 MCP 工具结果冻结 CLI UI`](../done/2026-08-10-prevent-large-mcp-results-from-freezing-cli.md)
  完成；有界 `WorkGroup` summary 同样不读取 payload。
- release 二进制已使用包含 13,388 个 stable event 的真实 session 验证折叠、breaker、展开状态、
  旧端滚动和日志。

# Task Tree

- 重构 History 并自动折叠消息间工作记录
  - [done] 完成现状、性能风险与参考实现调研
  - 修复既有 History 行为
    - [修复既有 History 的大历史扩展性](2026-08-16-stabilize-existing-history-scalability.md)
    - [防止大型 MCP 工具结果冻结 CLI UI](2026-08-10-prevent-large-mcp-results-from-freezing-cli.md)
  - [确定消息间工作记录的自动折叠语义](2026-08-16-define-history-work-folding-semantics.md)
  - [增加增量 History WorkGroup 投影](2026-08-16-add-incremental-history-work-groups.md)
  - 完成端到端大历史与折叠行为验证

# Details

- 状态：`discussion`。本文件只维护依赖关系和整体进度。
- 用户已确定拆分顺序：既有行为问题先独立修复；新行为语义先讨论确认；两个前置条件均完成后才编写自动折叠行为。
- 既有行为修复保留当前“一条 stable event 对应一行”的展示语义，不提前引入 `WorkGroup`。
- 新行为实现任务同时依赖大历史扩展性修复、大型 payload 修复和折叠语义确认。
- 大型 MCP payload 已有独立任务，本任务只引用，不重复建档。
- [Material 3 TUI 改进任务](../done/2026-08-16-review-material-3-tui-improvements.md)已完成，但相关改动仍在工作区中；进入实现前必须先消除工作区重叠。
- 本任务及其子任务不构成进入规划或实现的授权。

# Task Tree

- 增加增量 History WorkGroup 投影
  - 等待既有 History 扩展性与大型 payload 问题完成
  - 等待自动折叠语义确认
  - 确定 semantic entry、group identity 与 storage span contract
  - 确定跨 batch open-edge 状态与边界归并
  - 确定 bounded segment cache 和解析失效键
  - 确定按需 group detail window 与 child virtualization
  - 确定 append、revert、多 frontend 和 transient handoff
  - 制定迁移、测试和性能验证计划
  - 与用户确认完整计划后再进入实现

# Details

- 状态：`blocked discussion`。前置任务完成前不进入详细规划或实现。
- 前置任务：[修复既有 History 的大历史扩展性](2026-08-16-stabilize-existing-history-scalability.md)、[防止大型 MCP 工具结果冻结 CLI UI](2026-08-10-prevent-large-mcp-results-from-freezing-cli.md)、[确定消息间工作记录的自动折叠语义](2026-08-16-define-history-work-folding-semantics.md)。
- `WorkGroup` 顶层 descriptor 不得保留组内所有 `StableCleanEvent`；展开内容必须通过独立的有限 detail window 按需取得。
- semantic identity 必须与 storage span 分离，使跨 batch 归并、opposite-edge eviction 和重新载入不改变同一组的身份。
- 同一 raw revision 的语义解析结果必须复用；terminal width、theme、scroll、hover、expanded state 和普通 recomposition 不得触发重新解析。
- 第一阶段不引入持久化 sidecar index；只有基准证明 storage index/value cache 与有限 segment cache 仍不足时才重新讨论。
- 实现必须建立在 group-neutral 的 window、cursor 和缓存基础设施上，不复制另一套 History 数据源。
- 本任务不构成进入规划或实现的授权。

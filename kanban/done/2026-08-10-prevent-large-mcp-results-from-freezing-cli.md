# Task Tree

- [done] 防止大型 MCP 工具结果冻结 CLI UI
  - [done] 记录现场复现、进程指标与主线程调用栈
  - [done] 确认近似二次复杂度的长文本换行是冻结根因
  - [done] 确定线性换行算法与单宽度布局缓存
  - [done] [实现并验证超长 History 文本换行修复](../done/2026-08-18-optimize-long-history-text-wrapping.md)
  - [done] 完成性能回归与真实 release CLI 验收
  - [done] 明确展示上限与同源内容去重不属于本次性能修复

# Details

- 状态：`done`。冻结根因、线性修复、缓存和真实 release 验收均已完成。
- 原始复现是在 CLI History 中展开 IDEA MCP `get_project_modules` 的 `Result`：5,984 个 module，
  `structuredContent` 与 `content[0].text` 各约 400,298 字符，单个稳定事件约 840 KiB。
- 修复前 400,298 字符换行在约 101 秒内未完成；修复后十个冷 JVM 的中位数为 6.82 ms。
- release Linux CLI 已用真实的 448,816 字符工具结果验收：首次可见约 0.49 秒，密集布局约 1.04 秒，
  后续同宽度滚动无长停顿，日志无错误。
- 当前仍同时展示 `structuredContent` preview 和完整 text content；本任务不增加展示上限，也不执行同源内容去重。
- 2026-08-27 用户接受按“防止冻结”的核心结果完成归档，不为未选择的展示策略创建后续任务。

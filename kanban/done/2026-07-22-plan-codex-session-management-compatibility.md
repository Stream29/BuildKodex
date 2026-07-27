# Task Tree

- [done] 规划Codex Session导入兼容性
  - [done] 盘点Codex thread、rollout、subagent关系和Codex Lite AgentStorage的现有映射
  - [done] 定义导入根thread及其subagent thread的只读发现、选择和加载边界
  - [done] 定义单个Codex thread到单个AgentStorage的转换语义
  - [done] 根据Codex parent/path信息重建AgentStorage树，并定义孤立、缺失、重复和循环关系的降级语义
  - [done] 定义完整AgentStorage树到新Codex Lite Session的staging与原子发布语义
  - [done] 定义全局设置入口的根thread选择、导入进度和结果反馈
  - [done] 协调现有CodexSession重构与Multi-agent V2
  - [done] 制定跨版本兼容性验证与演进计划
  - [done] 与用户完成方案审核

# Details

- 状态：planning已经用户审核完成，后续实现也已完成。
- 本任务只规划Codex thread导入；全局设置任务只负责提供入口，不定义导入语义。
- 首版的导入单位是用户选择的根thread及其全部可达subagent thread。
- 每个Codex thread独立转换为一个AgentStorage；subagent关系由CodexSession树表达，不将subagent历史合并进root AgentStorage。
- 不实时挂载Codex Session，不对Codex源执行resume、fork、archive、同步或写回。
- Codex格式读取、AgentStorage转换与thread拓扑重建收敛在`openai:codex-cli-storage`；Lite Session repository只负责staging与发布导入结果。
- 已确认的实现边界固化在[Codex CLI Storage兼容性](../../checklist/codex-cli-storage.md)。
- 导出Session到Codex不在当前范围，已记录为[未来任务](../ongoing/2026-07-22-export-session-to-codex.md)。
- 实现记录见[实现Codex Session tree导入兼容性](2026-07-22-implement-codex-session-import-compatibility.md)。

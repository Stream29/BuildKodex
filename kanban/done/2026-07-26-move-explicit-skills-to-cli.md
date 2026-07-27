# Task Tree

- [done] 将显式skill选择迁入CLI
  - [done] 把turn prefix冻结与skill选择逻辑迁入CLI应用层
  - [done] 在普通消息和新session首条消息中注入选中的skill
  - [done] 移除SkillSelectionRuntime及其模块
  - [done] 迁移测试并验证相关目标

# Details

- `AgentContextPrefixProviderImpl`继续实现`SkillsResolver`并持有缓存。
- 通用Agent Runtime不解析CLI的skill选择语法。
- JVM与Linux Native的skill定向测试通过。
- 完整CLI JVM测试中，既有的session repository故障注入用例仍独立失败。

# Task Tree

- [done] 恢复prefix provider中的skills
  - [done] 让FileSystemSkillsResolver实时观测context settings
  - [done] 由AgentContextPrefixProviderImpl持有并委托resolver
  - [done] 让CLI复用provider持有的resolver
  - [done] 补充真实文件系统测试
  - [done] 验证相关平台

# Details

- 修复`AgentContextPrefixProviderImpl`固定返回空skills的问题。
- resolver保留元数据缓存，并随`contextSettings`中的`codexHome`变化更新扫描根。

# Task Tree

- [done] 简化AgentContextPrefixProviderImpl构造器
  - [done] 移除userHome参数和测试专用构造器
  - [done] 调整测试以隔离真实用户skills
  - [done] 验证相关平台

# Details

- `userHome`由宿主环境提供，不作为provider的外部配置。

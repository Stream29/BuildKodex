# Task Tree

- [done] 删除无效的 `llm-provider`
  - [done] 删除模块源码、测试和构建定义
  - [done] 移除 Gradle 模块注册
  - [done] 清理过期的源码与设计文档引用
  - [done] 验证 Gradle 配置和受影响模块编译

# Details

- `llm-provider` 没有生产消费者，只是对 `OpenAiClient` 两个方法的无行为转发。

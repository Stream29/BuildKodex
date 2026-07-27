# Task Tree

- [done] 实现Mosaic终端界面
  - [done] 展示会话列表、当前会话状态、模型配置和plan mode
  - [done] 展示已提交历史与独立的流式输出尾部
  - [done] 提供新建、切换、fork和checkout的可发现操作
  - [done] 为checkout提供完成turn快照选择与破坏性操作确认
  - [done] 提供文本输入、多轮发送和工具执行状态展示
  - [done] 在请求、压缩或工具批次进行中禁用不合法的fork与checkout操作

# Details

界面读取AgentStorage的已提交数据，不把Mosaic状态泄漏进core模型。选择历史点时只
展示会话管理器确认过的完成turn边界，不暴露原始稀疏timeline index。

界面应适合反复操作的终端工作流，而不是营销页或静态演示页。

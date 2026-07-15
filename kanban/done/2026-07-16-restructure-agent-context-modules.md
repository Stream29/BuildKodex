# Task Tree

- [done] 重构 AgentContext 模块与动态 prefix provider
  - [done] 确认 prefix、skill 与 collaboration 的模块边界
  - [done] 迁移现有 prefix contract/render 并替换为动态 provider
  - [done] 建立 skill contract/render 并迁移 skill catalog
  - [done] 建立 collaboration contract/render 并迁移 developer instructions
  - [done] 更新 AgentState、测试与相关设计文档
  - [done] 运行跨平台相关测试

# Details

`AgentContextPrefixProvider` 在每次普通请求投影时读取当前结构化上下文；它不是快照，也不负责持久化 selected skill 或其他回合绑定内容。

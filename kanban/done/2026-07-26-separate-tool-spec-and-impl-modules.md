# Task Tree

- 区分tool spec与实现模块
  - [done] 盘点现有工具的行为所有权
  - [done] 建立`tool/spec`目录
    - [done] 迁移`plan`
    - [done] 迁移`request-user-input`
    - [done] 迁移`get-context-remaining`
    - [done] 迁移`multi-agent`
  - [done] 移动Runtime拥有的handler胶水
    - [done] 移动`get-context-remaining` handler
    - [done] 移动`multi-agent` handler
  - [done] 更新Gradle依赖与测试
  - [done] 运行相关模块测试

# Details

- `tool/impl`只存放模块内拥有实际行为和资源的工具。
- `tool/spec`只存放名称、描述、schema和输入输出DTO。
- Kotlin package保持不变，避免目录重构扩大公开API修改面。

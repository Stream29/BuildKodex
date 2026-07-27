# Task Tree

- [done] 将tool-search归入实现模块
  - [done] 移动模块到`tool/impl/tool-search`
  - [done] 更新Gradle依赖
  - [done] 更新工具职责决策
  - [done] 运行相关测试

# Details

- `tool-search`拥有索引、搜索和结果投影行为，因此属于`tool:impl`。
- 它仍是Agent Runtime特殊处理的原语，不改成普通`Tool` handler。

# Task Tree

- [done] 修复 Session Browser 的选中索引访问
  - [done] 移除已失效的包装类型 `.value` 访问
  - [done] 更新对应的 SessionManager 测试断言
  - [done] 完成 CLI Linux Native 编译验证

# Details

`SessionManagerState.selectedSessionIndex` 当前直接使用 `Int?`。

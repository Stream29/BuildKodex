# Task Tree

- [done] 简化Agent environment实现的构造
  - [done] 移除无意义的挂起工厂
  - [done] 使用公开构造函数直接建立初始选择
  - [done] 移除构造和选择过程中的路径规范化
  - [done] 更新调用方与测试
  - [done] 验证受影响模块

# Details

- 上游负责传入合法的工作目录；本实现只保存并发布环境选择。
- environment impl、prefix filesystem与runtime skill的`allTests`通过。
- IntelliJ IDEA检查与构建通过；仅保留仓库已有的Mosaic和跨平台cinterop警告。

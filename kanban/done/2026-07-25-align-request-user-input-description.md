# Task Tree

- [done] 对齐`request_user_input`描述
  - [done] 对比Rust的可用mode投影与Kotlin当前注册逻辑
  - [done] 确认当前固定可用mode不需要动态spec
  - [done] 对齐静态描述
  - [done] 增加精确描述测试
  - [done] 运行相关模块测试

# Details

- Rust根据可用`ModeKind`动态生成工具描述。
- Kotlin当前暴露静态`RequestUserInputTools.spec`。
- Kotlin当前始终在Default和Plan mode注册该工具，因此可直接使用Rust对这两个mode生成的固定描述。
- 不改变spec构造方式。
- JVM、JS Node、Linux Native和macOS ARM64测试均通过。

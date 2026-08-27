# Task Tree

- [done] 清理 `Kodex/` 的 IDEA 警告并优化代码风格
  - [done] 使用 IntelliJ IDEA 扫描整个 `Kodex/` 子模块
  - [done] 逐项复核 IDEA 报告的警告
  - [done] 在不改变既有行为的前提下尽可能消除有效警告
  - [done] 统一并优化受影响代码的风格
  - [done] 运行适用的格式化、静态检查和测试编译
  - [done] 复查剩余警告并记录无法安全消除的项目

# Details

- 当前状态：已完成。
- 工作范围仅限根子模块 `Kodex/` 内部。
- 使用 IntelliJ IDEA 的全项目检查结果作为主要清理入口。
- 本任务记录不构成自动开工授权。
- 已清理 55 个受影响文件中的有效 IDEA lint 问题，保留显式 API、公开 API 形状、测试脚本字符串和 localhost 测试等有意设计。
- 已完成相关 JVM 主代码与测试代码编译，Gradle 构建成功；`git diff --check` 通过。

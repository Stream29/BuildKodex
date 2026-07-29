# Task Tree

- 将 compact runtime 纳入 decorator 模块树
  - [done] 迁移 physical module 与 Kotlin package
  - [done] 更新 runtime composition、调用方和 Gradle 坐标
  - [done] 更新架构决策并验证编译/测试

# Details

- 用户确认：compact 也属于 ResumableAgent decorator。
- 保持现有 compaction 与续跑行为，仅调整模块和包边界。
- 补齐 compact 测试源码直接使用的 `tool-tool-search` 测试依赖。
- 验证通过：compact JVM 测试，以及 runtime、session、CLI 与集成测试的目标编译。

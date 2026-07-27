# Task Tree

- [done] 将默认Agent environment实现改为普通impl
  - [done] 将模块从`environment/filesystem`迁移到`environment/impl`
  - [done] 将实现类型改名为`AgentEnvironmentSourceImpl`
  - [done] 更新依赖、调用方与测试
  - [done] 验证受影响模块

# Details

- 本任务只修正实现分类和命名，不改变环境选择、文件系统authority、时钟或时区行为。
- Gradle验证通过：environment impl、prefix filesystem与runtime skill的`allTests`。
- IntelliJ IDEA构建通过；仅报告已有的Mosaic deprecated property与Linux跨编译macOS cinterop警告。

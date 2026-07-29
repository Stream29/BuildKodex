# Task Tree

- [done] 将 runtime decorator 收纳到统一包
  - [done] 将 steer、tool、turn-hook 迁移到 `agentruntime.decorator.*`
  - [done] 更新 runtime composition 与测试引用
  - [done] 记录包边界并运行定向验证

# Details

- 用户要求将已确认的 runtime decorator 层收纳到 `agent-runtime/decorator` 包。
- 本任务只调整 Kotlin 包路径和源码目录；现有 Gradle 模块及其项目坐标保持不变。
- `CodexAgentCompactionRuntime` 是核心运行时层，不纳入 decorator 包。
- 工作树已有大量未提交变更；只修改本任务直接涉及的文件，不覆盖或还原其他变更。
- 已通过三个 decorator 模块和 `agent-session-composition` 的 JVM 编译，以及 steer、tool 的 JVM 测试。
- `agent-runtime-turn-hook:compileTestKotlinJvm` 仍因既有测试依赖缺少 `tool-tool-search` 而失败：`TurnHookRuntimeTest.kt` 的 `ToolSearchTools` import 无法解析；主代码编译已通过。

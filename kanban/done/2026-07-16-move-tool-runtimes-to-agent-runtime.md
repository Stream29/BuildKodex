# Task Tree

- [done] 将工具 runtime 迁移至 agent-runtime
  - [done] 将 plan、apply-patch、view-image、image-generation runtime 移至对应 agent-runtime 子模块
  - [done] 调整模块依赖与包名，令 tool 模块不再依赖 agent-runtime
  - [done] 迁移运行时测试并更新集成测试引用
  - [done] 验证 JVM、JS Node、Linux X64 构建与测试

# Details

`tool` 仅保留工具 spec、输入输出 schema 与 handler；工具调用编排属于 `agent-runtime`。

已通过受影响工具、runtime 与集成测试的 JVM、JS Node、Linux X64 测试；Gradle configuration cache 已写入。

# Task Tree

- 重组 agent runtime 模块树
  - [done] 确认 session 提供依赖与路径解析、runtime impl 负责工具和层级装配
  - [done] 迁移 AgentRuntime contract、runtime impl 与 decorator 物理模块
  - [done] 更新 session 调用方、Gradle 坐标与架构决策
  - [done] 编译验证并清理旧目录

# Details

- 用户指定：`AgentDependencies` 和 `AgentPathResolver` 留在 session；runtime 只依赖对应 contract，并自行构建工具及 runtime。
- `CodexAgentDependencies`与`AgentPathResolver`现位于`agent-session/contract`；`agent-runtime/impl`只借用它们，不关闭进程级依赖。
- 不恢复、移动或清理工作树中仅暂存而已从工作目录删除的旧文件。
- 已通过 runtime/session JVM 生产编译、steer/tool JVM 测试、filesystem JVM 测试、CLI common metadata 编译和 integration JS 测试编译。
- 额外 JVM 检查仍受既有工作树/环境问题阻断：Mosaic JDK 22 native binding、缺失的`integrationToolRuntime`，以及测试源码引用已从工作目录删除的multi-agent类型。

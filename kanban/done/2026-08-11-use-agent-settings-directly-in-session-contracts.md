# Task Tree

- [done] 统一 Session surface 的 Agent settings contract
  - [done] 审计重复配置投影与当前 frontend 用法
  - [done] 删除 Session surface 配置投影
  - [done] 直接暴露 `StateFlow<KodexAgentSettings>`
  - [done] 对齐 Agent 与 New Session 命令
  - [done] 对齐物化请求、测试与约束文档
  - [done] 使用 JDK 26 验证相关模块

# Details

- 用户明确要求 frontend 直接消费 `StateFlow<KodexAgentSettings>` 和按字段更新方法。
- persisted Agent 使用持久化 settings 真源；New Session 使用进程内 `MutableStateFlow`，物化时直接交付完整 settings。
- 不再保留 `AgentConfiguration`、`AgentConfigurationState` 或 `SessionSurfaceConfigurationState` 等重复投影。
- 更新方法只修改各自负责的字段，不能让 frontend 用旧完整快照覆盖 runtime-owned 字段。
- 公共更新面固定为 model、working directory、reasoning effort、service tier 与 collaboration mode。
- New Session materialization request 直接捕获 `KodexAgentSettings`，不再携带配置 draft wrapper 或 draft revision。
- 修改范围限于尚未接入的 contract 草案、对应测试和已确认架构约束，不改造现有 frontend 实现。
- JDK 26 下 Agent、Session v2、旧 Session/New Session 与 Application contract 的 `allTests` 均通过；Session v2 的 JVM 与 Linux x64 测试实际执行通过，非宿主测试按构建配置跳过。
- `:app-contract-agent:check` 与 `:app-contract-session-v2:check` 通过。
- IntelliJ IDEA 对全部改动 Kotlin 文件执行 build 成功，仅报告既有 Native cross-compilation 与 deprecated property 警告。
- 根仓库与 `Kodex/` 的 `git diff --check` 均通过。

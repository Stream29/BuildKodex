# Task Tree

- 为 Unified Exec 进程 session 发布完成状态
  - [done] 记录启动该 session 的原始 `ExecCommandArguments`
  - [done] 在 `ProcessSession.scope` 中等待退出并更新 `completed`
  - [done] 覆盖参数保留与退出完成状态的测试
  - [done] 运行相关验证

# Details

- 用户要求：`ManagedProcessSession` 保存原始 exec 参数，并在进程退出时将公开的 `completed: StateFlow<Boolean>` 更新为 `true`。
- 验证：`jvmTest`、`compileKotlinLinuxX64`和新增测试的`linuxX64Test --tests`均通过。
- `linuxX64Test`全量运行仍有既有`exec_command defaults to the client working directory`失败；报告确认新增完成状态测试通过，且该失败不涉及本次新增的观察状态。

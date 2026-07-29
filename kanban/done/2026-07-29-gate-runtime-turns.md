# Task Tree

- [done] 将 resume turn 的并发所有权收敛到 AgentRuntime
  - [done] 在 AgentRuntime contract 中公开只读 runningTurn
  - [done] 在 runtime composition 中以 CAS 拒绝并发 resume，并在结束时清理 turn
  - [done] 让 multi-agent 使用 runtime 的 runningTurn，移除重复的 turn-job 真源
  - [done] 保留 pending steer、状态和中断语义
  - [done] 将 follow-up 的并发 CAS 竞争视为已由另一调用启动的正常 no-op
  - [done] 增加针对并发 resume 与 multi-agent 生命周期的测试并验证

# Details

- 用户确认：`ResumableAgentLayer`是既定命名，已原样保留。
- `runningTurn`对外为`StateFlow<Job?>`，可写 slot 保持在 runtime 内部；并发 resume 抛出专用异常。
- Multi-agent保留六个独立工具 factory、纯路径`AgentPathResolver`与steer + resume-if-idle；没有client、adapter、gate或scheduler。
- 通过：`:agent-session-in-memory:jvmTest`、`:tool-multi-agent:jvmTest`、`:agent-session-multi-agent:jvmTest`、`:agent-session-filesystem:jvmTest`。
- `:cli-app:compileKotlinJvm`未能到达CLI源码：前置`Mosaic:mosaic-tty:compileJdk22KotlinJvm`缺少生成的原生绑定（`Libmosaic`等），与本任务无关。

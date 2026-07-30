# Task Tree

- [done] 修复 New session 物化后的自动标题交接
  - [done] 让 New session 提交先创建并切换到最终 root Agent runtime
  - [done] 由 root Agent runtime 触发一次标题生成
  - [done] 保持标题生成只依赖 AgentState，不接入 Session catalog
  - [done] 覆盖提交、切换与标题触发时序
  - [done] 运行定向验证

# Details

- New tab 提交先物化并激活 root runtime，再将输入交给该 runtime。
- 标题任务、one-shot gate 和写回只属于 root `AgentRuntimeViewModel`；`SessionTree` 不持有标题任务或 Session title 状态。
- 全局标题设置在 runtime 接受首条输入时读取；默认模型为 `gpt-5.3-codex-spark`。
- 已通过 `:cli-session-title:linuxX64Test`、`:cli-agent:linuxX64Test`、`:cli-session:linuxX64Test`、`:cli-new-session:linuxX64Test` 与 `:cli-app:linuxX64Test`。

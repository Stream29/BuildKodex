# Task Tree

- [done] 完善 Agent runtime 状态栏操作
  - [done] active turn 优先显示 Stop
  - [done] 空闲 ToolPending 显示 Clear pending
  - [done] 其他空闲状态显示 Resume
  - [done] 覆盖三态选择优先级
  - [done] 运行相关验证

# Details

- 状态栏始终按 `runningTurn`、`KodexAgentStateValue.ToolPending`、其他状态的顺序选择唯一操作。
- `app-cli-application:linuxX64Test` 的定向测试通过。
- JVM 定向测试被 Mosaic 既有的 `PlatformKt` 重复类名编译错误阻断。

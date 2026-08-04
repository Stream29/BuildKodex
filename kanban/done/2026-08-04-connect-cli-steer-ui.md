# Task Tree

- 接入CLI steer交互
  - [done] 盘点turn运行状态、composer与pending steer
  - [done] 确定提交提示、入队和展示路径
  - [done] 实现共享提交分流
  - [done] 实现composer提示与pending预览
  - [done] 覆盖提交分流与CLI展示
  - [done] 运行共享层和CLI定向验证

# Details

- Turn job运行时，非空输入提示提交到steer。
- 前端展示尚未交付的pending steer。
- 不改变SteerRuntime的领取语义。
- `AgentRuntimeViewModel.submitComposer()`按提交瞬间的`runningTurn`分流；运行中输入原子追加为`StableCleanEvent.UserMessage`，不建立新turn。
- CLI直接订阅已有`pendingSteer`窄状态，在composer上方显示有界预览。
- Composer仅在turn运行且草稿非空白时显示`Submit to steer`。
- `:app-shared-session:jvmTest`与`:app-cli-application:jvmTest`验证通过。
- JVM验证沿用项目既有限制，排除Mosaic JDK 22 binding编译任务。

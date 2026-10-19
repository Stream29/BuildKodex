# Task Tree

- `Confirm authorization and adopt the isolated candidate`()
- `Verify the real pending-preview fixture and related suites`()
- `Commit the test repair and delivery record`()

# Details

- 用户在研究结论后授权更新测试、提交、推送并发布；版本确认为 v0.5.1。发布编排见 [发布任务](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-19-release-kodex-0-5-1-948.md#L1)。
- 采用 [调查](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-19-investigate-running-steer-preview-timeout.md#L20) 的单文件候选，仅修改 ComposerHostRepairTest 中已过时的预览用例，不改生产抢占、CAS、超时或发布协议。
- 第一条 mock 响应的取消收尾由测试 gate 控制，让真实后端队列保持未消费；原 preview 1/2 次数、提示、输入清空及 notification 断言全部保留。
- gate 从初始化起由 finally 释放；释放后等待真实 turn 完成，检查 start/earlier/next steer 各存储一次、pending 清空且仅发起两次模型请求。不带诊断 println。
- 复验四个 JVM 模块：agent/view、composer/viewmodel、agent-runtime/steer、decorator/compact；测试数与报告由实际执行填写。仅用 mock 外部模型和独立临时 Home。
- 不接手 Antigravity 文档和 scratch_out.txt 的并行修改；测试修正、研究记录及发布状态按路径提交。

## 验收

- MacBook 私有 precheck 副本执行四个 JVM task：agent/view 23、composer/viewmodel 10、agent-runtime/steer 7、decorator/compact 34，共 **74 项，0 失败、0 跳过**；整次构建 94 秒。
- 正式代码与研究候选的差异仅为说明性注释及去掉诊断 println；不改生产源。任务测试全名与原来相同，未跳过或删除覆盖。
- 产品签名提交：`7f98b6f04afe5efa4b3bd790c8e6e7d04fd5424a`，签名验证为 G，只包含该测试文件。
- 子任务完成；发布状态和后续 Home/Native 门禁由父发布任务负责，不把单测通过当作已经发布。

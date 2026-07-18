# Task Tree

- [done] 诊断 `utils-process` 整套真实 I/O 测试的挂起
  - [done] 复现并收集挂起时的测试执行信息
  - [done] 确认是 `ProcessSession` 生命周期问题，而非 TestBalloon 调度问题
- [done] 重构 `ProcessSession` 的生命周期边界
  - [done] 用组合式 common core 取代平台 session 继承
  - [done] 让已交接的 stdin 写入始终获得成功或失败结果
  - [done] 区分正常完成与中止清理，先释放取消时阻塞的资源
  - [done] 迁移 JVM、Node、POSIX、Windows 后端
- [done] 整理 `ProcessSession` 的 `SendChannel` 契约
  - [done] 移除重复声明的 channel 成员
  - [done] 将输出读取改为扩展函数
  - [done] 让 common delegate 直接委托 `StdinChannel`
  - [done] 验证 JVM、Linux、Node 的真实 I/O 测试与 MingwX64 编译
- [done] 验证并发行为
  - [done] 为生命周期竞态补充回归测试
  - [done] 启用并验证 `processSessionTest` 的并发 compartment
  - [done] 运行 Linux、JVM、Node 验证，并记录基准结果
  - [done] 在 macOS 实机验证

# Details

公共生命周期现由组合式 `ProcessSessionDelegate` 持有；`ProcessSession` 的公开契约只继承 `CoroutineScope` 与 `SendChannel<String>`，输入直接委托给 `StdinChannel`。输出读取为 `ProcessSession.read` 扩展函数，内部实现不泄露到公开契约。stdin 交接后仅会以成功或失败完成调用方。取消会先中止 stdin、终止整棵进程树，再释放平台资源。

进程树策略：POSIX 启动层确认独立 process group 后才组杀，Node 在 Unix 使用 detached group、在 Windows 使用 `taskkill /T`，JVM 使用 `ProcessHandle.descendants()`，Windows Native 在恢复进程前绑定 Job Object。此前只终止 shell 根进程，派生的 `sleep` 仍持有 stdin，导致并发取消测试等待完整五秒。

验证：JVM `processSessionTest` 连跑 5 次通过，单轮 0.16 秒；Linux Native 连跑 10 次通过，单轮 0.12 秒；Node.js 连跑 3 次通过。三目标完整测试通过，未发现残留测试子进程。补充验证：macOS ARM64 与 Windows MingwX64 均通过完整 `utils-shell-client` 实机测试。

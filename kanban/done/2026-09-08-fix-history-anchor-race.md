# Task Tree

- [done] 修复 History 窗口替换与滚动锚点竞态
  - [done] 加载已确认的诊断与现有锚点能力
  - [done] 新增窗口延迟消费下的锚点身份回归
  - [done] 修正导航测试的帧推进与进度等待
  - [done] 删除重复位置恢复及无用锚点状态
  - [done] 执行 JVM 回归与 Native 编译检查
  - [done] 更新诊断记录并清理临时文件

# Details

- 用户批准沿上一轮结论正式修复第三项；复用 LazyListState 稳定 key 恢复，
  不引入新状态或同步协议，不放宽 2 s / 15 s 门槛。
- 依据：[已确认诊断](../../shared-context/findings/2026-09-08-history-test-failures.md)。
- 保留已有 TestBalloon 迁移及前两项修复，不改其他会话内容；不提交。
- 复用本机 Java 25 / Gradle 9.5.1 Daemon，仅运行本地/mock 测试。
- 回归设计：真实 ViewModel 与 LazyColumn 配合，测试暂缓向 renderer 交付新窗口，
  在旧 provider 帧和新 provider 帧均检查锚点对象及偏移不变；覆盖双向加载。
- 先验证旧实现回归失败，再删除 captureViewportAnchor、传递参数、位置恢复分支
  及私有锚点数据类。1000 项导航只经可见边界自动加载，等待时持续推进帧，
  并检查目标方向的窗口边界进度。
- 验证 History View/ViewModel 全组、LazyColumn 回归和筛选 mock；
  对重复运行的 1000 项导航记录耗时与峰值窗口，补 Linux x64 测试编译。
- 四项新增回归覆盖 older/newer 与 0/1 行偏移；先冻结旧 provider，再交付新窗口，
  两阶段检查锚点对象与屏幕偏移。旧实现四项均在旧 provider 锚点身份断言失败。
- 已移除重复位置恢复、captureViewportAnchor、传递参数及私有数据类；
  保留 follow-latest 行为和上一轮 Ready/pending 修复。
- 修复后四项新增回归通过；JVM History View 50 项、ViewModel 32 项、
  LazyColumn 35 项、筛选 mock 1 项通过，共 118 项，无跳过。
- History View/ViewModel 全组再跑一轮均通过；两轮 1000 项往返约 3.74 / 3.84 s，
  首次加载约 61 / 67 ms，窗口峰值均为 9；未放宽原时限。
- 两个 History 模块的 Linux x64 测试编译通过；本轮使用 JVM Mosaic 测试宿主
  验证行为，未重链接/启动 Native 测试或实际 CLI。
- 已更新诊断记录。临时日志已清理，无提交、真实凭据读取或模型调用。

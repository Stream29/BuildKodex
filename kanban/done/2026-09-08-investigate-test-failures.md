# Task Tree

- [done] 诊断测试失败并修复前两项
  - [done] 加载失败用例与相关状态契约
  - [done] 隔离复现并核对索引断言
  - [done] 定位 history 等待超时
  - [done] 复测导航耗时并核对进度条件
  - [done] 修正 mock 索引断言
  - [done] 修复双向分页 Ready/pending 发布顺序
    - [done] 新增同步响应 Ready 的双向回归测试
    - [done] 验证旧实现失败、新实现通过
  - [done] 运行 mock 与 History ViewModel 回归
  - [done] 确定导航回退与返回超时根因
    - [done] 对比直接请求兜底与持续帧推进
    - [done] 追踪目标方向进度与滚动锚点覆盖
    - [done] 验证诊断对照并恢复第三项源码
  - [done] 汇总已确认根因与最小修复范围

# Details

- 用户授权修复 mock 索引断言、分页 Ready/pending 竞态，并深入研究第三项；
  导航仅做可撤回实验，未保留第三项修复或放宽时限。
- 已校正四处索引断言；双向分页在 finally 清理 pending 后发布成功 Ready。
- 新增 TestBalloon 双向即时需求回归：旧实现两项超时，修复后通过。
- 正式 JVM 回归：ViewModel 32 项、mock 1 项、History View 46 项通过；
  撤回导航实验后再跑 ViewModel 32 项与 mock 1 项通过，Linux x64 测试编译通过。
- 导航 trace 证实新窗口位置索引被旧 provider 消费，锚点从 item 5 错到 item 4，
  触发反向分页。只依靠既有稳定 key 恢复的对照通过：
  30 项严格方向实验约 711 ms，1000 项约 4.42 s；
  恢复原始测试后的 View 全组 46 项通过，1000 项约 4.02 s，峰值 9。
- 证据与源码位置保存在[诊断记录](../../shared-context/findings/2026-09-08-history-test-failures.md)。
- 第三项后续已获授权并由[锚点修复任务](2026-09-08-fix-history-anchor-race.md)完成；
  首次加载超过 2 s 的旧失败未独立归因。
- 复用本机 Java 25 / Gradle 9.5.1 Daemon；本轮运行验证限定 JVM，
  Native 只补充编译检查，未重链接/启动 Native 测试或 CLI。
- 诊断改动及临时日志已清理；保留上轮迁移和其他会话改动，无提交、凭据读取或模型调用。

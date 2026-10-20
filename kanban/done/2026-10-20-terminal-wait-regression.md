# Task Tree

- `补充虚拟时间及私有进程锁等待回归`()
- `交接测试与资源清理检查`()

# Details

- 父任务：[Agent 等待锁体验](../executable/2026-10-02-cross-platform-agent-synchronization.md)，P3。
- 仅写 `Kodex/tool/unified-exec/build.gradle.kts` 和新的等待测试文件；不得修改 P2 正在维护的既有源码或测试。
- 注入现有 ShellClient/ProcessSession，不增生产接口；十分钟边界使用虚拟时间，真实 smoke 显式短预算、私有临时目录和 finally 清理。
- 不运行 Gradle 或真实锁实验，不操作共享锁或真实 Home，不提交；主代理统一调度运行。
- 已补单一调度器的虚拟时间边界、提前结束、输出消费及取消回归，以及短预算真实管道/私有 flock smoke；未修改生产等待算法或进程接口。
- 主代理补有界清理并保留原错误；Linux 首轮取消测试未等待异步 termination 完成，按现有 ProcessSession 契约增加 owner Job join 后再释放 holder。Mac JVM 工具组 40 例、Linux JVM 新增真实 smoke 5 例（含 2 例 flock）通过，无十分钟真实 sleep；其他平台限制见父任务。

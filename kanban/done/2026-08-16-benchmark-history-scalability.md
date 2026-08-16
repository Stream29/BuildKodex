# Task Tree

- [done] 测量 History 大历史性能
  - [done] 确定代表性数据与指标
  - [done] 测量 release 启动与 session 打开耗时
  - [done] 测量增量滚动至最旧端的吞吐
  - [done] 测量 RSS、PSS、CPU 与 I/O 增长
  - [done] 测量最旧端返回最新的延迟
  - [done] 复核日志并清理临时环境

# Details

- 状态：`done`。基准测试、日志复核和临时环境清理均已完成。
- 使用当前 Linux x64 release executable。
- 使用现有 13,388 条 stable event 的真实 session 副本和隔离 HOME。
- 重复测量启动到首屏、打开 session 到 History ready，并报告中位数和范围。
- 以受控 wheel chunk 增量滚动到最旧端，在阶段点读取 `/proc` 资源指标。
- 从最旧端触发 scroll-to-latest，测量 UI 恢复和 raw-value 重读。
- 结果只描述当前实现，不在没有对照构建的情况下宣称相对旧实现的加速倍数。
- 结果记录在 [`shared-context/findings/history-scalability-benchmark-2026-08-16.md`](../../shared-context/findings/history-scalability-benchmark-2026-08-16.md)。
- 主要剩余风险是 LRU miss 后并发读取触发的无界 native file-I/O worker 增长。

# Task Tree

- [done] 修复超长 History 文本换行热路径
  - [done] 限定最小修复边界
  - [done] 确定线性换行方案
  - [done] 确定单宽度布局缓存
  - [done] 记录修复前性能基线
  - [done] 实现线性换行与缓存
  - [done] 验证换行行为与 History 渲染
  - [done] 对比性能并运行 release CLI 冒烟

# Details

- 状态：`done`。
- 本任务只处理 `wrapToTerminalWidth` 的重复后缀解析，以及 `WrappedHistoryText` 在相同文本和宽度下的重复换行。
- 不修改 History contract、LazyColumn、Mosaic runtime、工具结果展示上限或内容去重策略。
- 换行仍以终端 cell 宽度为准，不拆分字素簇。
- `wrapToTerminalWidth` 对每个 hard line 只生成一次字素段；纯可打印 ASCII 使用等宽快速路径。
- `WrappedHistoryText` 用 composable 生命周期内的普通对象缓存最近一次宽度对应的行列表，不把缓存提升到 ViewModel。
- 性能验证覆盖超长 ASCII 单行的冷换行、同条件重复布局，以及 release CLI 的实际 History session。
- JVM 基线（宽度 120）：50,000 字符约 514 ms；100,000 字符约 1.65 s；200,000 字符约 16.4 s；400,298 字符在其约 101 秒执行窗口内未完成，整个进程 120 秒超时且峰值 RSS 约 644 MiB。
- 修复后 JVM 结果（宽度 120）：50,000 字符 1.772 ms；100,000 字符 3.676 ms；200,000 字符 2.082 ms；400,298 字符 2.883 ms。400,298 字符在十个冷 JVM 中的中位数为 6.82 ms。
- JVM 与 Linux Native 的 components/history 测试均通过；另用 2,000 组混合 ASCII、CJK、组合字符和 emoji 输入、10 种宽度逐项对照旧算法，结果一致。
- release Linux CLI 使用真实的 448,816 字符工具结果完成冒烟：首次可见渲染约 0.49 秒，约 1.04 秒后完成密集的 composition/layout，RSS 约从 95 MiB 增至 133 MiB；同宽度下继续滚动无长停顿，日志无错误。
- release executable 位于 `Kodex/app/cli/build/bin/linuxX64/releaseExecutable/kodex-cli.kexe`。
- 可复用性能结论记录于 `shared-context/findings/2026-08-18-long-history-text-performance.md`。
- 本任务结果已作为大型 MCP 结果冻结问题的最终性能验收依据；展示上限和同源内容去重不属于完成范围。

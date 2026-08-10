# Task Tree

- 防止大型 MCP 工具结果冻结 CLI UI
  - [done] 记录现场复现、进程指标与主线程调用栈
  - 确定大型工具结果的展示上限与展开策略
  - 确定 `structuredContent` 与同源文本内容的去重策略
  - 确定长文本换行的算法与布局缓存边界
  - 确定性能回归测试和验收阈值

# Details

- 状态：`await discussion`。
- 等待用户明确继续讨论；当前不进入规划或实现。
- 复现条件：在 CLI 历史中展开 IDEA MCP `get_project_modules` 的 `Result` 详情。
- 现场结果包含 5,984 个 module；`structuredContent` 和 `content[0].text` 各约 400,298 字符，单个稳定事件约 840 KiB。
- `structuredContent` 的 UI 预览已截断到 512 字符，但同源的 `content[0].text` 会完整进入历史详情渲染。
- 现场表现为输入无响应；UI 主线程约占 50–79% CPU，Kotlin/Native `Main GC thread` 约占 45–60% CPU，RSS 曾从约 331 MiB 增长到 390 MiB。
- GDB 主线程栈稳定落在 `ToolDetailBody` → `WrappedHistoryText` → `wrapToTerminalWidth` → `takeFirstFittingTerminalWidth` → `GraphemeIndices`。
- `Kodex/app/cli/history/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/history/CleanEventView.kt:943` 同时展示 structured result 和 content；其 text content 分支未应用预览上限。
- `Kodex/app/cli/history/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/history/AgentHistoryView.kt:370` 在布局测量期间执行完整换行。
- `Kodex/app/cli/components/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/components/TerminalText.kt:14` 对每段剩余文本重新调用字素安全截取；超长单行输入会产生近似二次复杂度和大量临时对象。
- 现场排查仅短暂附加并分离 GDB；未修改或终止运行中的进程。

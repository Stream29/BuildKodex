# Task Tree

- 说明如何阅读执行结果，以及从具体历史条目分支或回退。
  - 演示阅读旧历史时的位置保持、回到底部与按条目展开。
  - 演示 Fork from here 与需要确认的 Revert to here。
  - 演示工具摘要、Patch 分层差异、更多行与计划状态。
  - 按确认后的目录交付三语言内容、真实演示和浏览器验证。

# Details

- 跟随最新与主动跳转由 History ViewModel 管理：[AgentHistoryViewModel.kt:95](../../Kodex/app/viewmodel/history/src/commonMain/kotlin/io/github/stream29/kodex/cli/history/AgentHistoryViewModel.kt#L95)、[205](../../Kodex/app/viewmodel/history/src/commonMain/kotlin/io/github/stream29/kodex/cli/history/AgentHistoryViewModel.kt#L205)；`[↓]` 入口见 [AgentRuntimeScreen.kt:175](../../Kodex/app/view/application/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/AgentRuntimeScreen.kt#L175)。
- 条目菜单提供 Fork from here / Revert to here；回退保留所选条目、删除后续历史，并明确不可撤销：[SessionTreeCliScreen.kt:1412](../../Kodex/app/view/application/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/SessionTreeCliScreen.kt#L1412)、[1467](../../Kodex/app/view/application/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/SessionTreeCliScreen.kt#L1467)。
- 回退和分支受执行状态约束；不要把历史回退解释为文件或 Git 回滚。与工作台任务中的 `Check out` 定位区分。
- Patch 专用视图显示状态颜色、耗时、原始工具名、Changes 层和更多行：[PatchToolEventView.kt:84](../../Kodex/app/view/patch/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/patch/PatchToolEventView.kt#L84)。
- 计划是 `[ ] / [>] / [x]` 清单，不是原始 JSON：[PlanUpdateView.kt:37](../../Kodex/app/view/history/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/history/PlanUpdateView.kt#L37)。
- 工具摘要、按需展开 payload、命令运行状态有专项测试：[CleanEventViewTest.kt:215](../../Kodex/app/view/history/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/history/CleanEventViewTest.kt#L215)。示例需使用实际执行结果，不编造成功状态。

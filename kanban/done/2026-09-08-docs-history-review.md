# Task Tree

- [done] 说明如何阅读执行结果，以及从具体历史条目分支或回退。
  - [done] 演示阅读旧历史时的位置保持、回到底部与按条目展开。
  - [done] 演示 Fork from here 与需要确认的 Revert to here。
  - [done] 演示工具摘要、Patch 分层差异、更多行与计划状态。
  - [done] 修正验证 fixture 的未物化历史目标和旧 Fork 标题断言，重跑相关测试。
  - [done] 独立准备三语言草稿、可丢弃历史数据与安全演示步骤。
  - [done] 按确认后的目录交付三语言内容、真实演示和浏览器验证。

# Details

- 本轮将精简正文合入已确认的 review-history 页面，复用真实历史/结果组件与可丢弃测试数据导出回放，严格标明样例数据，不操作个人历史。
- 逐交互验收补充：真实 InMemory 历史已能显示并操作 Fork/Revert；为直接复用原确认框，将其可见性从 private 调整为 internal，不改变渲染或行为。现有三项 Session 测试失败分别涉及未物化目标与旧标题期待，本轮校正测试准备/断言，不放宽产品的有效目标检查。
- history/patch/patch-pages/history-actions 与 workspace 的 history-index 均通过实际浏览器逐帧核对；Fork/Revert 从真实条目菜单进入，检查源/分支/回退后的历史；长补丁最终显示 Show next 6 lines。三语言正式合入，相关 JVM 测试重跑通过。所有禁用分支未逐个成片，不作全状态覆盖声明；见[当前验收](../../shared-context/findings/docs-interaction-acceptance-2026-09-08.md)。

## 首批源码盘点与交付（历史记录）

- 跟随最新与主动跳转由 History ViewModel 管理：[AgentHistoryViewModel.kt:95](../../Kodex/app/viewmodel/history/src/commonMain/kotlin/io/github/stream29/kodex/cli/history/AgentHistoryViewModel.kt#L95)、[209](../../Kodex/app/viewmodel/history/src/commonMain/kotlin/io/github/stream29/kodex/cli/history/AgentHistoryViewModel.kt#L209)；`[↓]` 入口见 [SessionTreeUiPrimitives.kt:75](../../Kodex/app/view/application/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/SessionTreeUiPrimitives.kt#L75)。
- 条目菜单提供 Fork from here / Revert to here；回退保留所选条目、删除后续历史，并明确不可撤销：[SessionTreeCliScreen.kt:1412](../../Kodex/app/view/application/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/SessionTreeCliScreen.kt#L1412)、[1467](../../Kodex/app/view/application/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/SessionTreeCliScreen.kt#L1467)。
- 回退和分支受执行状态约束；不要把历史回退解释为文件或 Git 回滚。与工作台任务中的 `Check out` 定位区分。
- Patch 专用视图显示状态颜色、耗时、原始工具名、Changes 层和更多行：[PatchToolEventView.kt:84](../../Kodex/app/view/patch/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/patch/PatchToolEventView.kt#L84)。
- 计划是 `[ ] / [>] / [x]` 清单，不是原始 JSON：[PlanUpdateView.kt:37](../../Kodex/app/view/history/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/history/PlanUpdateView.kt#L37)。
- 工具摘要、按需展开 payload、命令运行状态有专项测试：[CleanEventViewTest.kt:215](../../Kodex/app/view/history/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/history/CleanEventViewTest.kt#L215)。示例需使用实际执行结果，不编造成功状态。
- 首批写入仅限本任务、`KodexDocs/drafts/history-review/` 与 `out/docs-demos/history-review/`；草稿文件和并行约束遵循总任务，不修改网站首页、导航或主题。
- 复用[实录流程](../../shared-context/findings/kodex-terminal-recording.md)，先定位已有历史/工具结果 fixture；Fork/Revert 仅作用于独立可丢弃示例会话。缺少可运行安全数据时列出构造入口和阻塞，不读取私人会话、不伪造成功输出，不擅自实现测试导出器。

- 2026-09-08 首批核对：当前 Kodex 基线为 `0.4.3`、HEAD `89205188d5da39f36b983a0bfb57752977feb21a`；源码工作树干净，已有 CLI SHA256 为 `92539baaa88f4cb78a58ad73c5a23844ef49f65babb4ee968a692c80f26e6720`。历史位置、条目展开、Fork/Revert 状态限制与确认、Patch 分层/更多行、计划标记均已按当前源码更新草稿。
- 可丢弃构造入口已列入 `KodexDocs/drafts/history-review/scenes.md`：`AgentHistoryActionTest.kt` 的 InMemory session、`FileSystemKodexSessionRepositoryTest.kt` 的临时文件系统结构，以及 `CleanEventViewTest.kt` / `PatchToolEventViewTest.kt` 的真实组件 fixture；未发现 CLI 导入/回放命令或可直接录制的持久 fixture。
- 已用独立 bwrap Home、tmux socket 和 CLI 副本完成启动预检，仅到 `[Sessions] [New Session]` 空画面；未录制 `.cast`，未创建 `out/docs-demos/history-review/`。不得把组件测试或预检画面当作历史成功结果。
- 现有结果文件核对：历史位置/条目交互/工具结果/菜单/Patch 测试结果分别为 4、6、21、2、7 项全通过；`agentHistoryActionTest` 的旧结果为 3 项中 2 项失败，且生成类与当前源码不同，不能作为运行成功证据。
- 正式三语言合入、真实历史/Fork/Revert/结果录制、产物审核和浏览器验收仍未完成；这些节点保持未完成，待安全 fixture、可追溯构建基线和目录批准后再推进。
- 主 Session 验收：接收首批三语言素材和可丢弃数据构造方案，未接收历史操作实录；旧测试报告不能当作本次运行通过，验收未重跑这些 Gradle 测试。见[首批验收](../../shared-context/findings/docs-subagent-acceptance-2026-09-08.md)。

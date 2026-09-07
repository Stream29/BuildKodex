# History 与 mock 对话测试失败诊断

## 已保留的修复

- mock 对话初始化只占快照 0；用户消息在 1，两条 assistant 消息在 2、3。
  [测试断言](../../Kodex/integration-test/src/commonTest/kotlin/io/github/stream29/kodex/integrationtest/MinimalAgentConversationTest.kt#L427)
  已按此校正；没有丢消息或改变运行逻辑。
- 分页曾先发布 Ready，再清除 pending；同步观察者的下一页请求会被拒绝。
  [双向完成顺序](../../Kodex/app/viewmodel/history/src/commonMain/kotlin/io/github/stream29/kodex/cli/history/AgentHistoryViewModel.kt#L582)
  已改为 finally 清理后发布成功 Ready。
- [回归测试](../../Kodex/app/viewmodel/history/src/commonTest/kotlin/io/github/stream29/kodex/cli/history/HistoryPagingReadinessTest.kt#L23)
  用 Unconfined 观察者立即响应 Ready；旧实现两项均超时，修复后通过。

## 导航竞态与正式修复

- 修复前的窗口更新使用两条不同路径：窗口经 StateFlow 进入 View，
  位置请求直接写 LazyListState。
  新窗口的位置索引可能先被仍持有旧窗口的 LazyColumn provider 消费。
- 30 项 trace 捕获：窗口从 `[10..3]` 更新为 `[11..4]`，恢复 item 5 的新位置为 7；
  测量却将位置 7 解析为旧窗口中的 item 4。目标 key 与实际 anchor key 不同。
  随后 old edge 自动请求 3，窗口退回 `[10..3]`，造成往返加载。
- [LazyListState.resolveAnchor](../../Kodex/app/contract/lazy-list/src/commonMain/kotlin/io/github/stream29/kodex/cli/components/LazyListState.kt#L194)
  已支持按稳定 key 在当前 provider 中定位。现已从[窗口发布](../../Kodex/app/viewmodel/history/src/commonMain/kotlin/io/github/stream29/kodex/cli/history/AgentHistoryViewModel.kt#L753)
  删除重复位置恢复、captureViewportAnchor 及其参数/数据类，仅保留 follow-latest 定位。
- 新增[锚点回归](../../Kodex/app/view/history/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/history/AgentHistoryAnchorTest.kt#L35)：
  双向加载、0/1 行偏移，冻结旧 provider 后再交付新窗口；两阶段均检查锚点对象与偏移。
  旧实现四项均因错误移动旧 provider 而失败，修复后全部通过。
- 原测试等 StateFlow 时不推进 Mosaic 帧，并直接请求 ViewModel 分页兜底。
  [新等待辅助函数](../../Kodex/app/view/history/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/history/AgentHistoryBoundedWindowTest.kt#L145)
  持续推进帧，只由可见边界触发自动分页，等待窗口边界朝目标方向前进或到达端点。
- 上述证据针对导航回退与返回超时，不涵盖历史上单次首次加载超过 2 s 的原因。

## 验证与实验边界

- 前两项修复阶段：JVM ViewModel 32 项、筛选 mock 1 项、History View 46 项通过。
  撤回导航实验后再次验证 ViewModel 32 项与 mock 1 项，并通过 Linux x64 测试编译。
- 原 reanchor 下，持续帧推进并严格等待方向进度的实验超时。
  跳过 reanchor 后，同一 30 项实验约 711 ms，1000 项约 4.42 s。
- 仅跳过 reanchor、恢复原始测试后，History View 全组 46 项通过；
  1000 项往返约 4.02 s，窗口峰值 9（12 行 viewport）。
- 第三项正式修复：JVM History View 50 项、ViewModel 32 项、LazyColumn 35 项、
  筛选 mock 1 项通过；History View 与 ViewModel 的 Linux x64 测试编译通过。
- History View/ViewModel 全组重复运行均通过；1000 项导航两轮约 3.74 / 3.84 s，
  首次加载约 61 / 67 ms，窗口峰值均为 9。
- 这些耗时是本机 JVM 实验样本，不是性能保证；未放宽 2 s / 15 s 断言。
- 未运行 Native 测试二进制或实际 CLI；未读取真实凭据或调用模型。

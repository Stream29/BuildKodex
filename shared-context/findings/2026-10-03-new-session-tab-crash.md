# 2026-10-03 Kodex New Session tab crash

## 历史基线与后续状态

- 下文是原调查，源码链接固定到仍包含 index admission 的
  `721ac0fe6360b7da0355af04cdf38d8113274bee`，不是当前实现。
- 后续准确目标 API、重复/移动/关闭目标回归已在
  [已完成任务](../../kanban/done/2026-10-03-investigate-new-session-tab-crash.md)
  核对归档；本报告不把该旧崩溃重新登记为待实现事项。

## 结论

- 用户报告的首发错误是 `Unable to run Kodex: Tab 8 is not a New Session.`。
- 源码已确认这是 **New Session 物化命令按易失的 tab index 寻址，且允许重复/迟到提交** 导致的未捕获异常，不是存储层或 RPC 错误。
- 最符合这条错误文本的路径是同一个 New Session 提交被触发两次：
  1. UI 两次读取到 tab index `8`，各自启动一个 `materializeNewSession(8)`。
  2. `ApplicationViewModelImpl` 的 `commandMutex` 让第二个命令排队。
  3. 第一个命令成功把 index `8` 的 New Session 替换成持久化 Session。
  4. 第二个命令取得锁后仍使用 index `8`，发现该位置已经是持久化 Session；基线的 `requireNotNull` 在此抛出 `IllegalArgumentException("Tab 8 is not a New Session.")`。
- 另一条同类路径是：提交回调捕获 index 后，用户关闭、创建或打开其他 tab，列表重排使这个 index 指向了别的 tab；当前实现不会按原始 New Session 实例重新确认。
- `Main` 会把该异常向上捕获并打印为 `Unable to run Kodex: ...`，因此一次本应可恢复的重复/过期 UI 命令会退出整个 CLI 宿主。

## 证据

- [`SessionTreeCliScreen.kt:492-496`](https://github.com/Stream29/Kodex/blob/721ac0fe6360b7da0355af04cdf38d8113274bee/app/view/application/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/SessionTreeCliScreen.kt#L492-L496) 在 UI 回调中从当前渲染快照计算 `index`，随后异步调用 `materializeNewSession(index)`；没有传递 New Session 实例或稳定身份。
- [`ApplicationViewModel.kt:165-184`](https://github.com/Stream29/Kodex/blob/721ac0fe6360b7da0355af04cdf38d8113274bee/app/viewmodel/application/src/commonMain/kotlin/io/github/stream29/kodex/cli/app/ApplicationViewModel.kt#L165-L184) 虽然用 `commandMutex` 串行化命令，但在锁内再次按 index 做类型断言；第一次成功后，重复命令必然可以命中非 New Session。
- [`AgentRuntimeScreen.kt:207-236`](https://github.com/Stream29/Kodex/blob/721ac0fe6360b7da0355af04cdf38d8113274bee/app/view/application/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/AgentRuntimeScreen.kt#L207-L236) 和 [`AgentRuntimeScreen.kt:238-272`](https://github.com/Stream29/Kodex/blob/721ac0fe6360b7da0355af04cdf38d8113274bee/app/view/application/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/AgentRuntimeScreen.kt#L238-L272) 将 New Session 的提交直接委托给宿主回调，没有提交中状态或重复提交门闩。
- [`SessionTreeUiPrimitives.kt:53-65`](https://github.com/Stream29/Kodex/blob/721ac0fe6360b7da0355af04cdf38d8113274bee/app/view/application/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/SessionTreeUiPrimitives.kt#L53-L65) 对每个匹配的提交键直接执行 `onSubmit()`；在物化完成前再次收到提交事件会产生第二个命令。
- [`NewSessionViewModel.kt:56-60`](https://github.com/Stream29/Kodex/blob/721ac0fe6360b7da0355af04cdf38d8113274bee/app/viewmodel/new-session/src/commonMain/kotlin/io/github/stream29/kodex/cli/newsession/NewSessionViewModel.kt#L56-L60) 的 `materialize()` 会先执行 draft 物化，再打开持久化 Session；该挂起边界足以让重复提交排队。
- [`Main.kt:30-50`](https://github.com/Stream29/Kodex/blob/721ac0fe6360b7da0355af04cdf38d8113274bee/app/cli/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/Main.kt#L30-L50) 的 `runMosaic` 外层异常处理会打印用户看到的前缀并结束本次运行。

## 运行时证据边界

- `/home/stream/.kodex/log/Kodex.log` 在本次检查窗口没有记录这条控制台字符串；该错误通过 `println` 输出，不会自动进入应用日志。
- 因此“用户刚刚看到的第一个事件”来自用户报告；重复提交与 tab 重排是基于当前源码的确定性故障机制，尚未声称拿到了本次交互的完整事件序列。
- 当前 `Kodex/` 子模块存在用户已有的未提交组件提取工作；本次调查未修改或清理这些文件。

## 修复边界

- 后续实现应以 New Session 的稳定对象/身份作为物化 admission key，而不是把渲染时的列表 index 作为跨挂起边界的身份。
- 同一页面的第二次提交必须被忽略、合并为同一请求，或返回可恢复的非致命结果；不能让重复命令传播到 `runMosaic`。
- 必须覆盖两类回归：重复提交，以及提交后 tab 列表重排；同时保留正常物化、失败重试和现有持久化语义。
- 上述是原调查时的修复建议；后来授权的实现与验收以本报告顶部关联的已完成任务为准。

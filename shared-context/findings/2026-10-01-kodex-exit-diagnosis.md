# Kodex 退出诊断，2026-10-01

## 历史基线与后续状态

- 下文保留 2026-10-01 的故障诊断，不描述当前产品行为。源码链接固定到
  `a4c70b8cd9e4c3fc98f7c7f61bfa269669324ab6`，该已提交基线包含当时的
  CAS `check`、宿主出口和清理顺序；不是对当时未提交工作树的逐字封存。
- 后来的 [CLI 本地 unhandled error → Hook 修复](../../kanban/done/2026-10-07-restore-cli-unhandled-error-hook.md)
  已恢复宿主可继续工作的错误出口。
  [所有权闭环记录](../../kanban/done/2026-10-07-plan-spec-impl-closure-and-consistency.md)
  留存后续清理修复与验收边界；本诊断不重新打开已修复项。
- 各 runtime settings 字段的完整冲突入口验收仍见
  [独立讨论](../../kanban/discussion/2026-10-01-fix-runtime-settings-edit-crash.md)，
  不因宿主出口修复而一并宣称通过。

## 结论

- 本次没有证据证明是 Kotlin/Native `SIGSEGV`、OOM 或 MCP 服务把 Kodex 打崩。
- 用户补充的控制台原文已经把首发异常指向源码：`Unable to run Kodex: The edited field changed. Review the latest value.` 对应 [`RpcAgentViewModel.kt:63-68`](https://github.com/Stream29/Kodex/blob/a4c70b8cd9e4c3fc98f7c7f61bfa269669324ab6/app/viewmodel/rpc/src/commonMain/kotlin/io/github/stream29/kodex/cli/rpc/RpcAgentViewModel.kt#L63-L68) 的未捕获 `check`。
- 更准确的死因是：一个运行时设置编辑使用旧字段快照；并发写入先改变了同一字段后，CAS 重试返回 `false`，`RpcAgentViewModel.edit` 将这个预期的冲突升级为 `IllegalStateException`。异常从 UI 的 `rememberCoroutineScope().launch` 冒泡到 `runMosaic`，最终由 [`Main.kt:47-50`](https://github.com/Stream29/Kodex/blob/a4c70b8cd9e4c3fc98f7c7f61bfa269669324ab6/app/cli/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/Main.kt#L47-L50) 打印并结束宿主。
- 可确认的是：Kodex 进程在 2026-10-01 16:19:20（Asia/Singapore，08:19:20 UTC）发生了整宿主退出/重启。当前进程随后于 16:19:30 启动。
- 退出时四个 Session runtime 在约 32 ms 内统一关闭；仍在收尾的请求随后访问已取消的 Session owner，产生 `IllegalStateException: AgentSession storage is closed.`。这是首发异常触发退出后的清理竞态，不是首发死因。

## 证据

- `/home/stream/.kodex/log/Kodex.log:33114-33124` 显示退出前仍有正常的工具调用，之后一个请求仅标记为 `Retryable`。
- `/home/stream/.kodex/log/Kodex.log:33125-33562` 显示：
  - Session 438、419、445、446 的 runtime/session 先后集中关闭；
  - 关闭后，`finishRequestResponse` 读取 `CachedAgentStorage.index` 时抛出 `AgentSession storage is closed.`；
  - 没有同一时间段的 `segfault`、OOM、Apport 或 native 崩溃栈。
- `/home/stream/.kodex/log/Kodex.log:33563` 之后重新出现 MCP transport 和 Session opened 事件；运行中的 `kodex-cli` PID 3722591 的启动时间是 16:19:30。
- `/var/log/kern.log`、`/var/log/syslog`、`/var/log/apport.log` 在 16:19 附近没有 Kodex 的 `segfault`、`signal 6`、OOM 或 crash report；本机也没有对应 core dump。
- `editField` 在同一字段发生并发变化时按设计返回 `false`，而 `RpcAgentViewModel.edit` 用 `check(...)` 把它变成异常：[`RpcSettings.kt:27-45`](https://github.com/Stream29/Kodex/blob/a4c70b8cd9e4c3fc98f7c7f61bfa269669324ab6/app/viewmodel/rpc/src/commonMain/kotlin/io/github/stream29/kodex/cli/rpc/RpcSettings.kt#L27-L45)、[`RpcAgentViewModel.kt:63-68`](https://github.com/Stream29/Kodex/blob/a4c70b8cd9e4c3fc98f7c7f61bfa269669324ab6/app/viewmodel/rpc/src/commonMain/kotlin/io/github/stream29/kodex/cli/rpc/RpcAgentViewModel.kt#L63-L68)。
- 运行时配置菜单直接在 `rememberCoroutineScope` 中启动这些 suspend 更新，没有异常边界：[`RuntimeStatusBar.kt:389-403`](https://github.com/Stream29/Kodex/blob/a4c70b8cd9e4c3fc98f7c7f61bfa269669324ab6/app/view/application/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/RuntimeStatusBar.kt#L389-L403)。普通异常随后由 `Main.kt` 打印 `Unable to run Kodex`：[`Main.kt:31-51`](https://github.com/Stream29/Kodex/blob/a4c70b8cd9e4c3fc98f7c7f61bfa269669324ab6/app/cli/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/Main.kt#L31-L51)。

## 已确认的清理缺陷

- 宿主 finally 当时顺序是先执行 `application.close()`、`views.close()`、`global.close()`、取消 owner，最后才执行 `application.shutdown()`：[`Application.kt:150-164`](https://github.com/Stream29/Kodex/blob/a4c70b8cd9e4c3fc98f7c7f61bfa269669324ab6/app/viewmodel/application/src/commonMain/kotlin/io/github/stream29/kodex/cli/app/Application.kt#L150-L164)。
- `application.close()` 会先关闭已打开的 Session handle；`application.shutdown()` 才调用 Session registry 的 `sessions.shutdown()`：[`ApplicationViewModel.kt:308-315`](https://github.com/Stream29/Kodex/blob/a4c70b8cd9e4c3fc98f7c7f61bfa269669324ab6/app/viewmodel/application/src/commonMain/kotlin/io/github/stream29/kodex/cli/app/ApplicationViewModel.kt#L308-L315)。
- `CachedAgentStorage` 用 Session owner 的 active 状态保护存储访问；owner 被取消后访问 `index` 直接抛出该异常：[`CachedAgentStorage.kt:69-102`](https://github.com/Stream29/Kodex/blob/a4c70b8cd9e4c3fc98f7c7f61bfa269669324ab6/agent-session/filesystem/src/commonMain/kotlin/io/github/stream29/kodex/agentsession/filesystem/CachedAgentStorage.kt#L69-L102)。
- 这与当时项目约束相反：Application shutdown 应先关闭 popup、New Session 和每个 Session 的 `shutdown()`，最后释放 application resources：[`cli-session-view-models.md:166-169`](https://github.com/Stream29/BuildKodex/blob/e18b12f8af3dab70347bf07fe3af248e1c179537/checklist/cli-session-view-models.md#L166-L169)。

## 归因边界

- **确认**：这次是一次未处理的“编辑字段冲突”异常导致的整宿主生命周期终止，随后重新启动；退出阶段又存在 Session storage 提前关闭导致的收尾异常。
- **未确认**：具体是哪一个 UI 控件或哪一次并发写入先改变了字段；日志没有记录编辑目标字段和两个写入者。
- **不应认定**：把 `AgentSession storage is closed` 当作首发崩溃原因；它发生在宿主已经开始清理之后。

## 后续处理

- 首要修复 `RpcAgentViewModel.edit` 的冲突处理：不要让预期的 CAS 冲突以未捕获 `IllegalStateException` 穿透 UI；应丢弃/提示并保持 Kodex 运行，补一个并发同字段编辑的 UI/VM 回归测试。
- 其次修复宿主退出顺序，并补一条“renderer 结束且 Agent 请求仍在进行”回归测试。
- 下次若再次出现非正常退出，应在不影响当前用户进程的独立诊断启动中开启 core/backtrace 捕获；本次没有 Kotlin/Native 或 Mosaic native fault 证据。

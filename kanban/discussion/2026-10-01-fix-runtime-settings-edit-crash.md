# Task Tree

- `Confirm the stale-field conflict boundary and affected UI entrypoints`()
- `Choose non-fatal conflict handling without changing CAS semantics`()
- `Add concurrent-edit regression coverage`()
- `Run focused JVM, Native, and CLI validation`()
- `Wait for implementation authorization before moving to planning`()

# Details

## 状态与范围

- 状态：Discussion，保留各字段冲突入口的剩余验收；本次提交记录不授权实现。
- 原问题：Kodex 控制台出现 `Unable to run Kodex: The edited field changed. Review the latest value.`，随后整个 CLI 宿主退出。
- 后续宿主出口已由 [本地 unhandled error → Hook 修复](../done/2026-10-07-restore-cli-unhandled-error-hook.md)
  恢复，不能继续把该旧退出路径当成尚未修复的现行行为。
  用户确认沿用原 unhandled error/Hook 语义，不另建错误中心；
  下列全部字段入口验收仍未由这一宿主修复一并关闭。
- 已确认死因：[2026-10-01 Kodex 退出诊断](../../shared-context/findings/2026-10-01-kodex-exit-diagnosis.md)：`RpcAgentViewModel.edit` 把预期的 CAS 字段冲突通过未捕获 `check(false)` 升级为 `IllegalStateException`。
- 首要代码位置：
  - [`RpcAgentViewModel.kt:63-68`](https://github.com/Stream29/Kodex/blob/a4c70b8cd9e4c3fc98f7c7f61bfa269669324ab6/app/viewmodel/rpc/src/commonMain/kotlin/io/github/stream29/kodex/cli/rpc/RpcAgentViewModel.kt#L63-L68)
  - [`RpcSettings.kt:31-45`](https://github.com/Stream29/Kodex/blob/a4c70b8cd9e4c3fc98f7c7f61bfa269669324ab6/app/viewmodel/rpc/src/commonMain/kotlin/io/github/stream29/kodex/cli/rpc/RpcSettings.kt#L31-L45)
  - [`RuntimeStatusBar.kt:389-403`](https://github.com/Stream29/Kodex/blob/a4c70b8cd9e4c3fc98f7c7f61bfa269669324ab6/app/view/application/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/RuntimeStatusBar.kt#L389-L403)
  - [`Main.kt:31-50`](https://github.com/Stream29/Kodex/blob/a4c70b8cd9e4c3fc98f7c7f61bfa269669324ab6/app/cli/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/Main.kt#L31-L50)
  - 这些链接固定到留存故障机制的旧基线，不指向迁移后的当前源码。

## 修复边界

- 保留后端真实 CAS 和字段冲突语义；不得通过无条件覆盖、静默重放或修改冻结 RPC contract 来掩盖冲突。
- 让同字段并发编辑成为非致命结果：UI 不得因一次 stale edit 终止 `runMosaic` 或整个 Kodex 进程。
- 沿用已确认的宿主报告/Hook 出口，核实各编辑入口是否正确接线；不把冲突改成成功、不自动重试，不自行新增提示系统。
- 检查运行时状态栏、Session Settings popup 及其他调用 `AgentSettingsViewModel` suspend 编辑方法的入口，不能只修单一调用点。
- 本任务不顺带处理已记录的 `AgentSession storage is closed` shutdown 顺序竞态；如需修复，另建或关联独立任务。

## 验收条件

- 注入两个并发写入同一个 runtime settings 字段，其中一个使用过期快照；冲突不能传播为未处理异常，Kodex 继续运行。
- 覆盖 model、reasoning effort、service tier、request-user-input mode、working directory 和 thread name 等实际编辑入口中适用的冲突路径。
- 验证无冲突写入仍保持现有 CAS、合并无关字段和取消语义。
- 运行受影响模块的 JVM 测试；在资源可用时运行对应 Native 测试并编译 CLI，记录未运行的平台，不用编译成功代替 CLI smoke。
- 保持 RPC wire shape、持久化设置格式和现有用户可见设置语义不变。

## 已知相关问题

- 原退出清理缺陷依据 [`Application.kt:150-164`](https://github.com/Stream29/Kodex/blob/a4c70b8cd9e4c3fc98f7c7f61bfa269669324ab6/app/viewmodel/application/src/commonMain/kotlin/io/github/stream29/kodex/cli/app/Application.kt#L150-L164)；
  后续所有权修复与剩余限制见 [闭环记录](../done/2026-10-07-plan-spec-impl-closure-and-consistency.md)，不在此重复立项。

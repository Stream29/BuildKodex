# Task Tree

- `Preserve the source-level crash finding`()
- `Add exact-tab materialization admission`()
- `Guard duplicate New Session submission`()
- `Add stale-tab and double-submit regression coverage`()
- `Verify later authorized exact-target implementation and scoped validation`()
- `Archive the resolved index-target crash without duplicating its repair`()

# Details

## Reconciled status — 2026-10-07

- **DONE — 原 `Tab 8 is not a New Session` 的 index admission 竞态已由
  后来授权的 [Application 根硬迁移](2026-10-06-cutover-application-and-session-roots.md#approved-exact-target-api-cutover)
  实施并验收；本轮无重复 patch 或新设计**。
- 当前 [公开 API/KDoc](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/spec/application/src/commonMain/kotlin/io/github/stream29/kodex/app/application/contract/ApplicationViewModel.kt#L128)
  接收精确 `NewSessionViewModel`，不是整数 index。
  [真实实现](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/application/src/commonMain/kotlin/io/github/stream29/kodex/cli/app/ApplicationViewModel.kt#L166)
  在 command mutex 内用 `===` 查当前 slot；目标已关闭/替换/物化则返回 null，
  重复排队不再次 create/append/resume；真实物化失败仍传播，不伪造成功。
- [生产调用](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/view/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/SessionTreeCliScreen.kt#L575)
  捕获原 selected child；
  [ExactTargetCommandTest](file:///home/stream/ACodeSpace/push/BuildKodex/Kodex/app/impl/application/src/commonTest/kotlin/io/github/stream29/kodex/cli/app/ExactTargetCommandTest.kt#L20)
  覆盖 duplicate/moved/closed/equal target、失败重试与取消。
  [根验收](2026-10-06-cutover-application-and-session-roots.md#coordinator-final-handoff-and-verification)、
  [前端终验](2026-10-07-final-audit-frontend-ownership.md)及
  [发布 gate](2026-10-07-release-kodex-0-4-10.md#exact-source-gate-execution)
  提供后来执行证据，最终 main `83749434` 已发布。
- 下文是原调查基线，保留旧路径和异常传播记录，不再作为等待授权的当前阻塞。
  Runtime settings 全字段冲突、equal-twin deletion 与所有 shutdown/平台故障
  不由这张精确目标修复卡一并关闭。

## 状态与范围

- 原调查状态：Discussion；2026-10-03 当时只完成调查，尚未开始实现、提交或推送。
- 用户报告：`Unable to run Kodex: Tab 8 is not a New Session.`，随后 Kodex CLI 退出。
- 调查结论与证据：[2026-10-03 New Session tab crash finding](../../shared-context/findings/2026-10-03-new-session-tab-crash.md)。
- 首要代码位置：
  - [`SessionTreeCliScreen.kt:492-496`](../../Kodex/app/view/application/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/SessionTreeCliScreen.kt#L492-L496)
  - [`ApplicationViewModel.kt:165-184`](../../Kodex/app/viewmodel/application/src/commonMain/kotlin/io/github/stream29/kodex/cli/app/ApplicationViewModel.kt#L165-L184)
  - [`AgentRuntimeScreen.kt:207-272`](../../Kodex/app/view/application/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/AgentRuntimeScreen.kt#L207-L272)
  - [`SessionTreeUiPrimitives.kt:53-65`](../../Kodex/app/view/application/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/SessionTreeUiPrimitives.kt#L53-L65)
  - [`Main.kt:30-50`](../../Kodex/app/cli/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/Main.kt#L30-L50)

## 已确认机制

- UI 在渲染快照中把 New Session 转换成整数 index，再异步发起物化；跨挂起边界没有保留精确 New Session 实例或稳定身份。
- `ApplicationViewModelImpl` 只串行化命令，不去重同一个 New Session 的重复物化。第一个命令成功替换 tab 后，第二个命令会对同一 index 执行非 New Session 断言并抛异常。
- tab 在提交回调与命令真正取得锁之间发生关闭、创建或打开时，也可能使旧 index 指向错误 tab。
- 该异常从 `rememberCoroutineScope`/`runMosaic` 路径传播到 `Main`，被转换为用户报告的 `Unable to run Kodex`，使 UI 级竞态升级为宿主退出。

## 验收条件

- 同一 New Session 连续提交或重复收到提交键时，最多物化一次，Kodex 继续运行。
- 提交命令等待期间发生 tab 关闭、创建、打开或选择变化时，回调仍只操作原始 New Session；过期回调不得物化其他 tab。
- 正常物化、物化失败后的重试、Popup 归属、tab 顺序和持久化 Session 语义保持不变。
- 运行受影响的 Application/View/CLI 测试；在资源可用时补充 Native 验证并记录未运行的平台。

## 相关工作与限制

- [`2026-10-03-extract-new-session-and-session-tab-bar-components.md`](2026-10-03-extract-new-session-and-session-tab-bar-components.md) 的未提交组件设计已经要求 owner/request identity 与 stale callback admission；它是相关设计输入，不代表本任务已完成修复。
- `Kodex/` 子模块已有用户工作树变更；本轮只新增本调查与看板文档，不改动或清理子模块工作。
- 不顺带处理已记录的 `AgentSession storage is closed` shutdown 竞态；如需处理，关联独立任务。
- 实现、提交和推送需要后续明确授权。

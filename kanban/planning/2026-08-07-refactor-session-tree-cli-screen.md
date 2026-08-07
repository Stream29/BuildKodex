# Task Tree

- 完成 CLI 全层级 ViewModel 迁移并收敛 `SessionTreeCliScreen`
  - [done] 重新盘点当前实现
    - [done] 盘点 application、Session、Agent 与 New Session 状态所有权
    - [done] 盘点顶层订阅、直接 runtime 访问与回调链
    - [done] 对照既有 ViewModel、frontend 与 lazy history 约束
    - [done] 确认 IntelliJ 与当前未提交改动
  - [done] 确认迁移范围
    - [done] 采用完整 ViewModel 分层迁移
    - [done] 纳入 state slice、按需 Agent ViewModel 与 frontend 迁移
    - [done] 保持 runtime、storage 与产品交互语义不变
    - [done] 确定兼容适配后删除旧 API 的实施顺序
  - 建立迁移回归基线
    - 固定标签、Session browser、设置、重命名和删除行为
    - 固定 Agent 选择、history revert/fork 与 Shell menu 行为
    - 固定 New Session 物化、cwd、运行指示器与 terminal title 行为
    - 为 ViewModel 创建数和 state slice emission 增加可观测断言
  - 重构 per-Agent ViewModel
    - 将 configuration、execution、token、pending steer 与 failure 拆为窄 slice
    - 让 Agent ViewModel 持有 composer、request-user-input 与 history ViewModel
    - 暴露 history capability、Shell registry 与精确 Agent 命令
    - 从 CLI 调用面移除对 `session.runtime`、storage 和 client 的直接访问
    - 保留有限 history window、generation 与 storage cache 语义
  - 重构 per-Session ViewModel
    - 将 summary、topology、selection 与 lifecycle 拆为窄 slice
    - 用轻量 topology slot 取代携带完整子 ViewModel 的 tree entry
    - 常驻 root Agent ViewModel 并按 address 复用 materialized registry
    - 展开节点只 materialize direct children，深层选择只 materialize 所需路径
    - 用轻量 runtime observation 更新未 materialize 节点状态
    - 删除 `renderRevision` 与子状态扁平复制
  - 重构 repository、application 与 New Session 边界
    - 将 Session catalog 与 opened Session ViewModel registry 分离
    - 让 application navigation 原子持有有序 tab 与 active child ViewModel
    - 分离 catalog、navigation、global overlay 与 lifecycle flow
    - 直接暴露 settings、models 与 auth 的既有真源 flow
    - 移除 `selectedTree`、`activeNewSession()` 与复制的 New Session 名称
    - 将 New Session 配置 draft、composer 与 materialization 拆为独立 slice
    - 保持 New Session 物化和 tab 替换由 application scope 串行化
    - 将跨 Session 命令显式寻址到 Session 与 Agent address
  - 迁移 Mosaic frontend
    - 让 application、Session、Agent 与 New Session composable 各自 collect 最窄 slice
    - 让业务动作由准确 owning ViewModel 直接处理
    - 仅保留 frontend overlay 请求与通用组件语义 callback
    - 将 sidebar expansion、history viewport、focus、dropdown 与 popup anchor 留在 frontend
    - 将 Session browser、settings 与确认/上下文弹层移出顶层 screen
    - 用 Session summary 驱动标签、侧栏、spinner 与 terminal title
    - 让 overlay 携带显式 owner identity 并在 owner 失效时关闭
  - 删除迁移适配层
    - 删除 aggregate `SessionTreeCliState` 与 `RootSessionViewState`
    - 删除旧 selected-routing wrapper 和重复 capability 推导
    - 删除 frontend 对 shared runtime/storage handle 的依赖
    - 清理失效模型、imports、tests 与模块依赖
  - 完成分层验证
    - 验证多 Session selection、draft、registry 与生命周期隔离
    - 验证大量 descendants 只立即创建 root Agent ViewModel 和有限 root history
    - 验证展开、深层选择、切换与关闭时的 materialize/复用/清理
    - 验证高频 composer、stream 与 Agent 状态不发布无关上层 slice
    - 验证 history window、revert/fork、request-user-input 与 Shell session 回归
    - 验证关键 composable 可跳过且无整屏高频重组
    - 运行相关 shared、CLI Linux X64 测试、IDE 检查与终端手工验证
  - [done] 形成完整修改计划
  - 等待用户授权进入 executable

# Details

- 状态：规划完成，尚未修改 `Kodex/` 实现；等待 executable 授权。
- 用户已确认选择完整分层迁移，不采用“仅拆文件”或“只整理当前组合层”的窄方案。
- 本任务重新定基线于当前 `Kodex/app/shared/*` 与 `Kodex/app/cli/*` 实现，不直接沿用 2026-07-26 规划中的旧 `SessionManager`/`CodexCliViewModel` 路径。
- 既有长期边界记录见[多 Session 与 ViewModel 规划](../executable/2026-07-26-plan-multi-session-view-models.md)；两项任务不得并行实施，本任务负责按当前代码重新细化剩余迁移。
- IntelliJ IDEA 正在打开本项目，活动文件是 `Kodex/app/cli/application/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/SessionTreeCliScreen.kt`。
- `Kodex/` 与根仓库均有用户未提交改动；实施必须等待重叠中的 running-session surface 改动稳定，并以当时工作树为基线。

## 当前问题

- `Kodex/app/cli/application/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/SessionTreeCliScreen.kt:85-570` 同时 collect application、Session、Agent、New Session、auth 和多类 overlay 状态，并直接路由四个层级的命令。
- `Kodex/app/cli/application/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/SessionTreeCliScreen.kt:955-1516` 还内嵌完整 settings、dropdown 和通用字段实现，使顶层文件当前约 1,544 行。
- `Kodex/app/shared/application/src/commonMain/kotlin/io/github/stream29/kodex/cli/app/SessionTreeCliViewModel.kt:30-38` 用单个 aggregate state 同时复制 catalog、tabs、active target、selected tree、settings 和 models。
- `Kodex/app/shared/application/src/commonMain/kotlin/io/github/stream29/kodex/cli/app/SessionTreeCliViewModel.kt:466-479` 将当前 `RootSessionViewModel.state` 再镜像到 application state，造成父层复制子层 mutable state。
- `Kodex/app/shared/session/src/commonMain/kotlin/io/github/stream29/kodex/cli/session/RootSessionViewModel.kt:22-39` 将 topology、selection、子 Agent ViewModel、history ViewModel 与 render invalidation 混在一个 state。
- `Kodex/app/shared/session/src/commonMain/kotlin/io/github/stream29/kodex/cli/session/RootSessionViewModel.kt:64-156` 会为所有 discovered descendants 立即创建 runtime/history ViewModel；有限 history 已存在，但仍会为未访问 Agent 启动详细 UI 投影。
- `Kodex/app/shared/agent/src/commonMain/kotlin/io/github/stream29/kodex/cli/agent/AgentRuntimeViewModel.kt:36-53` 将配置、执行、token、failure 和 durable state 聚合发布；CLI 仍通过公开 `session` 读取 runtime、storage、pending steer 与 Unified Exec client。
- `Kodex/app/cli/application/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/SessionTabBar.kt:37-108` 和 `Kodex/app/cli/application/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/SessionAgentSidebar.kt:44-207` 需要跨层解析或订阅子 ViewModel，说明父层没有提供正确的 Session summary/topology contract。
- `Kodex/app/cli/application/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/AgentRuntimeScreen.kt:38-135` 已能直接调用部分 Agent 命令，但 history capability、pending steer、Shell registry 和 overlay 仍经 raw runtime 或多跳 callback 接线。

## 目标所有权

- Application ViewModel 只拥有 Session catalog、opened Session/New Session registry、tab navigation、全局设置/认证/模型入口、跨 Session 命令和 lifecycle。
- Session ViewModel 只拥有一个 root Session 的 summary、轻量 Agent topology、selection、materialized Agent registry 和 Session 生命周期。
- Agent ViewModel 只拥有一个 Agent 的 configuration、execution、composer、history、stream/pending steer、blocking interaction、Shell registry 与精确命令。
- New Session ViewModel 保留每个虚拟 tab 的 composer、配置、cwd 与 creating/failure；真实 Session 的创建和原位 tab 替换仍由 application scope 原子编排。
- Frontend 保留布局、hover、展开意图、滚动、焦点、dropdown、popup anchor、菜单、dialog route、path picker 和 login popup。
- Application global overlay slice 只表达无框架的 surface/owner intent；Mosaic 的 settings route、anchor、dropdown 和 popup/dialog presentation 仍由 frontend 持有。
- 不建立全局 callback bus、通用 action registry 或把 Mosaic 类型放进 shared ViewModel。

## 迁移方式

- 先在旧 aggregate API 旁增加新 slice 和稳定 child handle，让 shared tests 与 frontend 可逐层迁移并保持可编译。
- `RootSessionViewModel` 继续使用现有 recursive `KodexAgentSession` tree 发现执行节点，但把“发现 Session handle”和“创建详细 Agent/history ViewModel”分开；本任务不新增 storage schema 或第二套 topology repository。
- Topology slot 只保存 address、parent、depth、轻量名称/活动/执行摘要和 materialization 状态，不内嵌子 ViewModel mutable state。
- Agent address 使用 root `sessionIndex` 与 Agent identity 的组合；registry、selection、overlay 和异步 completion 不用当前 active target 反查 owner。
- Root Agent ViewModel 在 Session 打开时创建；其他 Agent 只在展开 direct children、选择路径或明确打开时创建，且同一 address 始终复用同一实例。
- `AgentHistoryViewModel` 归入对应 Agent ViewModel 生命周期；现有有限 window、tail-first 加载、generation invalidation 和 raw cache 复用保持不变。
- Session summary 分开表达 root running 与 aggregate running；标签和 terminal title 继续使用 root-only 语义，侧栏继续显示每个 Agent 自身状态。
- Agent ViewModel 直接转发 Unified Exec 原始 `activeSessions` 真源，不建立第二份 completion-filtered Shell 状态。
- Frontend 完成迁移后一次性删除 aggregate adapter、`renderRevision`、selected-only wrapper 和 raw runtime 访问，避免长期保留两套 authority。

## 预计文件范围

- 共享 Agent：`Kodex/app/shared/agent/` 的 ViewModel、state 模型、模块依赖与测试。
- 共享 Session：`Kodex/app/shared/session/` 的 repository/Session ViewModel、topology/selection 模型与测试。
- 共享 Application/New Session：`Kodex/app/shared/application/`、`Kodex/app/shared/new-session/` 的 registry、slice、命令与测试。
- CLI Application：`SessionTreeCliScreen.kt`、`SessionTabBar.kt`、`SessionAgentSidebar.kt`、`AgentRuntimeScreen.kt`、`RuntimeStatusBar.kt` 及拆出的 browser/settings/overlay 文件和测试。
- 实施时更新 `checklist/cli-session-view-models.md`、`checklist/cli-view-model-state.md` 与必要的 frontend 约束，使其以当前 recursive `KodexAgentSession` 架构描述最终 contract。

## 验证边界

- Shared 测试使用既有 in-memory Session、真实 state flow 和确定性 test client；不新增 mocking framework。
- Materialization 测试显式统计 Agent/history ViewModel 创建、复用和关闭，证明 topology discovery 不再等于详细投影创建。
- Slice 测试记录 flow emission，证明 composer edit、stream delta、Shell activity 和 topology 更新只触发相应消费者。
- Frontend 测试覆盖 owner 切换后 overlay 自动失效，并证明业务命令不再依赖多跳 callback 或当前 active target 的迟绑定。
- 主要 Gradle 验证覆盖 `app-shared-agent`、`app-shared-history`、`app-shared-session`、`app-shared-new-session`、`app-shared-application`、`app-cli-agent`、`app-cli-history` 和 `app-cli-application` 的 Linux X64 tests。
- 适用时补充 JVM tests；既有 Mosaic/JDK 22 阻断不在本任务修复范围。
- 使用 IntelliJ 检查改动文件、Compose compiler metrics/重组计数、`git diff --check` 和真实终端交互验证。

## 非目标与依赖

- 不改变 AgentRuntime、Responses、tool/hook、storage、fork/revert 数据语义或 Session 持久化格式。
- 不改变当前标签、侧栏、settings、history、cwd、spinner、terminal title 或 Shell session 的产品行为。
- 不在本任务实现 `kanban/planning/2026-08-07-fix-history-follow-latest-intent.md` 的行为修复；两项任务按最终 Agent address 与 frontend-local state contract 重新对齐。
- 不在本任务实现 text input 高级编辑，也不设计 Desktop、split pane 或新视觉形态。
- 不以行数作为唯一验收；`SessionTreeCliScreen` 的验收标准是只保留 application shell、尺寸计算、active child 选择和顶层 overlay host。

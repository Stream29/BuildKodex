# Task Tree

- [done] 支持从任意已提交 history 条目执行回退或分叉
  - [done] 完成现状调查与语义确定
    - [done] 确认每条已提交 history 记录保留真实 sparse storage index
    - [done] 确认现有 focus、次级动作、PopupMenu 与 anchor 能力可直接复用
    - [done] 确认点击条目的操作边界为 `storageIndex + 1`
    - [done] 确认覆盖全部已提交条目并排除 pending、streaming 与 UI marker
    - [done] 确认 fork 以所选 Agent 为新 root 且不复制 descendants
  - [done] 让每条已提交 history 条目可聚焦
    - [done] 将完整的多行条目包装为一个稳定的 focusable secondary-action surface
    - [done] 为每个 generation-scoped stored entry 保持稳定 PopupMenu anchor
    - [done] 显示明确的 focus 与 hover 状态
    - [done] 保留可展开 tool 子控件及 LazyColumn 跨条目键盘导航
  - [done] 接入 frontend-local history 上下文菜单
    - [done] 让菜单请求携带 Agent identity、generation、storage index 与 anchor
    - [done] 将菜单状态从 history row 逐层传递到顶层 `TuiPopupHost`
    - [done] 提供 `Revert here` 与 `Fork here` 两个菜单项
    - [done] 在 Agent、generation、target 或 anchor 失效时关闭菜单
    - [done] 在 Agent 正在运行或目标不再存在时禁止执行操作
  - [done] 实现 `Revert here`
    - [done] 用 per-Agent pending revert 状态承载破坏性操作确认
    - [done] 在执行前重新校验目标 stored index 与 exclusive boundary
    - [done] 通过 live Agent 的 `modify` 边界调用 storage revert
    - [done] 保留点击条目并删除 boundary 起的全部 timeline suffix
    - [done] 作废旧自动标题 attempt 并按保留的 history 重新同步 one-shot gate
    - [done] root Agent 回退后刷新 Session catalog 中的标题投影
  - [done] 实现 `Fork here`
    - [done] 将现有 latest-root fork 命令推广为显式 selected-Agent 与 boundary 命令
    - [done] 将所选 Agent 的 boundary prefix 复制为新的 root Session
    - [done] 不复制 source descendants 且不修改 source Agent
    - [done] 为 target 追加 `[fork] <boundary title>` settings change point
    - [done] 打开并选择新 Session tab
  - [done] 完成验证
    - [done] 覆盖所有 stable event 类型的 exact-boundary revert 与 fork
    - [done] 覆盖 stale target、运行中 Agent、失败恢复与 history generation 失效
    - [done] 覆盖 root 与 subagent fork、target 标题及不复制 descendants
    - [done] 覆盖整条多行记录的 focus、右键、`Shift+F10`、菜单定位与焦点恢复
    - [done] 覆盖含可展开子控件的 tool 条目不会丢失 context-menu 操作
    - [done] 运行相关 shared、history 与 CLI application Gradle 验证

# Details

## 状态

- 已完成history entry focus、上下文菜单、精确边界revert/fork及状态同步。
- 菜单只在所选Agent没有active turn job且处于稳定状态时弹出；后端命令会再次校验。
- 已生成可直接试用的Linux X64 release CLI。

## 冻结范围

- “任意 history 条目”指 `AgentHistoryWindow.entries` 中拥有稳定 storage index 的全部 committed `StableCleanEvent`。
- Pending tool、当前 streaming response、loading/failure/empty marker 没有稳定 snapshot boundary，不提供菜单。
- `Revert here` 与 `Fork here` 都使用 `entry.index + 1` 作为 exclusive boundary，因此点击条目保留，之后的所有 storage timeline change point 被删除或不复制。
- 支持当前所选 root Agent 或 subagent。Subagent fork 后成为新 Session 的 root；不复制其 descendants。
- Revert 只回退所选 Agent 的 storage timelines，不回滚 Session topology 或外部环境副作用。
- Revert 遵守既有 per-Agent checkout confirmation 决策；Fork 直接创建并切换到新 Session。

## 实施前能力与缺口

- `Kodex/app/viewmodel/history/src/commonMain/kotlin/io/github/stream29/kodex/cli/history/AgentHistoryModels.kt:25` 已为每条 committed event 保存真实 sparse index。
- `Kodex/app/cli/view/history/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/history/AgentHistoryView.kt:134` 已用 generation 与 storage index 建立 LazyColumn key，但第 207 行仅直接渲染 event，尚无整条 focus/anchor surface。
- `Kodex/app/cli/components/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/components/TuiPressable.kt:32` 已统一 focus、右键与 `Shift+F10` secondary action，可直接承载完整 history entry。
- `Kodex/app/cli/components/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/components/TuiPopup.kt:66` 已提供稳定 anchor；`Kodex/app/cli/view/application/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/SessionTreeCliScreen.kt:173` 已提供 host-level popup sibling 模式。
- `Kodex/app/viewmodel/history/src/commonMain/kotlin/io/github/stream29/kodex/cli/history/AgentHistoryViewModel.kt:141` 已在 latest index 回退后提升 generation 并重载有限窗口。
- `Kodex/agent-storage/contract/src/commonMain/kotlin/io/github/stream29/kodex/agentstorage/contract/KodexAgentStorage.kt:152` 与第 176 行已有全 timeline revert/fork 原语；允许任意 committed event boundary 后需同步更新过时的 turn-only 调用说明。
- `Kodex/app/viewmodel/session/src/commonMain/kotlin/io/github/stream29/kodex/cli/session/SessionRepositoryViewModel.kt:120` 当前只从 root storage fork；`Kodex/app/viewmodel/application/src/commonMain/kotlin/io/github/stream29/kodex/cli/app/SessionTreeCliViewModel.kt:165` 当前只支持 active root 的 latest boundary。
- `Kodex/app/viewmodel/agent/src/commonMain/kotlin/io/github/stream29/kodex/cli/agent/AgentRuntimeViewModel.kt:87` 尚无 UI-originated revert 命令。
- `Kodex/app/shared/session-title/src/commonMain/kotlin/io/github/stream29/kodex/cli/sessiontitle/AgentTitleGeneration.kt:89` 只能永久 suppress attempt，实施 revert 时需要按保留 history 恢复正确的 one-shot 状态。

## 验证范围

- 定向测试覆盖 `app-shared-session-title`、`app-shared-agent`、`app-shared-history`、`app-shared-session`、`app-shared-application`、`app-cli-history` 与 `app-cli-application`。
- 至少运行相关 JVM tests 与 Linux X64 compilation；再按实际改动运行格式化、API 或其他适用检查。
- 验证不得创建 mock；使用现有 in-memory Session/Storage 与 Mosaic test runtime。

## 验证结果

- `app-shared-session-title`、`app-shared-agent`、`app-shared-history`、`app-shared-session`和`app-shared-application` JVM tests通过。
- `app-cli-history`和`app-cli-application` Linux X64 tests通过。
- `app-cli-application:compileKotlinLinuxX64`和`linkReleaseExecutableLinuxX64`通过。
- `git diff --check`通过；IntelliJ静态检查仅有Mosaic animation依赖的项目同步误报。

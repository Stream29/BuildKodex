# Task Tree

- 为 TUI Codex patch renderer 制定实现计划
  - [done] 调查当前 patch 数据与 history 渲染链路
  - [done] 对照 Codex TUI 的 patch/diff 展示基线
  - [done] 确认前端独立解析 raw patch 与低频重解析约束
  - 明确 frontend parser 输出契约、模块边界与错误回退
  - 明确 parsed snapshot 的稳定 identity、缓存与失效边界
  - 明确 summary、diff body、状态、主题、换行和性能语义
  - 拆分实现、集成与验证步骤
  - 与用户确认计划后再进入实现

# Details

- 状态：`await planning`。本轮只完成调研与建档，不实施 renderer。
- 调研基线：2026-07-26 的 Kodex working tree；Codex 参考源码为 shared context commit `61a44880a85d2fd0d8770908dea5733495e571c8`。
- 用户已确定 raw `apply_patch` input 是前端解析的事实源。前端必须独立执行一次 parse，不能依赖或复用工具执行端已经产生的 parsed object；可以复用无状态 parser 实现。
- frontend projection 必须持有稳定的 parsed snapshot。相同 history item 与 raw patch revision 只能复用既有成功或失败结果，不得因对象重建而重复 parse。
- terminal width、theme、scroll、hover、expanded state、tool pending/completed 状态和普通 recomposition 只允许重新格式化或绘制，不得进入 parse cache key。
- raw patch revision 变化才允许失效并重新 parse。若以后展示 `ToolCallInputDelta`，不得对每个 delta 或 render frame 执行 full parse，必须使用有界的增量或合并更新边界。
- 当前 history 投影按 call id 配对通用 tool call/output，`apply_patch` 仍投影为普通 `ConversationHistoryItem.ToolCall`，只保留 raw input 和文本结果：`Kodex/cli/app/src/commonMain/kotlin/io/github/stream29/kodex/cli/app/model/ConversationHistory.kt:92`、`:142`。
- 当前 history renderer 的展示结果是 `List<String>`；工具调用统一显示可展开的 `Input`/`Output`，没有 patch 语义或 span 样式：`Kodex/cli/app/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/HistoryView.kt:145`、`:193`。
- 当前 stream 投影没有消费 `ToolCallInputDelta`；patch 输入不会作为专用 streaming history item 展示：`Kodex/cli/app/src/commonMain/kotlin/io/github/stream29/kodex/cli/app/model/SessionStreaming.kt:22`。
- `utils:patch` 已有解析后的 `Patch`、add/delete/update hunk，以及包含 old/new content、move 和 affected paths 的 `PatchApplyResult`：`Kodex/utils/patch/src/commonMain/kotlin/io/github/stream29/kodex/utils/applypatch/PatchModels.kt:16`、`:79`、`:98`。
- `StablePatchToolEvent` 已明确将 parsed diff 与 success/failure execution result 作为专用 clean event；仓库内尚无该模型的生产者或 UI 消费者：`Kodex/agent-storage/clean-models/src/commonMain/kotlin/io/github/stream29/kodex/agentstorage/cleanmodels/StablePatchToolEvent.kt:20`。
- 当前 `ApplyPatchToolClient.apply` 返回 `Unit` 并丢弃 `PatchApplyResult`，tool output 只写入成功或失败文本。因此现有 history 链路无法提供 Codex renderer 的精确 applied delta；前端 renderer 仍须从 raw input 建立自己的 parsed snapshot：`Kodex/tool/apply-patch/src/commonMain/kotlin/io/github/stream29/kodex/tool/applypatch/ApplyPatchToolClient.kt:13`、`Kodex/tool/apply-patch/src/commonMain/kotlin/io/github/stream29/kodex/tool/applypatch/ApplyPatchTools.kt:42`。
- Mosaic 已支持 `AnnotatedString`/`SpanStyle`、前景色、背景色和 terminal theme；基础富文本不需要先修改 fork。`Text` 只裁剪超宽行，因此 styled diff 仍需在 renderer 中按 terminal cell width 预先换行：`Kodex/Mosaic/mosaic-runtime/src/commonMain/kotlin/com/jakewharton/mosaic/ui/Text.kt:62`、`Kodex/Mosaic/mosaic-runtime/src/commonMain/kotlin/com/jakewharton/mosaic/TerminalState.kt:9`。
- Codex 参考实现使用独立 `FileChange` 模型，区分 add/delete/update 与 move：`shared-context/codex/codex-rs/tui/src/diff_model.rs:7`。
- Codex history patch cell 按 path 排序，显示单文件或多文件 summary、增删行计数及完整 diff body；成功时保留该 patch block，失败时追加失败 block：`shared-context/codex/codex-rs/tui/src/history_cell/patches.rs:7`、`shared-context/codex/codex-rs/tui/src/chatwidget/tool_lifecycle.rs:11`、`:154`。
- Codex diff body 覆盖 add/delete/update、rename、line number、hunk gap、长行硬换行、相对路径、light/dark 与不同色深；syntax highlight 对大 diff 有上限保护：`shared-context/codex/codex-rs/tui/src/diff_render.rs:351`、`:410`、`:482`。
- 项目既有决策要求 `apply_patch` 保持专用 stable clean event，不退化为 generic stable tool event：`checklist/clean-model-rust-alignment.md:5`、`:6`。
- renderer 必须保持为 history row 的纯展示逻辑；storage I/O、协议配对和 patch 解析不得进入 composition，通用 `LazyColumn` 不感知 patch 模型：`checklist/tui-interaction-components.md:33`、`:85`，`checklist/cli-view-model-state.md:74`。
- 正式计划需与 [CLI 全层级 contract/ViewModel 迁移](../done/2026-08-07-refactor-session-tree-cli-screen.md)兼容，避免把新组件绑定到完整 history snapshot。
- 正式计划的验证范围至少覆盖结构化投影、成功/失败 parse 缓存、add/delete/update/move、多文件排序与计数、窄终端和 Unicode 换行、主题样式、history stable identity/展开行为及大 patch 的有界渲染。
- 需用可观测 parse counter 验证：同一 raw patch revision 在 resize、theme、scroll、hover、展开、tool result 配对及普通 recomposition 中均不增加 parse 次数。

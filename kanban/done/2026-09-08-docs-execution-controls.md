# Task Tree

- [done] 说明如何在任务执行期间追加要求及控制运行状态。
  - [done] 演示运行中键入并提交内容进入 Pending steer。
  - [done] 核对 Stop、Clear pending、Resume、Compact 的出现条件与作用。
  - [done] 说明状态栏配置、token 数和执行耗时的实际含义。
  - [done] 独立准备三语言草稿、运行状态构造与安全演示步骤。
  - [done] 按确认后的目录交付三语言内容、真实演示和浏览器验证。
  - [done] 补录空闲首次提交，与运行中 steer 的实际输入路径对照。
  - [done] 验收新原生执行片段的浏览器回放与真实结果来源。

# Details

- 本轮用户明确要求真实使用场景，授权只读本地凭据。`execution.cast` 已替换为独立文件系统 Session 中的真实模型运行：加载英文历史后提交新回合、运行中 steer 被消费、Stop 后 Resume、真实待回答问题 Clear pending、真实 Compact 完成。实际修改两个示例文件并通过四项检查；仅操作演示 Home/project，原始会话只读。
- 默认播放器 2 倍速、压缩空闲间隔；原始 ANSI 和时间线未改写。旧 fixture 片段不再作为正式 execution 资产。新片段 13 个检查点已在浏览器匹配，来源、真实工具结果和隐私边界见[补录验收](../../shared-context/findings/docs-showcase-refinement-2026-09-08.md)；下列首批边界不代表本轮结果。
- 本轮将精简正文合入已确认的 run-session 页面，并在测试入口导出真实运行界面组件的示例状态与交互，不冒充在线模型响应。
- execution 的 9 个检查点和三语言页面已通过浏览器验收。真实 Composer 接收键入/提交；Stop/Clear pending/Resume/Compact 点击驱动离线 fixture。没有模型请求、真实工具取消或压缩完成结果；空闲首次提交仍未成片，留在任务树。见[当前验收](../../shared-context/findings/docs-interaction-acceptance-2026-09-08.md)。

## 首批源码盘点与交付（历史记录）

- 运行时提交 Composer 文本会加入 `pendingSteer`，空闲时才接受新一轮输入：[AgentRuntimeViewModel.kt:199](../../Kodex/app/viewmodel/agent/src/commonMain/kotlin/io/github/stream29/kodex/cli/agent/AgentRuntimeViewModel.kt#L199)。
- 界面显示 Pending steer 预览和提交提示：[AgentRuntimeScreen.kt:172](../../Kodex/app/view/application/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/AgentRuntimeScreen.kt#L172)、[304](../../Kodex/app/view/application/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/AgentRuntimeScreen.kt#L304)。不能宣称追加要求会立即抢占当前工具。
- 状态栏根据状态选择 Stop / Clear pending / Resume，并按能力显示或禁用 Compact：[RuntimeStatusBar.kt:70](../../Kodex/app/view/application/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/RuntimeStatusBar.kt#L70)。
- 对应执行入口：[AgentRuntimeViewModel.kt:232](../../Kodex/app/viewmodel/agent/src/commonMain/kotlin/io/github/stream29/kodex/cli/agent/AgentRuntimeViewModel.kt#L232)。说明停止、恢复与清理待处理调用的区别，不将它们统称为取消。
- 模型、推理强度、服务层级、提问模式和工作目录可从状态栏进入；耗时位于 Composer 分隔线：[AgentRuntimeScreen.kt:175](../../Kodex/app/view/application/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/AgentRuntimeScreen.kt#L175)。
- token 数/压缩效果应按录制版本核实，不写成账号余额或保证某种压缩比例。
- 首批写入仅限本任务、`KodexDocs/drafts/execution-controls/` 与 `out/docs-demos/execution-controls/`；草稿文件和并行约束遵循总任务，不修改网站首页、导航或主题。
- 复用[实录流程](../../shared-context/findings/kodex-terminal-recording.md)，先定位现有 runtime/ViewModel 测试 fixture。空 Home 禁网探测不能验证真实模型运行；无法安全复现时交付具体 fixture 入口、步骤和阻塞，不访问个人账号、不编造运行状态，不擅自实现测试导出器。
- 2026-09-08 首批已完成源码/fixture 研究，并写入 `KodexDocs/drafts/execution-controls/zh-CN.md`、`zh-TW.md`、`en-US.md`、`scenes.md`。相关 steer、compact、state 和 UI control JVM 测试已用 Temurin 25 顺序运行成功。
- 当前未生成 `out/docs-demos/execution-controls/` 录制；现有空 Home 禁网 CLI 没有在线模型运行入口，缺少安全的 runtime/ViewModel 到真实屏幕/PTY 状态桥接。保留真实演示、最终合入和浏览器验收节点未完成。
- 主 Session 验收：接收首批三语言素材、fixture 入口与阻塞说明，未接收运行控制实录。没有安全状态桥接的阻塞需统一协调，不能以源码/状态测试替代成片。见[首批验收](../../shared-context/findings/docs-subagent-acceptance-2026-09-08.md)。

# Task Tree

- 说明如何在任务执行期间追加要求及控制运行状态。
  - 演示运行中提交内容进入 Pending steer，与空闲时新一轮提交区分。
  - 核对 Stop、Clear pending、Resume、Compact 的出现条件与作用。
  - 说明状态栏配置、token 数和执行耗时的实际含义。
  - 按确认后的目录交付三语言内容、真实演示和浏览器验证。

# Details

- 运行时提交 Composer 文本会加入 `pendingSteer`，空闲时才接受新一轮输入：[AgentRuntimeViewModel.kt:199](../../Kodex/app/viewmodel/agent/src/commonMain/kotlin/io/github/stream29/kodex/cli/agent/AgentRuntimeViewModel.kt#L199)。
- 界面显示 Pending steer 预览和提交提示：[AgentRuntimeScreen.kt:172](../../Kodex/app/view/application/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/AgentRuntimeScreen.kt#L172)、[304](../../Kodex/app/view/application/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/AgentRuntimeScreen.kt#L304)。不能宣称追加要求会立即抢占当前工具。
- 状态栏根据状态选择 Stop / Clear pending / Resume，并按能力显示或禁用 Compact：[RuntimeStatusBar.kt:70](../../Kodex/app/view/application/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/RuntimeStatusBar.kt#L70)。
- 对应执行入口：[AgentRuntimeViewModel.kt:232](../../Kodex/app/viewmodel/agent/src/commonMain/kotlin/io/github/stream29/kodex/cli/agent/AgentRuntimeViewModel.kt#L232)。说明停止、恢复与清理待处理调用的区别，不将它们统称为取消。
- 模型、推理强度、服务层级、提问模式和工作目录可从状态栏进入；耗时位于 Composer 分隔线：[AgentRuntimeScreen.kt:175](../../Kodex/app/view/application/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/AgentRuntimeScreen.kt#L175)。
- token 数/压缩效果应按录制版本核实，不写成账号余额或保证某种压缩比例。

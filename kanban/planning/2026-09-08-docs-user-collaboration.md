# Task Tree

- 说明用户怎样回答 Agent 问题和审核建议的新会话。
  - 演示选项回答、Other 自由输入与自动焦点推进。
  - 核对 ask user / no question 模式及已有答案草稿的行为。
  - 演示建议会话内容、接受前配置和带反馈的拒绝。
  - 按确认后的目录交付三语言内容、真实演示和浏览器验证。

# Details

- 选项确认后焦点转到下一题；Other 转入自由文本；最后显式提交：[AgentRuntimeView.kt:91](../../Kodex/app/view/agent/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/agent/AgentRuntimeView.kt#L91)。
- 当前界面模式标签为 `ask user` / `no question`，不可按其他工具接口臆造额外模式：[SessionTreeUiPrimitives.kt:138](../../Kodex/app/view/application/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/SessionTreeUiPrimitives.kt#L138)。
- Suggested Sessions 展示名称和完整任务提示，提供 Accept / Reject，拒绝支持可选反馈：[AgentRuntimeView.kt:184](../../Kodex/app/view/agent/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/agent/AgentRuntimeView.kt#L184)。
- 接受前配置针对该批建议，包含模型、推理强度、服务层级、提问模式、工作目录：[AgentRuntimeScreen.kt:146](../../Kodex/app/view/application/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/AgentRuntimeScreen.kt#L146)；不误称为每项单独配置。
- 录制前沿 ViewModel 核对会话创建与反馈的实际结果；不能把“接受建议”解释成任务已经完成或自动合并结果。

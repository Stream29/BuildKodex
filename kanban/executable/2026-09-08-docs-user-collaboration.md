# Task Tree

- 说明用户怎样回答 Agent 问题和审核建议的新会话。
  - [done] 演示选项回答、Other 自由输入与自动焦点推进。
  - [done] 核对 ask user / no question 模式及已有答案草稿的行为。
  - [done] 演示建议会话内容、接受前配置和带反馈的拒绝。
  - [done] 独立准备三语言草稿、表单/审批数据与安全演示步骤。
  - [done] 按确认后的目录交付三语言内容、真实演示和浏览器验证。
  - 补录批次工作目录选择及 Accept 后创建/打开可丢弃 Session 的实际结果。

# Details

- 本轮将精简正文合入已确认的 run-session 页面，复用真实问答/审批组件和安全测试数据导出交互回放；不创建联网 Agent。
- questions 7 帧与 suggestions 9 帧已逐行回放核对，三语言正式页通过浏览器检查；表单含回改草稿/显式提交，建议含模型分层和提问模式配置、反馈拒绝。提交由 fixture 断言验证；Accept 结果尚未展示，不能把按钮可见算完成。见[当前验收](../../shared-context/findings/docs-interaction-acceptance-2026-09-08.md)。

## 首批源码盘点与交付（历史记录）

- 选项确认后焦点转到下一题；Other 转入自由文本；最后显式提交：[AgentRuntimeView.kt:91](../../Kodex/app/view/agent/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/agent/AgentRuntimeView.kt#L91)。
- 当前界面模式标签为 `ask user` / `no question`，不可按其他工具接口臆造额外模式：[SessionTreeUiPrimitives.kt:138](../../Kodex/app/view/application/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/SessionTreeUiPrimitives.kt#L138)。
- Suggested Sessions 展示名称和完整任务提示，提供 Accept / Reject，拒绝支持可选反馈：[AgentRuntimeView.kt:184](../../Kodex/app/view/agent/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/agent/AgentRuntimeView.kt#L184)。
- 接受前配置针对该批建议，包含模型、推理强度、服务层级、提问模式、工作目录：[AgentRuntimeScreen.kt:146](../../Kodex/app/view/application/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/AgentRuntimeScreen.kt#L146)；不误称为每项单独配置。
- 录制前沿 ViewModel 核对会话创建与反馈的实际结果；不能把“接受建议”解释成任务已经完成或自动合并结果。
- 首批写入仅限本任务、`KodexDocs/drafts/user-collaboration/` 与 `out/docs-demos/user-collaboration/`；草稿文件和并行约束遵循总任务，不修改网站首页、导航或主题。
- 复用[实录流程](../../shared-context/findings/kodex-terminal-recording.md)，定位现有问答/建议会话 fixture；区分真实组件显示、提交结果与在线模型行为。若没有安全可运行入口，记录具体阻塞，不拿手绘表单或伪造 ANSI 替代，不擅自实现测试导出器。
- 2026-09-08 首批交付：`KodexDocs/drafts/user-collaboration/zh-CN.md`、`zh-TW.md`、`en-US.md` 与 `scenes.md`。已核对 AgentRuntimeView、AgentRuntimeScreen、RequestUserInput/SuggestSubagent ViewModel、模式工具规格、历史投影及相关 Mosaic/ViewModel 测试；四个相关 JVM 测试任务通过。
- 当前未生成 `out/docs-demos/user-collaboration/`：现有 CLI 没有 pending `request_user_input` / `suggest_subagent_task` 的安全 fixture 注入口，唯一接近的真实运行 probe 依赖 `realOpenAiClient()`；不使用私人账号/会话，也不实现测试导出器。正式录制与网站合入、浏览器验收仍未完成，根节点保持未完成。
- 主 Session 验收：接收首批三语言素材与表单/审批 fixture 研究，未接收真实 pending 交互录制；与执行控制、历史任务共享安全场景入口阻塞。见[首批验收](../../shared-context/findings/docs-subagent-acceptance-2026-09-08.md)。

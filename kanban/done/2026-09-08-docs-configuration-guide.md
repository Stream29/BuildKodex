# Task Tree

- [done] 说明配置作用域、即时入口及自动会话标题。
  - [done] 核对全局、当前会话、新会话默认值与创建草稿的作用域。
  - [done] 演示状态栏与设置页修改同类选项的入口。
  - [done] 演示自动标题开关及独立模型、推理强度配置。
  - [done] 独立准备三语言草稿、作用域对照与安全录制证据。
  - [done] 校正源码基线及复用录制的实际覆盖范围。
  - [done] 按确认后的目录交付三语言内容、真实演示和浏览器验证。

# Details

- 本轮合入 configuration 页面，校正来源记录，使用新构建的隔离 CLI 补 New session/Title generation 控件实录；标题请求仍用测试验证，不使用私人在线账号。
- 当前 configuration 为重建 CLI 的 26 个检查点，含独立标题模型/推理、禁用/重启用、默认值与草稿隔离、状态栏分层菜单和提问模式；全部回放一致。三语言正式内容通过浏览器检查；仅完成控件/作用域说明，不声称实录生成标题。见[当前验收](../../shared-context/findings/docs-interaction-acceptance-2026-09-08.md)。

## 首批源码盘点与交付（历史记录）

- 当前会话与新会话设置使用不同 ViewModel/入口，不能统称为一个全局设置：[SettingsPopup.kt:683](../../Kodex/app/view/settings/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/settings/SettingsPopup.kt#L683)、[808](../../Kodex/app/view/settings/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/settings/SettingsPopup.kt#L808)。
- 自动会话标题独立配置开关、Title model 和 Title reasoning：[SettingsPopup.kt:536](../../Kodex/app/view/settings/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/settings/SettingsPopup.kt#L536)。
- 新会话草稿的配置隔离及首次创建时快照有测试：[NewSessionViewModelTest.kt:38](../../Kodex/app/viewmodel/new-session/src/commonTest/kotlin/io/github/stream29/kodex/cli/newsession/NewSessionViewModelTest.kt#L38)。编写前沿实现确认默认值变更何时生效，不承诺会修改已有会话。
- 状态栏模型、推理强度、服务层级为分层菜单，另有提问模式与目录入口：[RuntimeStatusBar.kt:201](../../Kodex/app/view/application/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/RuntimeStatusBar.kt#L201)。
- 认证、账号用量、上下文来源、MCP、Hooks 的归属由结构任务统一决定；本任务不重复编写其他章节或擅自调整用户配置。
- 首批写入仅限本任务、`KodexDocs/drafts/configuration-guide/` 与 `out/docs-demos/configuration-guide/`；草稿文件和并行约束遵循总任务，不修改网站首页、导航或主题。
- 复用[实录流程](../../shared-context/findings/kodex-terminal-recording.md)，只改隔离 Home 的演示配置；配置控件展示与真正触发自动标题请求分别标注，不能把禁网设置演示当作自动标题生成验证。
- 首批已交付 `KodexDocs/drafts/configuration-guide/{zh-CN,zh-TW,en-US,scenes}.md`，并将已验证的真实 Settings 片段副本放入 `out/docs-demos/configuration-guide/`；仅完成草稿、作用域对照、源码/测试基线和安全证据整理。
- 已运行 `KodexDocs` 的 `npm test`（最终 14/14 通过），未重建 CLI、未运行 Gradle/联网标题生成，也未接入正式路由或进行浏览器验收；目录确认、真实标题生成证据、正式合入与浏览器验收仍阻塞，因此本任务继续留在 `planning/`，根节点未完成。
- 主 Session 验收：草稿及复用画面保留，但 `scenes.md` 的当前 HEAD/工作树描述与本轮基线不符，测试数仍写 10/10；计划步骤与已录步骤需分开。复用片段没有录到 New session 默认值/Title generation 操作，不能作为这些控件的演示完成证明。见[首批验收](../../shared-context/findings/docs-subagent-acceptance-2026-09-08.md)。

# Task Tree

- 说明配置作用域、即时入口及自动会话标题。
  - 核对全局、当前会话、新会话默认值与创建草稿的作用域。
  - 演示状态栏与设置页修改同类选项的入口。
  - 演示自动标题开关及独立模型、推理强度配置。
  - 按确认后的目录交付三语言内容、真实演示和浏览器验证。

# Details

- 当前会话与新会话设置使用不同 ViewModel/入口，不能统称为一个全局设置：[SettingsPopup.kt:683](../../Kodex/app/view/settings/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/settings/SettingsPopup.kt#L683)、[808](../../Kodex/app/view/settings/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/settings/SettingsPopup.kt#L808)。
- 自动会话标题独立配置开关、Title model 和 Title reasoning：[SettingsPopup.kt:536](../../Kodex/app/view/settings/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/settings/SettingsPopup.kt#L536)。
- 新会话草稿的配置隔离及首次创建时快照有测试：[NewSessionViewModelTest.kt:38](../../Kodex/app/viewmodel/new-session/src/commonTest/kotlin/io/github/stream29/kodex/cli/newsession/NewSessionViewModelTest.kt#L38)。编写前沿实现确认默认值变更何时生效，不承诺会修改已有会话。
- 状态栏模型、推理强度、服务层级为分层菜单，另有提问模式与目录入口：[RuntimeStatusBar.kt:201](../../Kodex/app/view/application/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/RuntimeStatusBar.kt#L201)。
- 认证、账号用量、上下文来源、MCP、Hooks 的归属由结构任务统一决定；本任务不重复编写其他章节或擅自调整用户配置。

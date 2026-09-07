# Task Tree

- 说明工作台的会话切换、双侧栏和历史定位。
  - 核对左右侧栏、拖动宽度与菜单保持展开的实际操作。
  - 编排历史索引悬停预览、Check out 定位及终端进程管理示例。
  - 补充标签溢出导航与非当前标签右键操作的说明。
  - 按确认后的目录交付三语言内容、真实演示和浏览器验证。

# Details

- 左右侧栏独立持有展开/钉住状态；展开动画与宽度拖动见 [SessionTreeCliScreen.kt:96](../../Kodex/app/view/application/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/SessionTreeCliScreen.kt#L96)，拖动结束保存宽度见同文件 [427](../../Kodex/app/view/application/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/SessionTreeCliScreen.kt#L427)。
- 侧栏可选择 None、Terminal sessions、History index，不应误写成固定的会话目录：[SessionSidebar.kt:170](../../Kodex/app/view/application/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/SessionSidebar.kt#L170)。
- 历史索引悬停加载完整内容，问答条目复用只读表单：[SessionSidebar.kt:707](../../Kodex/app/view/application/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/SessionSidebar.kt#L707)。
- `Check out` 仅定位到历史条目，不回退对话、更不操作 Git：[SessionTreeCliScreen.kt:743](../../Kodex/app/view/application/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/SessionTreeCliScreen.kt#L743)。
- 终端进程条目悬停显示 ID 与完整命令，右键 `Close session` 调用进程会话关闭；与关闭 Kodex 会话标签区分：[SessionSidebar.kt:647](../../Kodex/app/view/application/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/SessionSidebar.kt#L647)。
- 标签横向滚动、选中项进入视口、右键非当前标签而不切换选择有专项测试：[SessionTabBarTest.kt:169](../../Kodex/app/view/application/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/app/SessionTabBarTest.kt#L169)、[316](../../Kodex/app/view/application/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/app/SessionTabBarTest.kt#L316)；录制前核对实际运行。
- 内容需等结构确认，录制复用播放器子任务；不为演示更改产品交互。

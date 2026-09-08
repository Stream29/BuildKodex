# Task Tree

- 说明工作台的会话切换、双侧栏和历史定位。
  - [done] 核对左右侧栏、拖动宽度与菜单保持展开的实际操作。
  - [done] 编排历史索引悬停预览、Check out 定位及终端进程管理示例。
  - [done] 补充标签溢出导航与非当前标签右键操作的说明。
  - [done] 独立准备三语言草稿、场景步骤与安全录制证据。
  - [done] 按确认后的目录交付三语言内容、真实演示和浏览器验证。
  - [done] 用真实特性开发会话的英文脱敏片段替换单一 UserMessage 索引样例。
    - [done] 只读筛选本地会话，确认纯 Kodex 特性开发范围。
    - [done] 翻译稳定历史片段，保留条目类型、顺序和已回答结果。
    - [done] 重录多种条目预览与 Check out。
    - [done] 验证新原生片段的浏览器回放并更新来源。
  - 补录会话目录打开、重命名、关闭、归档/恢复和删除确认的实际结果。
  - [done] 验证完整应用壳层的预览区域切换与延迟关闭并录制。

# Details

- 当前新原生 history-index 的 10 个检查点已逐行匹配，包含四类预览及浮窗交接/关闭；后面的 6 帧 fixture 和缺少壳层时序记录属于前轮结果，已被本次取代。见[补录验收](../../shared-context/findings/docs-showcase-refinement-2026-09-08.md)。
- 本轮用户明确允许从本地选择纯 Kodex 特性开发 Session。已选 Session 178「侧栏history index」中索引 697–826 的连续稳定历史片段，包含用户消息、计划更新、已回答问题和助手最终消息。仅制作英文脱敏副本，不修改原会话，不读取凭证，不重新执行历史中的工具/指令；片段不是整个会话的完整翻译。
- 后续用户又授权只读认证运行真实场景；在该英文历史的独立续篇中完成实际模型调用。新的 history-index 采用原生 CLI，而非测试侧保持 hover 请求的 fixture：录到用户/计划/助手/已回答问题四类预览、移入浮窗保持、移出关闭、索引菜单、Check out 与回到底部。原始 Session 178 仍完全只读。
- 本轮将精简正文合入已确认的 workspace 页面，复用真实 CLI 片段；有数据的交互复用测试侧真实渲染，明确录制种类和证据边界。
- workspace 原生 16 帧、history-index 6 帧、terminal-sessions 4 帧、session-tabs 4 帧已逐行回放核对；三语言五宽度检查通过。终端关闭仅作用于 fixture 句柄，标签 Rename/Close 只是菜单可见；不算真实进程终止或会话目录流程通过。
- 历史索引测试保持 hover 请求以跨越独立输入层，真实内容与 Check out 已验证，完整壳层关闭时序仍缺实录；据[当前验收](../../shared-context/findings/docs-interaction-acceptance-2026-09-08.md)保留新增叶节点，不提前完成根任务。

## 首批源码盘点与交付（历史记录）

- 左右侧栏独立持有展开/钉住状态；展开动画与宽度拖动见 [SessionTreeCliScreen.kt:96](../../Kodex/app/view/application/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/SessionTreeCliScreen.kt#L96)，拖动结束保存宽度见同文件 [427](../../Kodex/app/view/application/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/SessionTreeCliScreen.kt#L427)。
- 侧栏可选择 None、Terminal sessions、History index，不应误写成固定的会话目录：[SessionSidebar.kt:170](../../Kodex/app/view/application/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/SessionSidebar.kt#L170)。
- 历史索引悬停加载完整内容，问答条目复用只读表单：[SessionSidebar.kt:707](../../Kodex/app/view/application/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/SessionSidebar.kt#L707)。
- `Check out` 仅定位到历史条目，不回退对话、更不操作 Git：[SessionTreeCliScreen.kt:743](../../Kodex/app/view/application/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/SessionTreeCliScreen.kt#L743)。
- 终端进程条目悬停显示 ID 与完整命令，右键 `Close session` 调用进程会话关闭；与关闭 Kodex 会话标签区分：[SessionSidebar.kt:647](../../Kodex/app/view/application/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/SessionSidebar.kt#L647)。
- 标签横向滚动、选中项进入视口、右键非当前标签而不切换选择有专项测试：[SessionTabBarTest.kt:169](../../Kodex/app/view/application/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/app/SessionTabBarTest.kt#L169)、[316](../../Kodex/app/view/application/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/app/SessionTabBarTest.kt#L316)；录制前核对实际运行。
- 内容需等结构确认，录制复用播放器子任务；不为演示更改产品交互。
- 首批写入仅限本任务、`KodexDocs/drafts/workspace-tour/` 与 `out/docs-demos/workspace-tour/`；草稿文件和并行约束遵循总任务，不修改网站首页、导航或主题。
- 复用[实录流程](../../shared-context/findings/kodex-terminal-recording.md)：先录空 Home 下的双侧栏操作；历史索引/终端进程示例另核对安全数据准备，不能用空白索引声称覆盖完成。
- 首批已保留 `KodexDocs/drafts/workspace-tour/{zh-CN,zh-TW,en-US,scenes}.md` 与 `out/docs-demos/workspace-tour/` 的真实 PTY 实录；录制使用 120×36、独立临时 Home/socket/CLI 副本且未捕获输入。
- 实录已验证双侧栏 hover/分别钉住/移开后保持展开、左侧宽度拖动并持久化为 38 列、`None` 与 `Terminal sessions` 菜单；历史预览/Check out、活动终端进程菜单和标签溢出仍因安全数据/离线多会话入口不足而阻塞，详见 `scenes.md`。
- 已执行 `:app-view-application:linuxX64Test --rerun-tasks`（显式复用 Gradle daemon 的 Java 25，77 tests / 0 failures / 0 skipped）；未做浏览器播放器验收、目录确认或正式合入，因此任务根节点及后续叶节点保持未完成。
- 主 Session 已补做交付录制的独立浏览器重放，11 份原终端抓屏按顺序逐行匹配；通过范围仅双侧栏片段，非完整工作台章节。见[首批验收](../../shared-context/findings/docs-subagent-acceptance-2026-09-08.md)。

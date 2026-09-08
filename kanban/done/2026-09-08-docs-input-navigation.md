# Task Tree

- [done] 说明终端输入编辑和目录选择的细节。
  - [done] 演示多行粘贴、软换行、光标定位与撤销/重做。
  - [done] 核对输入快捷键、提交/换行配置及终端原生文本选择边界。
  - [done] 演示目录过滤、进入/确认、鼠标侧键返回及分层 Escape。
  - [done] 独立准备三语言草稿、场景步骤与安全录制证据。
  - [done] 按确认后的目录交付三语言内容、真实演示和浏览器验证。

# Details

- 本轮合入 workspace 页面，复用实录，补当前源码键位设置的可追溯片段；原生 Shift+拖选不是网页播放器能力，不以此阻塞已实现功能说明。
- 当前 input 已用新构建 CLI 重录，17 个检查点逐行回放一致；configuration 另含换行键选项实录。三语言 workspace 正式合入并通过五宽度检查。Shift+拖拽仅说明宿主终端边界，不算实录通过。见[当前验收](../../shared-context/findings/docs-interaction-acceptance-2026-09-08.md)。

## 首批源码盘点与交付（历史记录）

- 输入组件统一处理粘贴和换行规范化；Ctrl+Z 撤销、Ctrl+Y 重做、Ctrl+W 删除前一个词：[TextInput.kt:221-241](../../Kodex/app/view/components/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/components/TextInput.kt#L221-L241)、[TextInput.kt:558-573](../../Kodex/app/view/components/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/components/TextInput.kt#L558-L573)。
- 光标按终端 cell 映射，软换行不插入真实换行：[TextInput.kt:82-156](../../Kodex/app/view/components/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/components/TextInput.kt#L82-L156)；Unicode、粘贴原子编辑和视口行为见 [TextInputTest.kt:83-103,251-333](../../Kodex/app/view/components/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/components/TextInputTest.kt#L83-L103)。
- 提交与换行键由 Composer 配置决定，文案不能硬编码单一组合：[SessionTreeUiPrimitives.kt:50-62,194-202](../../Kodex/app/view/application/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/app/SessionTreeUiPrimitives.kt#L50-L62)、[ComposerInputTest.kt:24-59](../../Kodex/app/view/application/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/app/ComposerInputTest.kt#L24-L59)。
- 目录选择器直接输入过滤；Escape 先清过滤，再关闭；鼠标 Button8 返回父目录：[DirectoryPickerPopup.kt:97-134](../../Kodex/app/view/path-picker/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/pathpicker/DirectoryPickerPopup.kt#L97-L134)。
- 过滤后 Enter 进入首项、再次确认的流程有测试：[DirectoryPickerPopupTest.kt:130](../../Kodex/app/view/path-picker/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/pathpicker/DirectoryPickerPopupTest.kt#L130)。
- Shift+拖拽等终端原生行为需要在录制环境实际确认；不把播放器文字选择与真实应用内选择混为一谈。
- 首批写入仅限本任务、`KodexDocs/drafts/input-navigation/` 与 `out/docs-demos/input-navigation/`；草稿文件和并行约束遵循总任务，不修改网站首页、导航或主题。
- 复用[实录流程](../../shared-context/findings/kodex-terminal-recording.md)已通过的简繁中文粘贴、撤销/重做和目录过滤；扩展软换行、键位配置与 Button8 前重新核对真实终端输入，未验证的原生文本选择单列说明。
- 首批产物已写入 `KodexDocs/drafts/input-navigation/{zh-CN,zh-TW,en-US,scenes}.md`；审核录制与逐帧证据在 `out/docs-demos/input-navigation/`，包括 `input-navigation.cast`、tmux 帧、播放器回放页和 `verification.json`。
- 新实录为 120×36 的 asciicast v3，真实输入包含长 CJK/English 软换行、cell 光标移动、Ctrl+W、Ctrl+Z/Y、`do` 过滤、Enter 进入/确认、SGR Button8 返回和三层 Escape。播放器逐关键帧比对通过，CJK cell 偏移、真彩色和 1440/1024/768/390/320 px 无横向溢出已核对；窄屏易读性仍不等于已验证。
- 当前二进制 SHA256 为 `92539baaa88f4cb78a58ad73c5a23844ef49f65babb4ee968a692c80f26e6720`，`Kodex/` HEAD 为 `89205188d5da39f36b983a0bfb57752977feb21a`；二进制未重建且无法证明来自该源码提交。运行中的旧 Settings 画面未暴露当前源码的换行/提交下拉框，因此该键位配对仅以源码与测试为依据，不能冒充本次实录验证。
- 未运行 Gradle/mosaicTest，也未重建共享 Kodex 构建目录；未验证真实终端 `Shift+拖拽` 文本选择。正式三语言合入、确认网站目录、VitePress 集成浏览器验收仍待后续授权，故本任务根节点和总任务根节点保持未完成。
- 主 Session 独立重放交付录制，15 份原终端抓屏按顺序逐行匹配；接收首批素材和已录输入/目录片段，不把当前源码键位配置或原生文本选择列为实录已验证。见[首批验收](../../shared-context/findings/docs-subagent-acceptance-2026-09-08.md)。

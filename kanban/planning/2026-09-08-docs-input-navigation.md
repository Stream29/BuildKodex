# Task Tree

- 说明终端输入编辑和目录选择的细节。
  - 演示多行粘贴、软换行、光标定位与撤销/重做。
  - 核对输入快捷键、提交/换行配置及终端原生文本选择边界。
  - 演示目录过滤、进入/确认、鼠标侧键返回及分层 Escape。
  - 按确认后的目录交付三语言内容、真实演示和浏览器验证。

# Details

- 输入组件统一处理粘贴和换行规范化；Ctrl+Z 撤销、Ctrl+Y 重做、Ctrl+W 删除前一个词：[TextInput.kt:215](../../Kodex/app/view/components/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/components/TextInput.kt#L215)。
- 光标按终端 cell 映射，软换行不插入真实换行：[TextInput.kt:77](../../Kodex/app/view/components/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/components/TextInput.kt#L77)；Unicode、粘贴原子编辑和视口行为见 [TextInputTest.kt:251](../../Kodex/app/view/components/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/components/TextInputTest.kt#L251)。
- 提交与换行键由 Composer 配置决定，文案不能硬编码单一组合：[ComposerInputTest.kt:25](../../Kodex/app/view/application/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/app/ComposerInputTest.kt#L25)。
- 目录选择器直接输入过滤；Escape 先清过滤，再关闭；鼠标 Button8 返回父目录：[DirectoryPickerPopup.kt:97](../../Kodex/app/view/path-picker/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/pathpicker/DirectoryPickerPopup.kt#L97)。
- 过滤后 Enter 进入首项、再次确认的流程有测试：[DirectoryPickerPopupTest.kt:130](../../Kodex/app/view/path-picker/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/pathpicker/DirectoryPickerPopupTest.kt#L130)。
- Shift+拖拽等终端原生行为需要在录制环境实际确认；不把播放器文字选择与真实应用内选择混为一谈。

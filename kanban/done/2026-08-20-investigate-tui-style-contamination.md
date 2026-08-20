# Task Tree

- [done] 调查 TUI 跨区域样式污染
  - [done] 确认受影响的渲染层级
  - [done] 构造最小复现并定位根因
  - [done] 核对侧栏、History 与 Dialog 路径
  - [done] 汇总修复边界与验证建议

# Details

- 状态：`done`。本轮未修改产品代码。
- 本轮只调查左侧栏、History reasoning dim 样式和 Dialog 之间的跨区域显示干扰，不修改产品代码。
- 根因位于 `Kodex/Mosaic/mosaic-runtime/src/commonMain/kotlin/com/jakewharton/mosaic/surface.kt:220`。Bold 与 Dim 共用关闭码 SGR 22，但当前逐项切换逻辑把两者当成可独立关闭的属性。
- 当前编码会错误处理 `Dim → Bold`、`Bold+Dim → Bold`、`Bold+Dim → Dim`；SGR 22 同时清除 Bold 和 Dim，终端状态因此偏离 `TextPixel`。
- 侧栏 hover 可在 `Kodex/app/view/components/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/components/TuiInteractionStyle.kt:22` 生成 `Dim+Bold`，而 reasoning 在 `Kodex/app/view/history/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/history/CleanEventView.kt:303` 请求 Dim；二者位于同一逐行 ANSI 输出中。
- Dialog 在 `Kodex/app/view/components/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/components/TuiPopup.kt:260` 给背景 cell 追加 Dim，使原 Bold cell 变为 `Bold+Dim`，从而触发同一错误转换。
- 修复应位于 Mosaic ANSI 编码器，成组处理 Bold/Dim 强度状态；不应在侧栏、History 或 Dialog 增加局部 reset workaround。
- 临时 JVM 回归复现生成了 `SGR 1;2` 后接 `SGR 22` 的错误输出；使用 tmux 3.6 解释后，目标字符确实不再带 Dim。临时测试、脚本和 socket 均已删除。
- 建议增加 Bold/Dim 四种状态的完整 4×4 转换测试，再覆盖侧栏相邻 reasoning 与模态背景两条组件路径。

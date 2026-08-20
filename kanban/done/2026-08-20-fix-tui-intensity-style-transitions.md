# Task Tree
- [done] 修复 TUI 强度样式串扰
  - [done] 修正 ANSI `Bold`/`Dim` 状态切换
  - [done] 覆盖全部强度状态转换
  - [done] 运行 Mosaic 渲染回归测试

# Details
- 状态：`done`。
- 目标：修复侧栏、历史记录和弹窗之间由 ANSI 强度样式残留导致的显示异常。
- 已确认根因：终端 SGR `22` 会同时关闭 `Bold` 与 `Dim`，现有渲染器在只关闭其中一个样式时没有重新启用另一个仍需保留的样式。
- 修复方案：将 `Bold` 和 `Dim` 作为共享终端强度状态处理；发生转换时先在必要时发出 `22`，再按目标样式重新发出 `1` 和/或 `2`。
- 修改：
  - `Kodex/Mosaic/mosaic-runtime/src/commonMain/kotlin/com/jakewharton/mosaic/surface.kt`
  - `Kodex/Mosaic/mosaic-runtime/src/commonTest/kotlin/com/jakewharton/mosaic/AnsiRenderingTest.kt`
- 验证：
  - 通过 `:mosaic-runtime:jvmTest --tests com.jakewharton.mosaic.AnsiRenderingTest`。
  - `:mosaic-runtime:spotlessCheck` 仅因未修改的 `mosaic-runtime/src/commonTest/kotlin/com/jakewharton/mosaic/layout/DrawTextStyleOverlayTest.kt` 导入排序失败。

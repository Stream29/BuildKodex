# Task Tree

- [done] 修复对话框后的双宽字符导致内部背景条纹
  - [done] 在真实 CLI 和组件快照中复现问题
  - [done] 定位宽字符 leader/continuation 与绘制顺序的交互
  - [done] 添加双宽字符 ANSI 背景回归测试
    - [done] 覆盖弹窗内部从宽字符 leader 开始的 cell 相位
    - [done] 覆盖弹窗内部从 continuation 开始的 cell 相位
  - [done] 将对话框清屏移到调用方绘制 modifier 之前
    - [done] 保留 Dim、焦点、按键和指针行为
    - [done] 不改变 Mosaic 全局宽字符存储语义
  - [done] 更新 TuiDialog 绘制顺序决策
  - [done] 运行组件 JVM 与 Linux 原生测试
  - [done] 重建 Linux release CLI 并在 tmux 复测
  - [done] 清理临时会话和文件

# Details

- 状态：`done`。绘制顺序已修正，回归和真实 CLI 复测均通过。
- 用户已授权直接实施。
- 根因位于 `TuiDialog`：调用方背景先于子 `Filler` 绘制。宽字符 continuation 的样式存储在 leader；后续清屏拆分 cell 时，原 continuation 保留未指定背景，形成隔列背景。
- 修复范围限定在 Kodex `TuiDialog` 的清屏顺序。先把覆盖范围规范化为单宽空格，再绘制调用方背景和内容。
- 预计修改：
  - `Kodex/app/view/components/src/mosaicMain/kotlin/io/github/stream29/kodex/cli/components/TuiPopup.kt`
  - `Kodex/app/view/components/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/components/TuiDialogTest.kt`
  - `checklist/tui-interaction-components.md`
- 验证：
  - `:app-view-components:jvmTest`
  - `:app-view-components:linuxX64Test`
  - `:app-cli:linkReleaseExecutableLinuxX64`
  - tmux TRUECOLOR ANSI 捕获不再在对话框内部交替重置背景。
- 实施结果：
  - 使用位于调用方 modifier 之前的 `drawBehind` 清除对话框覆盖范围，并移除后绘制的子 `Filler`。
  - ANSI 回归先在旧实现确认 `A蓝/B默认/C蓝`，修复后两个 cell 相位均为连续背景。
  - tmux 中 Settings 受汉字覆盖的四行均只在对话框右边缘输出一次 `49m`。

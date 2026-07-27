# Task Tree

- [done] 简化TUI按钮的状态视觉
  - [done] 保留方括号作为所有按钮的固定边界
  - [done] 移除焦点、悬停和按下状态对应的额外括号
  - [done] 用Bold表示悬停、反色视频表示按下、Dim表示禁用
  - [done] 覆盖按钮状态的ANSI渲染
  - [done] 更新交互组件决策记录

# Details

- 焦点由物理终端光标表示，不再重复投影为文本装饰。
- 已通过 `:tui-components:linuxX64Test` 和 `:tui-demo:linuxX64Test`，并重连 Linux 调试可执行文件。

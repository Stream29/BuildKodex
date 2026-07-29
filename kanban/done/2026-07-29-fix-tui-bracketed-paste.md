# Task Tree

- [done] 修复 TUI 大段粘贴的可靠性
  - [done] 复现字符丢失、Tab 焦点逸出与多行超高
  - [done] 在 Mosaic TTY 层聚合 bracketed paste 为原子事件
  - [done] 将粘贴事件沿焦点路径分发给当前控件
  - [done] 让 Composer 原子插入粘贴内容
  - [done] 补充终端、runtime 与 Composer 回归测试
  - [done] 运行针对性 Gradle 与真实 PTY 验证

# Details

- 用户于 2026-07-29 明确授权实现大段粘贴修复。
- 未关闭用户保留的 tmux CLI；验证使用真实 PTY 的测试基础设施。
- 高度上限与滚动作为后续独立工作，本任务只修复粘贴字符完整性与粘贴内容不触发快捷键。

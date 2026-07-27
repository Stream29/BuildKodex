# Task Tree

- [done] 为TUI按钮提供按压反色反馈
  - [done] 在`TuiButton`的pressed分支应用终端反色视频
  - [done] 覆盖按下和释放后的ANSI渲染状态
  - [done] 记录按钮按压视觉语义

# Details

- 使用终端的reverse-video语义；在默认黑底白字主题中表现为白底黑字，且不会把其他终端主题硬编码成黑白。
- `:tui-components:linuxX64Test`通过；Linux调试可执行文件已重新链接。

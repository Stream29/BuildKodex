# Task Tree

- [done] 简化 Settings 页面的选中表达
  - [done] 移除 route 和换行键选项中的星号
  - [done] 以背景色统一表达当前选中项
  - [done] 移除右侧重复的 route 标题
  - [done] 更新设置页交互与颜色测试
  - [done] 运行 TUI Demo 相关验证

# Details

左侧 route 已经表达当前页面，右侧内容不再重复显示 `Global settings` 等标题。

验证通过：`:tui-demo:jvmTest`、`:tui-demo:linuxX64Test`、`:tui-demo:linkDebugExecutableLinuxX64`。真实 80×20 tmux 终端确认页面中不再出现星号和重复 route 标题。

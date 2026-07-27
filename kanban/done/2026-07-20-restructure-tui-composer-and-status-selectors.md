# Task Tree

- [done] 重构TUI对话布局与状态选择器
  - [done] 将历史、输入框、状态与操作重排为上中下布局
  - [done] 在状态栏提供模型选择器
  - [done] 在状态栏提供推理强度选择器
  - [done] 让选择器支持悬停、鼠标点选和方向键选择
  - [done] 覆盖布局与交互行为测试
  - [done] 更新TUI交互组件决策记录

# Details

- 下拉菜单向上展开，避免遮挡底部状态与操作区域。
- 选择器复用现有的模型目录与会话设置更新路径。
- 已通过 Linux Native 与 JVM 的 `:tui-components`、`:tui-demo` 测试。
- macOS 远端工作树未包含本地未提交的 TUI 模块，未覆盖远端目录。

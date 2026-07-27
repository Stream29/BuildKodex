# Task Tree

- [done] 为TUI焦点系统加入方向键遍历
  - [done] 定义方向键与文本输入、菜单、滚动等局部控件的优先级
  - [done] 依据控件终端坐标在同一焦点scope中移动焦点
  - [done] 保留现有Tab和Shift+Tab遍历
  - [done] 覆盖水平、垂直、边界和弹层焦点测试

# Details

方向键应能在可聚焦控件之间移动，同时不破坏当前控件已经拥有的方向键语义。

已验证：`tui-focus`、`tui-components`、`tui-demo` 的 JVM 与 Linux X64 测试，以及 `tui-demo` Linux X64 可执行文件链接。

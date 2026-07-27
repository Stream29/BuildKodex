# Task Tree

- [done] 建立可复用的 TUI Dialog 基础组件
  - [done] 为模态内容提供进入焦点、scope 限制与退出焦点恢复
  - [done] 基于 `TuiPopupHost` 实现居中覆盖与背景输入拦截
  - [done] 阻断父 action scope 的快捷键，保留 Dialog 输入控件的普通按键
  - [done] 处理 Escape 和可选的背景点击关闭
  - [done] 覆盖居中、模态焦点、关闭与背景输入测试

# Details

本轮只提供 Dialog 基础组件，不设计具体的全局设置表单或滚动表单控件。

已验证：`tui-focus`、`tui-components`、`tui-demo` 的 JVM 与 Linux X64 测试，以及 `tui-demo` Linux X64 可执行文件链接。

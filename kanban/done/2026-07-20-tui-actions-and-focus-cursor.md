# Task Tree

- [done] 为TUI引入作用域化快捷键动作
  - [done] 实现`TuiAction`、`TuiActionScope`和焦点宿主路由
  - [done] 让按钮和快捷键复用同一语义动作
  - [done] 将物理终端光标锚点泛化到所有可聚焦控件
  - [done] 迁移Demo工具栏快捷键
  - [done] 覆盖焦点、快捷键和按钮交互测试

# Details

- 快捷键遵循Compose的父级preview路由：保留宿主动作、内层作用域、页面作用域、焦点遍历、当前焦点目标、未处理回退。
- `TuiAction`的可用性独立于工具栏的响应式可见性；同一动作可以由按钮、键盘和未来菜单复用。
- `:tui-focus:linuxX64Test :tui-components:linuxX64Test :tui-demo:linuxX64Test`通过，并复用了配置缓存。
- JVM测试被Mosaic现有`mosaic-tty-terminal`的`AtomicInt`未解析引用阻断；本次修改未触及该模块。

# Task Tree

- [done] 修复TUI弹层菜单固定截断模型项
  - [done] 以终端可用高度替换固定的七行上限
  - [done] 保留空间不足时的焦点窗口与键盘可达性
  - [done] 覆盖完整显示和受限窗口两种菜单高度
  - [done] 运行TUI组件与demo的相关测试

# Details

模型、推理强度菜单不再固定截断为七行，而是使用当前终端宿主高度；更小的终端仍通过活动项窗口保持键盘可达。

验证通过：`:tui-components:jvmTest`、`:tui-demo:jvmTest`、`:tui-components:linuxX64Test`、`:tui-demo:linuxX64Test`、`:tui-demo:linkDebugExecutableLinuxX64`。

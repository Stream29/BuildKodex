# Task Tree

- [done] 将TUI状态菜单改为组件化弹层
  - [done] 实现弹层宿主、锚点和基于已测量内容的位置策略
  - [done] 让弹层在宿主表面覆盖渲染并正确处理菜单外点击
  - [done] 用新组件替换demo中的`Box`、`onPlaced`和手工偏移逻辑
  - [done] 覆盖位置、悬停、键盘选择和菜单外关闭行为
  - [done] 运行TUI组件与demo的相关测试及Linux链接验证

# Details

采用与Compose `Popup`相近的分层：调用处只声明触发器锚点和弹层内容；宿主负责表面覆盖层；位置策略根据锚点、宿主尺寸和已测量的弹层内容决定坐标。终端没有独立窗口，因此覆盖层仍在同一Mosaic表面渲染，但坐标计算和命中行为不再泄漏到业务UI。

已通过`:tui-components:jvmTest`、`:tui-demo:jvmTest`、`:tui-components:linuxX64Test`、`:tui-demo:linuxX64Test`和`:tui-demo:linkDebugExecutableLinuxX64`验证。

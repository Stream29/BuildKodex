# Task Tree

- [done] 为TUI按钮接入可靠的鼠标悬停
  - [done] 在Mosaic添加命中路径上的enter/exit hover回调
  - [done] 覆盖Mosaic的同级切换与离开命中区域语义
  - [done] 让`TuiPressable`维护hover状态并呈现按钮样式
  - [done] 覆盖Kodex组件回归测试

# Details

- 终端已经启用`MouseTracking.AnyEvents`；Mosaic根据当前命中路径维护hover生命周期，下游只消费enter/exit，不自行追踪全局鼠标位置。
- `:mosaic-runtime:linuxX64Test`、`:tui-components:linuxX64Test`和`:tui-demo:linuxX64Test`通过；新的Linux调试可执行文件已重新链接。

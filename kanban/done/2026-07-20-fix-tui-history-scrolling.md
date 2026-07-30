# Task Tree

- [done] 修复TUI对话历史滚动
  - [done] 确认终端滚轮事件到达历史视图的路径
  - [done] 让历史视图把滚动状态应用到实际可见行
  - [done] 覆盖鼠标滚动与历史偏移渲染的回归测试
  - [done] 在Linux Native Mosaic测试链路验证长历史浏览

# Details

用户报告对话历史区域无法正常滚动。修复遵循`checklist/tui-interaction-components.md`中由Kodex持有滚动状态、Mosaic仅负责原始指针事件的边界；已通过`:tui-demo:linuxX64Test`验证。
